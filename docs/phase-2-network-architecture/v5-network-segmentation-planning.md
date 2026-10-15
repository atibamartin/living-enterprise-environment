# v5 — Network Segmentation Planning and Policy Decisions

**Project:** Living Enterprise Environment (LEE)  
**Phase:** 2 — Network Architecture & Segmentation  
**Status:** Plan implemented and baseline validated (October 2026)  
**Related:** `v4-enterprise-network-design.md`, `v6-active-directory-network-preparation.md`, `v6a-pfsense-deployment-and-validation.md`

## 1. Objective

Translate the enterprise topology into **explicit, auditable trust boundaries**. Identify who needs to communicate with whom, which service is necessary, why it is necessary, and how to validate both permitted and denied behavior.

**Policy rule:** *No capability → no requirement → no firewall rule.*

## 2. Trust classification and intended access

| Zone | Workload | Trust posture | Baseline |
|---|---|---|---|
| SERVER | Domain controller / DNS | High-value identity infrastructure | Permit essential approved outbound services; block unsolicited zone initiation |
| CLIENT | Domain-joined Windows workstation | User workload | Permit tested AD dependencies to `LEE_DC`; deny other destinations unless justified |
| ATTACK | Kali simulation host | Untrusted | Deny protected-zone access; permit only optional ping to own pfSense interface |
| MANAGEMENT | Ubuntu admin node | Privileged administration | Permit Ubuntu → pfSense WebGUI TCP 443 only in Phase 2 |
| WAN | VMware NAT upstream | External/untrusted transit | No user-defined inbound pass rules; WAN private/bogon blocking shown |

**Important:** The SERVER zone is not an implicit source of unrestricted management traffic. A management system is privileged but must still have least-privilege connectivity.

## 3. As-built traffic matrix

| Source | Destination | Protocol / port | Policy | Reason / evidence |
|---|---|---|---|---|
| CLIENT net | DC `.10.10` | TCP/UDP 53 | Pass | DNS and DC discovery |
| CLIENT net | DC | TCP/UDP 389 | Pass | LDAP / CLDAP directory discovery |
| CLIENT net | DC | TCP/UDP 88 | Pass | Kerberos authentication |
| CLIENT net | DC | TCP 445 | Pass | SMB, SYSVOL and GPO dependencies |
| CLIENT net | DC | TCP 135 | Pass | RPC Endpoint Mapper |
| CLIENT net | DC | TCP 49152–65535 | Pass | Windows dynamic RPC needed for observed GP processing |
| CLIENT net | DC | UDP 123 | Pass | Domain time synchronization |
| DC `.10.10` | `APPROVED_DNS` | TCP/UDP 53 | Pass | DNS forwarders 8.8.8.8 and 8.8.4.4 |
| DC | `APPROVED_NTP` | UDP 123 | Pass | Approved NIST peers |
| DC | Destination **outside** `NON_PUBLIC_IPV4` | TCP 443 | Pass | Scoped public HTTPS egress; inverted alias match verified |
| DC | ATTACK net | IPv4 any | Block/log | Prevent server-initiated attack-zone access |
| DC | MANAGEMENT net | IPv4 any | Block/log | Prevent unapproved server-to-admin traffic |
| DC | CLIENT net | IPv4 any | Block/log | Prevent unsolicited server-initiated client traffic; stateful replies still work |
| Ubuntu `.40.10` | pfSense (self) | TCP 443 | Pass | Firewall WebGUI administration |
| Kali `.30.10` | pfSense ATTACK `.30.1` | ICMP echo request | Pass/log | Narrow optional gateway diagnostic |
| Kali | DC `.10.10` | TCP 445 | Deny | Confirmed with `nc` and matching pfSense ATTACK logs |
| Kali | Client `.20.100` | TCP 445 | Deny | Confirmed with `nc` and matching pfSense ATTACK logs |
| Kali | Ubuntu `.40.10` | TCP 22 | Deny | Confirmed with `nc` and matching pfSense ATTACK logs |
| Ubuntu | DC `.10.10` | ICMP / TCP 445 | Deny | Confirmed with ping, `nc`, and MANAGEMENT logs |
| All other unspecified inter-zone flows | Any | Unapproved | Default deny | No blanket allow rules on protected zones |

**Clarification:** This matrix records Phase 2's *observed and implemented* policy. It does not imply the AD port list is universal or complete for every possible domain workload (e.g., password changes, domain joins, GC lookups or future remote administration). Future requirements must be tested separately.

## 4. Firewall rule evaluation and statefulness

pfSense processes interface rules where packets **enter**. The first matching applicable rule determines the disposition (subject to other pfSense-generated rules and existing states). The firewall tracks established permitted sessions, so return packets do not require an unconditional reverse-direction pass rule.

**Example:** CLIENT `.20.100` initiates TCP 445 to DC `.10.10`. CLIENT interface permits it; the DC response follows the established state. The SERVER-originated block to CLIENT is not supposed to interrupt that response.

**Configuration inspection checklist:** action; enabled/disabled state; interface; IPv4/IPv6; protocol; source and source port; destination and destination port; destination inversion; rule order; logging; and any relevant states. A description such as “public HTTPS” does not itself restrict destinations.

## 5. Incremental rule development method

1. Identify a specific business/service requirement.
2. Predict the required destination, transport and port.
3. Attempt the service **before** adding a pass rule; record error and pfSense deny entry.
4. Add the smallest useful pass rule; log when appropriate.
5. Save and **Apply Changes**; retest.
6. Confirm the application succeeds, not merely a TCP handshake.
7. Check firewall logs for the intended rule and document the evidence.
8. Remove or disable temporary validation rules; review whether the remaining scope is justified.

