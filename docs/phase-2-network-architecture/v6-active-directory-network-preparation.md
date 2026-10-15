# v6 — Active Directory Network Preparation and Validation

**Project:** Living Enterprise Environment (LEE)  
**Phase:** 2 — Network Architecture & Segmentation  
**Status:** Implemented and validated (October 2026)  
**Related:** `v4-enterprise-network-design.md`, `v5-network-segmentation-planning.md`, `v6a-pfsense-deployment-and-validation.md`, `phase-2-closeout.md`

## 1. Objective

Move the existing Active Directory domain controller and domain-joined Windows client from a flat VMware NAT network into separate pfSense-routed SERVER and CLIENT zones **without breaking domain identity services**. Validate actual application behavior—directory discovery, secure channel, authentication, SYSVOL/Group Policy and time synchronization—rather than treating an open port as proof of success.

Domain: `livingenterprise.lab` (NetBIOS `LEE`). Domain controller: `LEE-WIN-SRV-01`. Workstation: `LEE-WIN-CLT-01`.

## 2. Before-and-after addressing

| System | Earlier VMnet8 NAT address | Final zone / address | Gateway | DNS |
|---|---|---|---|---|
| Windows Server / DC | 192.168.133.130 | SERVER VMnet10 / 192.168.10.10 static | 192.168.10.1 | Local DNS on DC (loopback) |
| Windows 11 client | 192.168.133.131 | CLIENT VMnet11 / 192.168.20.100 DHCP | 192.168.20.1 | 192.168.10.10 |
| pfSense | New router | SERVER .10.1; CLIENT .20.1 | WAN via VMnet8 | N/A |

CLIENT DHCP scope: `192.168.20.100–192.168.20.199`, gateway `.20.1`, DNS `.10.10`, domain suffix `livingenterprise.lab`, lease 7200 seconds.

**Why this matters:** Domain clients must use a DNS service capable of resolving internal AD records. An external resolver alone cannot discover `_ldap`/`_kerberos` SRV records in the private domain.

## 3. Migration preparation

1. Confirm the existing domain controller and client were healthy on the original flat network.
2. Record old addressing, NIC configuration, domain details and critical service status.
3. Take pre-change VMware snapshots and plan rollback.
4. Establish pfSense interfaces and routing before moving workloads.
5. Move the DC first; re-register its AD/DNS records and validate DC services.
6. Configure CLIENT DHCP and narrowly scoped CLIENT-to-DC permissions.
7. Move the client; verify its new address, gateway and **effective DNS server**, not only DHCP lease information.
8. Validate AD features incrementally and use pfSense logs to identify missing network dependencies.

## 4. Domain controller migration and health

**Final configuration:** DC `192.168.10.10/24`, gateway `192.168.10.1`, local DNS resolver. Old `.133.130` address/gateway assumptions were removed and DNS records refreshed. The Windows Server 2025 DC remained the domain's directory, DNS and Kerberos authority.

**Useful verification commands (run on the DC):**

```powershell
ipconfig /all
Get-NetIPAddress -AddressFamily IPv4
Get-Service NTDS,DNS,KDC,W32Time
dcdiag /test:Connectivity
dcdiag /test:DNS
Get-ADDomainController
Resolve-DnsName LEE-WIN-SRV-01.livingenterprise.lab
Resolve-DnsName _ldap._tcp.dc._msdcs.livingenterprise.lab -Type SRV
Resolve-DnsName _kerberos._tcp.livingenterprise.lab -Type SRV
```

**Observed:** DC service checks and `dcdiag` connectivity/DNS tests passed. DC discovery identified `192.168.10.10`; internal LDAP/Kerberos SRV records were present. The server retained IPv6 link-local/AAAA behavior; IPv6 was **not** disabled on the domain controller solely because the lab's routed plan uses IPv4.

### DNS forwarding

Configured forwarders: `8.8.8.8` and `8.8.4.4`, with root hints disabled for the intended forwarding path. pfSense SERVER policy allows the DC to reach `APPROVED_DNS` on TCP/UDP 53. Both internal directory lookups and external forwarded DNS queries were validated.

```powershell
Get-DnsServerForwarder
Resolve-DnsName www.microsoft.com
```

**Security decision:** DNS egress was limited to approved forwarders rather than a general UDP 53 allow to any address.

### Domain controller NTP

The DC was configured to use approved NIST time peers (`time-a-g.nist.gov` / `129.6.15.28` and `time-c-b.nist.gov` / `132.163.96.3`) with client-mode `0x8` settings. SERVER pfSense permits UDP 123 to `APPROVED_NTP`. `w32tm` output demonstrated synchronization; initial root dispersion after restart decreased as synchronization settled.

```powershell
w32tm /query /source
w32tm /query /status
w32tm /query /peers
```

