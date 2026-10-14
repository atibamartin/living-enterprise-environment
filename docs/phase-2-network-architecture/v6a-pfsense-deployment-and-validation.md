# LEE Phase 2 — pfSense Deployment and Segmentation Validation

**Project:** Living Enterprise Environment (LEE)  
**Document:** `v6a-pfsense-deployment-and-segmentation-validation.md`  
**Status:** Core implementation validated; closeout evidence and snapshots pending  
**Evidence period:** September–October 14, 2026  
**Related:** `v4-enterprise-network-design.md`, `v5-network-segmentation-planning.md`, `v6-active-directory-network-preparation.md`

## 1. Executive summary

Phase 2 transformed the LEE lab from a flat VMware NAT network into a segmented enterprise-style environment routed and filtered by pfSense. A Windows Server domain controller, domain-joined Windows 11 client, Ubuntu management host, and Kali attack host now occupy separate Layer 3 zones. Access between zones is **default deny**, with narrowly scoped allowances for demonstrated enterprise requirements.

The central engineering principle is **“Connectivity isn't the goal. Required connectivity is the goal.”** The build uses controlled testing to identify required flows, create narrowly targeted rules, retest, and collect supporting evidence. The environment is designed to support Phase 3 security monitoring and subsequent controlled threat simulation.

**Important scope distinction:** pfSense and core segmentation—originally listed as Phase 5 v11/v12—were implemented in Phase 2. The original v11/v12 objectives should be marked *implemented in Phase 2*, not rebuilt later. Additional hardening, telemetry, and advanced segmentation remain future work.

## 2. Objectives and acceptance criteria

1. Replace flat, shared NAT networking with distinct SERVER, CLIENT, ATTACK, and MANAGEMENT subnets.
2. Route internal networks through pfSense, preserving VMware NAT as upstream WAN connectivity.
3. Keep Active Directory, DNS, Kerberos, SMB/SYSVOL, Group Policy, and time synchronization functional across SERVER/CLIENT segmentation.
4. Enforce default-deny inter-zone access and permit only required flows.
5. Establish a management path to pfSense without relying on broad anti-lockout or default LAN pass rules.
6. Prove that the ATTACK zone can reach its own gateway for diagnostics but cannot reach protected services without an explicit exception.
7. Preserve before/after evidence and decisions for future SOC investigations and portfolio demonstrations.

## 3. Architecture and address plan

```text
                       VMware NAT / VMnet8
                     192.168.133.0/24
                       GW 192.168.133.2
                              |
                       pfSense WAN (em0)
                    DHCP ~192.168.133.132
                              |
                 +------------+-------------+
                 |       LEE-pfSense-01      |
                 +------------+-------------+
                              |
          +-------------------+--------------------+------------------+
          |                   |                    |                  |
       SERVER              CLIENT              ATTACK           MANAGEMENT
       VMnet10             VMnet11             VMnet12            VMnet13
   192.168.10.0/24     192.168.20.0/24    192.168.30.0/24   192.168.40.0/24
   pfSense .10.1        pfSense .20.1       pfSense .30.1      pfSense .40.1
       |                   |                    |                  |
  Windows DC          Windows 11             Kali              Ubuntu
  192.168.10.10       192.168.20.100       192.168.30.10     192.168.40.10
```

| Zone | VMnet | pfSense interface | Gateway | System | Host address | Assignment |
|---|---|---|---|---|---|---|
| WAN | VMnet8 | em0 | VMware .133.2 | pfSense WAN | DHCP, observed ~.133.132 | VMware NAT DHCP |
| SERVER | VMnet10 | em1 | 192.168.10.1 | LEE-WIN-SRV-01 | 192.168.10.10 | Static |
| CLIENT | VMnet11 | em2 | 192.168.20.1 | LEE-WIN-CLT-01 | 192.168.20.100 | pfSense DHCP |
| ATTACK | VMnet12 | em3 | 192.168.30.1 | LEE-Kali-01 | 192.168.30.10 | Static NetworkManager |
| MANAGEMENT | VMnet13 | em4 | 192.168.40.1 | LEE-Ubuntu-01 | 192.168.40.10 | Static |

**VMware network settings:** Internal VMnets are custom isolated networks with VMware DHCP and host virtual adapters disabled. pfSense supplies routing and, on CLIENT, DHCP. The client DHCP pool is `192.168.20.100–192.168.20.199`; advertised DNS is `192.168.10.10`, gateway `192.168.20.1`, domain suffix `livingenterprise.lab`.