**Troubleshooting model:** Observe → hypothesize → collect evidence → isolate → make one controlled change → retest → validate.

## 6. Case study: RPC and Group Policy

**Symptom:** Client domain discovery and Kerberos/SMB connectivity had improved, but Group Policy computer processing initially timed out.

**Evidence:** pfSense logs showed denied client-to-DC TCP 135 and high TCP ports. TCP 135 is the RPC Endpoint Mapper; after endpoint discovery, Windows services may require a separate dynamic RPC destination port.

**Controlled experiment:** A temporary CLIENT `.20.100` → DC TCP **49679** pass rule allowed a targeted connectivity test (`Test-NetConnection 192.168.10.10 -Port 49679` returned True). Later a DC-scoped TCP **49152–65535** pass rule accommodated dynamic RPC. The temporary TCP 49679 rule was **disabled** and retained as historical validation evidence.

**Application validation:** `gpupdate /target:computer /force` completed successfully, and `gpresult /scope computer /r` identified the DC and applied Default Domain Policy.

**Security trade-off:** The dynamic RPC range is broad in destination-port count but **narrow in source/destination** (CLIENT net → the DC host). This was a pragmatic functional baseline, not proof that every high port is required. A later hardening exercise may constrain RPC server ports and narrow the firewall allowance after testing.

## 7. Case study: isolating Kali without losing gateway diagnostics

- Kali configured on `192.168.30.10/24` with default route `192.168.30.1`.
- `ip neigh show` resolved the gateway MAC; Layer 2 access was intact.
- `ping 192.168.30.1` initially failed and ATTACK logs showed default-denied ICMP.
- Added one **ICMP echo request** pass from Kali `.30.10` to its pfSense interface `.30.1`.
- Ping then returned **4/4 replies** and the pass was logged.
- TCP probes from Kali to SERVER 445, CLIENT 445 and MANAGEMENT 22 still timed out with corresponding denies.

**Conclusion:** A precise diagnostic permission does not require relaxing protected-zone isolation.

## 8. Aliases: maintainable policy objects

| Alias | Value / meaning |
|---|---|
| `LEE_DC` | 192.168.10.10 |
| `LEE_CLIENT_NET` | 192.168.20.0/24 |
| `LEE_ATTACK_NET` | 192.168.30.0/24 |
| `LEE_MANAGEMENT_NET` | 192.168.40.0/24 |
| `APPROVED_DNS` | 8.8.8.8, 8.8.4.4 |
| `APPROVED_NTP` | 129.6.15.28, 132.163.96.3 |
| `NON_PUBLIC_IPV4` | 10/8, 172.16/12, 192.168/16, 127/8, 169.254/16, 100.64/10, 0/8, 192.0.0/24, 192.0.2/24, 198.18/15, 198.51.100/24, 203.0.113/24, 224/4, 240/4 |

The DC HTTPS rule uses **destination invert match** on `NON_PUBLIC_IPV4`, port TCP 443. The alias is a custom exclusion list and does **not** by itself prove every remaining address is safe or publicly routable.

## 9. Validation plan and collected evidence

**Positive tests**
- Domain controller discovery (`nltest /dsgetdc`), secure channel (`nltest /sc_query`).
- DNS resolution, Kerberos authentication and domain-user logon.
- Computer GPO refresh and `gpresult`.
- Domain time resynchronization (`w32tm`).
- Ubuntu-to-pfSense HTTPS administration.
- Kali-to-own-gateway ICMP diagnostic.

**Negative tests**
- Kali-to-DC TCP 445, Kali-to-client TCP 445, Kali-to-Ubuntu TCP 22.
- Ubuntu-to-DC ICMP and TCP 445.
- Unapproved Kali DNS requests to stale VMware resolver `.133.2` denied; deliberately not opened.

**Evidence format:** Record date/time and time zone, initiating command, source IP, destination IP/port, observed result, pfSense interface, matched rule and action. Keep screenshots of rule inventories and aliases; avoid public disclosure of credentials or raw pfSense configuration XML.

## 10. Simulations: policy lifecycle (future work)

For a later authorized attack exercise:
1. Write a simulation objective and expected indicators.
2. Take baseline snapshots; verify logging and monitoring.
3. Create **temporary**, narrow, logged access only if the scenario requires it (specified source, target, protocol, duration).
4. Run the controlled test and capture both allowed and denied evidence.
5. Disable/remove the exception, clear lingering states if necessary, and revalidate default deny.
6. Document rollback and detection results.

**No attack-simulation exceptions were added as a standing inter-zone access policy during Phase 2.**

## 11. Outstanding improvements / scope boundaries

- Additional AD protocols may be needed for future workloads; no speculative rules should be added.
- CLIENT Internet egress was not broadly opened merely because Windows generated blocked HTTPS attempts.
- Kali's `/etc/resolv.conf` retained stale VMware NAT DNS `192.168.133.2`; blocked queries were observed, and configuration was intentionally left unchanged absent a requirement.
- No floating rules were configured; WAN had no user-defined pass rules in the captured inventory.
- Wazuh MONITORING zone and telemetry exceptions are Phase 3 tasks.

## 12. Result

The final policy makes **required** client-to-DC services functional while preserving logged denial of unauthorized attack and management flows. The traffic matrix serves as the baseline for Phase 3 monitoring and later controlled simulation.