**Why:** Kerberos relies on sufficiently synchronized clocks; domain time is an identity-service dependency, not merely cosmetic system configuration.

## 5. Client migration and DNS correction

The Windows 11 client was moved to VMnet11 and received `192.168.20.100/24` from pfSense. A stale DNS setting initially referenced the old DC address `192.168.133.130`, even though the DHCP lease and new IP were correct. The effective DNS settings were subsequently reconciled and client-to-DC DNS queries were logged as passes.

```powershell
ipconfig /all
Get-DnsClientServerAddress -AddressFamily IPv4
Resolve-DnsName LEE-WIN-SRV-01.livingenterprise.lab
Resolve-DnsName _ldap._tcp.dc._msdcs.livingenterprise.lab -Type SRV
```

**Lesson:** A correct DHCP address does not guarantee that an interface is actually using the intended DNS server. Check the effective adapter configuration.

## 6. AD dependency discovery and firewall changes

The team avoided adding an unrestricted CLIENT-to-SERVER rule. Each dependency was observed, tested and permitted specifically to the DC host (`LEE_DC = 192.168.10.10`).

| CLIENT → DC | Why it was permitted | Validation |
|---|---|---|
| TCP/UDP 53 | DNS and service discovery | DNS lookups succeeded; pfSense pass logs |
| TCP/UDP 389 | LDAP and CLDAP | TCP 389 reachable; DC discovery succeeded |
| TCP/UDP 88 | Kerberos | TCP 88 reachable; secure channel and later domain logon succeeded |
| TCP 445 | SMB/SYSVOL and policy files | TCP 445 reachable; later computer GPO processing succeeded |
| TCP 135 | RPC Endpoint Mapper | pfSense logs showed matching allowed endpoint discovery |
| TCP 49152–65535 | Dynamic RPC to DC | High-port traffic passed; GPO computer refresh completed |
| UDP 123 | Domain time | `w32tm` resync changed source to the DC |

**Important:** These are the *as-built* rules for this lab, not a blanket universal AD firewall template. Source is CLIENT subnet and destination is the single DC host. See `v5` for risk discussion and `v6a` for exact rule ordering.

### Why TCP 135 was not enough

TCP 135 tells a Windows client where to find an RPC service. The client may then connect to a separate dynamic TCP endpoint. During troubleshooting, a DC endpoint at **TCP 49679** was tested successfully:

```powershell
Test-NetConnection 192.168.10.10 -Port 49679
```

After a DC-scoped dynamic RPC range rule was enabled, computer Group Policy refresh succeeded. The temporary rule allowing only TCP 49679 was **disabled**; it remains in pfSense as a documented test artifact. The high-port range remains an area for future hardening if Windows RPC port allocation is deliberately constrained and revalidated.

## 7. Domain controller discovery and secure channel

**Commands on CLIENT:**

```powershell
nltest /dsgetdc:livingenterprise.lab /force
nltest /sc_query:livingenterprise.lab
Test-NetConnection 192.168.10.10 -Port 389
Test-NetConnection 192.168.10.10 -Port 88
Test-NetConnection 192.168.10.10 -Port 445
```

**Observed:** `nltest /dsgetdc` located `LEE-WIN-SRV-01.livingenterprise.lab` at `192.168.10.10`. The secure-channel query initially failed with `ERROR_NO_LOGON_SERVERS (1311)` during the restricted-rule stage; after the required services were allowed it returned `Status = 0 0x0 NERR_Success`.

**Interpretation:** Network-level reachability and actual domain trust communication must be verified separately. The successful secure-channel query was stronger evidence than a port test alone.

## 8. SYSVOL and Group Policy validation

An early `Test-Path "\\livingenterprise.lab\SYSVOL"` returned False even after TCP 445 was reachable, and an initial computer policy refresh timed out. That result did **not** prove that the SYSVOL share was missing; possible causes included authentication, referrals, RPC dependencies and service state. The team continued testing rather than opening all traffic.

**Final observed results:**

```powershell
gpupdate /target:computer /force
# Computer Policy update has completed successfully.

gpresult /scope computer /r
# Group Policy was applied from: LEE-WIN-SRV-01.livingenterprise.lab
# Applied Group Policy Objects: Default Domain Policy
```

On October 14, 2026, `gpresult` showed the computer policy had applied from the domain controller. This is **application-level** proof that the workstation can retrieve and process its required domain computer policy in the segmented architecture.

**Permissions lesson:** Under the ordinary domain user session, `gpresult /scope computer /r` returned **Access Denied**. In an appropriately elevated session, it succeeded. That is a permissions/context difference, not evidence that GPO connectivity failed.

## 9. Domain user authentication and Kerberos evidence