**Planned Phase 3 addition (not deployed):** MONITORING, proposed `192.168.50.0/24`, gateway `192.168.50.1`, dedicated VMnet and pfSense interface subject to available virtual NIC capacity. Wazuh will reside here; telemetry and dashboard rules will be explicitly scoped after requirements are verified.

## 4. Baseline and migration rationale

Originally, Ubuntu, Kali, the domain controller, and the Windows client shared VMnet8 (`192.168.133.0/24`). Their original addresses were approximately `.129`, `.128`, `.130`, and `.131`, respectively. That flat design supported initial build and authentication exercises but did not enforce meaningful zone separation. Migrating to pfSense made network access an explicit engineering decision.

### Migration sequence

1. Deploy pfSense with five VMware NICs; assign WAN, SERVER, CLIENT, ATTACK, and MANAGEMENT.
2. Configure isolated VMnets, interface addresses, routing, and aliases.
3. Move Ubuntu to MANAGEMENT and verify restricted WebGUI access at `https://192.168.40.1`.
4. Move the Windows Server DC to SERVER; repair/verify AD DNS registration, forwarding, and time source.
5. Move Windows 11 to CLIENT, enable pfSense DHCP, and identify required AD flows using failures and logs.
6. Move Kali to ATTACK, assign a static address, and test blocked inter-zone access.
7. Record validated results and retain baseline snapshots.

## 5. pfSense policy model

**Rules are evaluated on the interface where traffic enters pfSense.** Thus a Kali-initiated flow is evaluated on ATTACK, not SERVER. Rules are stateful: return traffic for an established permitted flow generally does not need a separate reverse-direction pass rule. pfSense rule descriptions document intent but do not enforce it; source, destination, protocol, ports, interface, and ordering do.

### 5.1 Aliases and named objects

| Object | Meaning |
|---|---|
| `LEE_DC` | Domain controller host `192.168.10.10` |
| `LEE_CLIENT_NET` | `192.168.20.0/24` |
| `LEE_ATTACK_NET` | `192.168.30.0/24` |
| `LEE_MANAGEMENT_NET` | `192.168.40.0/24` |
| `APPROVED_DNS` | Google DNS forwarders `8.8.8.8`, `8.8.4.4` |
| `APPROVED_NTP` | NIST time peers `129.6.15.28`, `132.163.96.3` |
| `NON_PUBLIC_IPV4` | Address alias used with inverted destination match for public HTTPS; **verify its actual contents before treating it as comprehensive** |

### 5.2 SERVER policy (implemented)

- Block/log `LEE_DC` → `LEE_ATTACK_NET`, any IPv4.
- Block/log `LEE_DC` → `LEE_MANAGEMENT_NET`, any IPv4.
- Block/log `LEE_DC` → `LEE_CLIENT_NET`, any IPv4.
- Pass `LEE_DC` → public destination selection, TCP/443, using inverted `NON_PUBLIC_IPV4` alias.
- Pass `LEE_DC` → `APPROVED_DNS`, TCP/UDP 53.
- Pass `LEE_DC` → `APPROVED_NTP`, UDP 123.
- Former broad LAN IPv4/IPv6 pass rules disabled; anti-lockout rule disabled after management access was established and verified.

**Caution:** The SERVER inter-zone block rules do not independently prevent traffic initiated from ATTACK or CLIENT. Their respective ingress interface policies do that. Stateful replies to permitted sessions remain possible.

### 5.3 CLIENT → DC policy (implemented and tested)

| Service | Protocol / destination port | Why permitted | Evidence |
|---|---|---|---|
| DNS | TCP/UDP 53 | Domain discovery, name resolution | `Resolve-DnsName`; pfSense pass logs |
| LDAP/CLDAP | TCP/UDP 389 | Directory/DC discovery | `nltest /dsgetdc`; port and logs |
| Kerberos | TCP/UDP 88 | Domain authentication | Domain login, `klist`, port and logs |
| SMB | TCP 445 | SYSVOL, NETLOGON and policy files | UNC share listing and pass logs |
| RPC Endpoint Mapper | TCP 135 | Discover RPC endpoints | Pretest denied; later TCP connectivity and pass logs |
| Dynamic RPC | TCP 49152–65535 | Support dynamically allocated DC RPC endpoints | `gpupdate`, `gpresult`, pfSense logs for TCP 49679 and later 49681 |
| NTP | UDP 123 | Workstation sync to DC | `w32tm /resync /rediscover` and status |

**Verify NTP rule inventory:** Client NTP synchronization was demonstrated, but the exact final pfSense NTP rule configuration should be captured in the final rules export/screenshot rather than inferred solely from successful synchronization.

**RPC decision:** Initial traffic to TCP/135 was blocked, with Group Policy errors and long timeouts. After allowing Endpoint Mapper, the client was denied dynamic RPC TCP/49679. A narrow exception proved the dependency; the final standard dynamic RPC range was selected because the allocated endpoint later changed to TCP/49681. This is a deliberate availability-versus-exposure trade-off: it is restricted to CLIENT source network and DC destination but permits any TCP service listening within that range. The obsolete single-port diagnostic rule should remain disabled after range validation.

### 5.4 MANAGEMENT policy

Ubuntu `192.168.40.10` can access pfSense management HTTPS at `192.168.40.1:443` through an explicit TCP rule. Access was revalidated after disabling the former anti-lockout rule. Other management-to-zone permissions are to be granted only after identifying a real administrative requirement.

### 5.5 ATTACK policy

Default deny is retained for ATTACK → protected zones. A diagnostic exception allows **ICMP Echo Request only** from `192.168.30.10` to the pfSense ATTACK interface `192.168.30.1`, with logging. This is not an inter-zone pass rule.

Kali's DNS queries to the former VMware NAT resolver `192.168.133.2:53/UDP` were blocked and logged. The legacy resolver configuration is a follow-up; no access should be opened solely to suppress the log messages.

## 6. Active Directory migration and validation

### 6.1 Domain controller

- DC `LEE-WIN-SRV-01`, domain `livingenterprise.lab`, static `192.168.10.10/24`, gateway `192.168.10.1`.
- AD DS, DNS, and KDC services confirmed; A and SRV records verified; `dcdiag` DNS/Connectivity tests passed.
- DNS forwarders: `8.8.8.8` and `8.8.4.4`; root hints disabled for forwarding configuration.
- PDC time service uses specified NIST peers; NTP sync and status validated.
- Do not disable IPv6 on the domain controller solely because the lab's routed zones use IPv4.

### 6.2 Client DHCP and DNS

The domain-joined Windows 11 workstation moved to VMnet11 and received `192.168.20.100/24` from pfSense. A stale former DNS setting (`192.168.133.130`) was observed during migration; later domain resolution and authentication succeeded using the DC's new address. Preserve this as a troubleshooting lesson: **verify actual adapter DNS, not only DHCP lease details.**

### 6.3 Authentication, shares, and policy

- `nltest /dsgetdc:livingenterprise.lab /force` located `LEE-WIN-SRV-01` at `192.168.10.10`.
- `nltest /sc_query:livingenterprise.lab` returned `NERR_Success` after required authentication paths were enabled.
- On the DC, `Get-SmbShare -Name SYSVOL,NETLOGON` confirmed both shares, and the SYSVOL path existed.
- From the client, UNC access to `\\LEE-WIN-SRV-01.livingenterprise.lab\SYSVOL` and `\\livingenterprise.lab\SYSVOL` succeeded under domain user `LEE\alicia.carter`.
- `klist` for Alicia showed a Kerberos TGT and service tickets including LDAP and CIFS, with AES-256 ticket encryption.
- `gpupdate /target:computer /force` completed successfully after RPC connectivity was corrected.
- Elevated `gpresult /scope computer /r` showed `Default Domain Policy` applied from `LEE-WIN-SRV-01.livingenterprise.lab` on October 14, 2026.
- `gpresult /scope user /r` under Alicia showed a domain user context, Finance OU, and `GG-Finance-Users`; **no user-side GPO was reported as applied**. This is not equivalent to failed authentication; a user GPO would be needed to test actual user policy settings.

**Local versus domain user:** Local admin `LEE-WIN-CLT-01\LEE-LocalAdmin` is not a domain user and does not automatically receive domain-user Kerberos tickets or access to domain resources. Computer Group Policy is processed in the computer context even when invoked by a local administrator. Reading computer-scope RSOP with `gpresult` may require elevation; Alicia's unelevated `gpresult /scope computer /r` returned Access Denied and succeeded in an elevated session.