The earlier test session was the workstation's local administrator (`LEE-WIN-CLT-01\LEE-LocalAdmin`), for which `klist` showed zero tickets. This was expected for that session and did not demonstrate a domain Kerberos outage.

Later, the user signed in as `LEE\alicia.carter` and ran:

```powershell
whoami
klist
gpresult /scope user /r
```

**Observed:**
- `whoami` → `lee\alicia.carter`.
- `klist` → **five** Kerberos tickets, including a TGT (`krbtgt`) and LDAP, ProtectedStorage and CIFS service tickets.
- Ticket encryption → `AES-256-CTS-HMAC-SHA1-96`.
- `gpresult /scope user /r` identified Alicia Carter in the Finance OU and membership in `GG-Finance-Users`.
- No user GPO appeared as applied; that is not a failure when no applicable user policy is configured.

**Why this matters:** The tests verify more than a reachable DC: they demonstrate an authenticated domain-user session with real Kerberos service-ticket acquisition.

## 10. Client time synchronization recovery

**Initial state:**

```powershell
w32tm /query /source
# Local CMOS Clock
w32tm /query /status
# Leap Indicator: 3 (not synchronized); Stratum: 0
```

**Controlled recovery:**

```powershell
w32tm /resync /rediscover
w32tm /query /source
w32tm /query /status
```

**Observed:**
- Source: `LEE-WIN-SRV-01.livingenterprise.lab`.
- Reference IP: `192.168.10.10`.
- Leap indicator: `0` (no warning).
- Stratum: `3`.
- Successful sync time: October 13, 2026 at 6:16:05 PM local.

The client uses the DC as its domain time source, while the DC synchronizes upstream to approved NIST servers. The client-to-DC UDP 123 rule is scoped to this need.

## 11. Validation matrix

| Test | Result | Evidence meaning |
|---|---|---|
| DC DNS/AD service checks | Pass | Directory infrastructure survived address migration |
| Internal SRV/A lookups | Pass | DNS service discovery works |
| External DNS via approved forwarders | Pass | Required outbound resolver path works |
| `nltest /dsgetdc` | Pass | Client locates DC across firewall |
| `nltest /sc_query` | Pass after rule refinement | Domain secure channel operational |
| TCP 389/88/445/49679 | Pass after scoped rules | Required network endpoints reachable |
| `gpupdate /target:computer /force` | Pass | Computer policy processing functional |
| `gpresult /scope computer /r` elevated | Pass | Default Domain Policy applied from DC |
| `whoami` / `klist` as Alicia | Pass | Domain logon and Kerberos tickets obtained |
| `gpresult /scope user /r` | Pass | Domain user/group context confirmed |
| `w32tm` resync and source | Pass | Client synchronizes to domain controller |

## 12. Troubleshooting and lessons learned

1. **Migrate deliberately:** change one dependency at a time and verify health after every network move.
2. **DNS is foundational:** stale interface DNS can mask otherwise correct IP/gateway configuration.
3. **AD is a collection of services:** DNS, LDAP, Kerberos, SMB, RPC and time each contribute to functioning domain operations.
4. **An open port is not an application test:** TCP 445 True did not settle SYSVOL/GPO status.
5. **RPC is a two-stage conversation:** endpoint discovery on TCP 135 can lead to dynamic high TCP ports.
6. **Logon context matters:** local-admin `klist` with no tickets does not invalidate domain authentication; test as a real domain user.
7. **Privileges matter for diagnostics:** elevated `gpresult` may be needed for computer scope.
8. **Time is an identity dependency:** check source, stratum and last sync, not just clock display.
9. **Prefer explicit evidence over assumptions:** correlate CLI results with pfSense passes/denies.

## 13. Evidence to attach / link

- DC static address and DNS settings (redact sensitive values as needed).
- `dcdiag` connectivity/DNS and `Get-ADDomainController` output.
- pfSense CLIENT rule inventory and temporary TCP 49679 rule shown disabled.
- `nltest` success, GPO success, elevated `gpresult`.
- Domain-user `whoami`, `klist`, `gpresult` output.
- `w32tm` before/after status and source.
- Corresponding pfSense pass/deny screenshots.

**Avoid publishing** passwords, tokens, raw pfSense configuration backup or other secrets. Preserve timestamps and host/IP references useful for incident analysis.

## 14. Result and handoff

The Windows domain controller and workstation now operate on **separate security zones** while supporting DNS, directory discovery, secure channel, domain logon, Kerberos, computer Group Policy and time synchronization. The implementation and firewall evidence are recorded in `v6a` and `phase-2-closeout.md`.

**Next:** Phase 3 will add a dedicated Wazuh MONITORING zone. Define collection and administration flows first; do not weaken existing AD segmentation to accommodate the monitoring system.