### 6.4 Time synchronization

Before correction, the client reported `Local CMOS Clock`, `Leap Indicator: 3`, `Stratum: 0`. After:

```powershell
w32tm /resync /rediscover
w32tm /query /source
w32tm /query /status
```

The client reported source `LEE-WIN-SRV-01.livingenterprise.lab`, stratum 3, leap indicator 0, and reference IP `192.168.10.10`. This matters because time drift can undermine Kerberos and log correlation. pfSense log timestamps were kept in UTC; Windows outputs were shown in local time, so timestamps must be normalized when correlating evidence.

## 7. Kali migration and ATTACK-zone testing

### 7.1 VMware and static IP

VMware network adapter changed from VMnet8 to **Custom VMnet12**. Initial Kali state was `eth0 UP` without an IPv4 address, no default route, and NetworkManager showing disconnected. With no ATTACK DHCP server, a static NetworkManager profile was created:

```bash
nmcli connection add \
  type ethernet \
  ifname eth0 \
  con-name LEE-ATTACK-STATIC \
  ipv4.method manual \
  ipv4.addresses 192.168.30.10/24 \
  ipv4.gateway 192.168.30.1 \
  ipv6.method disabled

nmcli connection up LEE-ATTACK-STATIC
ip -br addr
ip route
ip neigh show
```

**Observed:** `eth0` had `192.168.30.10/24`, default route via `192.168.30.1`, and ARP neighbor `192.168.30.1 dev eth0 lladdr 00:0c:29:f9:29:56 REACHABLE`. This establishes Layer 2 reachability even when ICMP is filtered.

### 7.2 Controlled gateway ICMP experiment

**Before:** `ping -c 4 192.168.30.1` returned 100% loss. pfSense ATTACK default-deny logs showed ICMP blocked from `192.168.30.10` to `192.168.30.1` (October 14, 14:42:40–14:43:05 in firewall log display).

**Change:** Explicit ATTACK rule: Pass IPv4 ICMP Echo Request, source `192.168.30.10`, destination `192.168.30.1` (ATTACK address), logging enabled. Description: `Optional ICMP Attack Allow`.

**After:** `ping -c 4 192.168.30.1` returned 4/4 replies, 0% loss, average RTT ~0.664 ms. pfSense pass log matched the new rule at October 14 15:52:20 (display time). **Conclusion:** firewall policy—not a broken VMnet connection—explained the initial ping failure.

### 7.3 Protected-zone connection tests

| Source | Target | Command | Observed |
|---|---|---|---|
| Kali `192.168.30.10` | DC `192.168.10.10:445` | `nc -nvz -w 3 192.168.10.10 445` | Connection timed out |
| Kali `192.168.30.10` | Client `192.168.20.100:445` | `nc -nvz -w 3 192.168.20.100 445` | Connection timed out (client powered on) |
| Kali `192.168.30.10` | Ubuntu `192.168.40.10:22` | `nc -nvz -w 3 192.168.40.10 22` | Connection timed out |

**Evidence precision:** These timeouts are consistent with ATTACK default deny, but the screenshots supplied so far do **not** show matching filtered pfSense block events for each of these three individual TCP attempts. Capture matching logs to fully attribute each timeout to firewall policy. For the SSH test, confirm the Ubuntu service is listening if using it as an application-level reachability test.

## 8. Troubleshooting and lessons learned

| Observation | Hypothesis / evidence | Controlled action | Outcome / lesson |
|---|---|---|---|
| Windows client received new DHCP address but had stale DNS | Old interface DNS setting persisted | Verify actual DNS server and correct configuration | DHCP address alone does not prove DNS migration |
| DNS/LDAP/Kerberos reachability succeeded but SYSVOL initially failed | SMB blocked; later local versus domain security context differed | Allow TCP/445; test IP, FQDN, and domain UNC with appropriate identity | Transport success and authorization are separate checks |
| Computer Group Policy stalled for minutes | Event `7320` error `0x6BA` and repeated `7017` ~126-second RPC calls; pfSense blocked TCP/135 | Allow Endpoint Mapper TCP/135 | RPC endpoint discovery required |
| TCP/135 allowed but GPO still needed dynamic endpoint | pfSense blocked TCP/49679 | Test temporary port, then standard dynamic RPC range | Endpoint allocation changed to TCP/49681; don't hardcode one transient port |
| Local admin lacked Kerberos tickets and remote CIM attempt failed | Local identity cannot be treated as domain principal | Test using `LEE\alicia.carter` for domain authentication; avoid using WinRM error alone as RPC evidence | Identity and transport must be assessed separately; WinRM has its own service/port requirements |
| Kali pinged gateway unsuccessfully | ARP neighbor reachable; pfSense logs showed ICMP deny | Permit only ICMP Echo Request to ATTACK gateway | 4/4 ping replies; no broad inter-zone rule required |
| Kali attempted DNS via old VMware gateway | Blocked UDP/53 to `192.168.133.2` | Leave blocked; inspect resolver config later | Legacy settings can survive migration; log noise is not automatically a firewall requirement |
| DC attempted public TCP/80 | Default deny logged | No exception absent verified business requirement | Observe first; don't chase every blocked egress request |

**Repeatable method:** Observe → hypothesize → collect evidence → isolate → make one controlled change → retest → validate.

## 9. Evidence inventory

Preserve sanitized screenshots or exported text showing:

1. VMware VMnet12 attachment for Kali and pfSense interface assignments.
2. SERVER rules and disabled legacy broad LAN passes.
3. CLIENT rules for DNS, LDAP, Kerberos, SMB, RPC Endpoint Mapper, and dynamic RPC.
4. pfSense pass logs for TCP/135 and dynamic RPC TCP/49681.
5. `gpupdate /target:computer /force` success and elevated `gpresult /scope computer /r` showing Default Domain Policy.
6. `whoami`, `klist`, and `gpresult /scope user /r` from domain user Alicia.
7. `w32tm /query /source` and `/status` showing DC synchronization.
8. Kali `ip -br addr`, `ip route`, `ip neigh show`.
9. ATTACK ICMP deny logs, new scoped ICMP rule, ICMP pass log, and 4/4 ping output.
10. ATTACK default-denied DNS attempts to `192.168.133.2:53`.
11. **Still to capture:** ATTACK → SERVER/CLIENT/MANAGEMENT TCP deny log entries correlated with test timestamps.
12. Final pfSense rule inventory, aliases, snapshots, and exported configuration (protect any secrets).

## 10. Phase 2 closeout checklist

- [x] Define four isolated internal networks and WAN uplink.
- [x] Deploy pfSense and assign five interfaces.
- [x] Migrate DC, client, Ubuntu, and Kali to designated zones.
- [x] Verify AD authentication, SYSVOL, computer GPO, and time synchronization.
- [x] Validate CLIENT dynamic RPC rule and gateway ICMP exception.
- [x] Demonstrate default-denied ATTACK DNS and initial gateway ICMP.
- [x] Attempt protected-zone connections from Kali; all timed out.
- [ ] Capture matching firewall deny entries for each protected-zone test.
- [ ] Verify obsolete single-port TCP/49679 diagnostic rule is disabled.
- [ ] Review Kali's inherited DNS resolver and determine future egress requirements.
- [ ] Export/review full final rule and alias inventory; verify client NTP rule.
- [ ] Capture final snapshots of Kali and pfSense and record snapshot names.
- [ ] Update Phase 2 README and roadmap cross-references.

## 11. Phase 3 handoff: Dedicated MONITORING zone

**Proposed, not yet implemented:** `192.168.50.0/24` with pfSense gateway `192.168.50.1` and Wazuh server on a reserved static address. Select a dedicated VMware VMnet and verify available pfSense NIC capacity before assignment. Establish telemetry flow requirements (agents, syslog, dashboard/API, updates) first; then allow only necessary sources, destinations, and ports. Restrict dashboard administration to designated MANAGEMENT hosts. Do not grant ATTACK unrestricted access to the monitoring stack. Validate time synchronization and consistent UTC timestamp interpretation before correlating pfSense and host events.

## 12. Engineering conclusions

Phase 2 establishes a functional and testable segmented enterprise foundation. Active Directory services work across controlled CLIENT-to-SERVER paths; the ATTACK zone has a defined static identity and default-deny posture; pfSense rules produce evidence of both permitted and denied traffic. Remaining items are closeout validation, rule inventory, DNS hygiene, and snapshots—not a repeat of the architecture build. The environment is ready for **Phase 3 planning**, with Wazuh isolated in a new MONITORING zone.

