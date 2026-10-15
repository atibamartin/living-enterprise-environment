# v4 — Enterprise Network Design

**Project:** Living Enterprise Environment (LEE)  
**Phase:** 2 — Network Architecture & Segmentation  
**Status:** Implemented and validated (October 2026)  
**Related:** `v5-network-segmentation-planning.md`, `v6-active-directory-network-preparation.md`, `v6a-pfsense-deployment-and-validation.md`, `phase-2-closeout.md`

## 1. Objective and business case

Design a realistic, defensible small enterprise network that separates systems by role and trust level while maintaining essential business services. Phase 1 had placed the domain controller, Windows client, Ubuntu administrator host and Kali attack host on a **single VMware NAT network (192.168.133.0/24)**. That flat layout made lateral access easy and provided no meaningful inter-zone policy enforcement.

The target architecture places a stateful pfSense firewall between purpose-specific networks. It makes permitted access deliberate, testable and observable rather than treating reachability as a default entitlement.

**Design principle:** *Connectivity is not the goal. Required connectivity is the goal.*

## 2. Design requirements

1. Maintain Active Directory, DNS, Kerberos, SYSVOL/SMB, Group Policy and domain time for a domain-joined Windows client.
2. Isolate a Kali attack system from production-like workloads by default.
3. Separate administrative access from user, server and attack traffic.
4. Provide controlled outbound services for the domain controller (approved DNS, approved NTP and restricted HTTPS).
5. Generate interpretable firewall pass/deny evidence for investigation and later Wazuh ingestion.
6. Support future monitoring infrastructure without placing it in the management zone.
7. Permit recovery through VMware snapshots and a private pfSense configuration backup.

## 3. As-built topology

```text
                        VMware NAT / upstream
                       VMnet8 192.168.133.0/24
                      VMware gateway 192.168.133.2
                                  |
                             pfSense WAN
                           em0 (DHCP address)
                                  |
                 +----------------+----------------+
                 |       LEE-pfSense-01             |
                 |  Stateful routing and policy    |
                 +-----+-------+-------+-----------+
                       |       |       |           |
             SERVER em1| CLIENT| ATTACK| MANAGEMENT|
               VMnet10 |VMnet11|VMnet12|VMnet13    |
               .10.1   | .20.1 | .30.1 | .40.1     |
                 |     |       |       |           |
          WIN-SRV-01 WIN-CLT-01 Kali-01 Ubuntu-01
          .10.10     .20.100    .30.10  .40.10

Future (not deployed in Phase 2): dedicated MONITORING VMnet,
proposed 192.168.50.0/24, for Wazuh.
```

All IPv4 addresses shown below use prefix `/24` within their respective zones.

| Zone | VMware network | Subnet | pfSense NIC / gateway | System | Assignment |
|---|---|---|---|---|---|
| WAN | VMnet8 NAT | 192.168.133.0/24 | em0 / DHCP | VMware upstream gateway .133.2 | VMware NAT |
| SERVER | VMnet10 | 192.168.10.0/24 | em1 / 192.168.10.1 | LEE-WIN-SRV-01 / 192.168.10.10 | Static |
| CLIENT | VMnet11 | 192.168.20.0/24 | em2 / 192.168.20.1 | LEE-WIN-CLT-01 / 192.168.20.100 | pfSense DHCP |
| ATTACK | VMnet12 | 192.168.30.0/24 | em3 / 192.168.30.1 | LEE-Kali-01 / 192.168.30.10 | Static |
| MANAGEMENT | VMnet13 | 192.168.40.0/24 | em4 / 192.168.40.1 | LEE-Ubuntu-01 / 192.168.40.10 | Static |

**VMware configuration:** internal VMnets are custom networks, with VMware DHCP and host virtual adapters disabled. pfSense supplies routing; pfSense DHCP is enabled for the CLIENT zone. The WAN continues to use VMware NAT for upstream connectivity.

## 4. Design decisions and reasons

### 4.1 pfSense as the routed security boundary

**Decision:** Connect five virtual NICs to pfSense: one WAN and four internal zones.  
**Why:** Traffic crossing zones must pass through a common policy enforcement and logging point.  
**Trade-off:** More configuration and troubleshooting than a flat network, but closer to real enterprise routing and access control.

### 4.2 Separate workloads by role

- **SERVER:** trusted infrastructure (AD DS, DNS, Kerberos, domain time and SMB/SYSVOL). A compromise here carries high organizational impact.
- **CLIENT:** domain workstation that must request a limited set of services from the DC.
- **ATTACK:** untrusted Kali host, isolated until a deliberate, time-bounded simulation is authorized.
- **MANAGEMENT:** Ubuntu administration workstation with narrowly scoped pfSense WebGUI access.
- **WAN:** upstream transit; not a trusted management or workload segment.

The zones represent *trust boundaries*, not merely different IP ranges.

### 4.3 Client DHCP and server static addressing

The CLIENT DHCP pool is `192.168.20.100–192.168.20.199`, with gateway `192.168.20.1`, DNS server `192.168.10.10`, and search domain `livingenterprise.lab`. The DC has static `192.168.10.10` so clients can consistently locate directory and DNS services. Kali and Ubuntu also use stable addresses to make firewall evidence and access rules unambiguous.

### 4.4 Default deny and stateful return traffic

pfSense interface rules evaluate traffic as it **enters** an interface. A CLIENT-initiated connection to an allowed DC service can receive responses through the established state; a second broad SERVER-to-CLIENT pass rule is unnecessary. Separate explicit SERVER-originated block rules express policy and are useful evidence, but do not prevent valid stateful replies.

### 4.5 Management is privileged, not automatically unrestricted

Ubuntu's management role does not imply permission to reach every machine. Phase 2 allows Ubuntu `192.168.40.10` to access pfSense's HTTPS administration service on TCP 443; tests confirmed Ubuntu-to-DC ICMP and SMB remained denied.

### 4.6 Dedicated monitoring network deferred to Phase 3

Wazuh is planned for a separate **MONITORING** segment, tentatively `192.168.50.0/24`. This network was **not created in Phase 2**. Its collection, dashboard, agent enrollment and administrative flows must be specified before any firewall allowances are added.

## 5. Packet walk: three design examples

**CLIENT → DC / Kerberos TCP 88**
1. CLIENT `192.168.20.100` recognizes DC `192.168.10.10` as off-subnet.
2. CLIENT sends packet to its default gateway `192.168.20.1` (pfSense CLIENT).
3. pfSense checks inbound CLIENT rules; a scoped pass to `LEE_DC` TCP/UDP 88 permits the connection.
4. pfSense routes to SERVER; the DC responds through the stateful session.
5. The pass is observable in pfSense logs when rule logging is enabled.

**Kali → DC / SMB TCP 445**
1. Kali `192.168.30.10` sends toward gateway `192.168.30.1`.
2. ATTACK has no rule permitting DC TCP 445.
3. Default deny drops the attempted connection; `nc` times out and ATTACK log records source, destination and port.

**Ubuntu → pfSense / HTTPS TCP 443**
1. Ubuntu `192.168.40.10` targets the firewall's own WebGUI.
2. MANAGEMENT pass rule specifically matches source Ubuntu, destination `This Firewall (self)`, TCP 443.
3. Administrative access works without enabling management access to SERVER, CLIENT or ATTACK.

## 6. Architecture verification

| Verification | Observed result |
|---|---|
| CLIENT receives pfSense DHCP lease and DC DNS | `192.168.20.100`, gateway `.20.1`, DNS `.10.10` after correction of earlier stale DNS settings |
| CLIENT discovers DC across subnet | `nltest /dsgetdc:livingenterprise.lab /force` found `.10.10` |
| CLIENT applies domain computer policy | `gpupdate /target:computer /force` succeeded; `gpresult` listed Default Domain Policy |
| Kali is on ATTACK network | `ip -br addr` `.30.10/24`, `ip route` default `.30.1` |
| Kali has gateway link-layer reachability | `ip neigh show` resolved pfSense gateway MAC |
| Kali-to-protected-zone tests | DC:445, CLIENT:445, Ubuntu:22 timed out with matching ATTACK denies |
| Management isolation | Ubuntu-to-DC ping and TCP 445 denied with MANAGEMENT logs |
| Narrow ICMP diagnostic exception | Kali ping `.30.1` succeeded 4/4 after explicit ICMP echo-request rule |

A failed ping by itself is **not** proof of failed network connectivity. In this lab, Kali's ARP/neighbor entry established Layer 2 reachability, while pfSense logs proved the original ICMP echo requests were dropped by policy.

## 7. Changes from original design

- Original flat VMware NAT addressing was replaced by the five-zone layout.
- The original Phase 5 pfSense/segmentation objectives (`v11`, `v12`) were implemented earlier during Phase 2 and are cross-referenced rather than duplicated.
- A separate monitoring zone was explicitly planned for Phase 3 rather than combining telemetry and privileged management.
- The actual AD dependency set was discovered through incremental testing, not granted as an unrestricted allow-any rule.

## 8. Evidence and documentation references

- VMware virtual network editor and VM NIC assignments (retain screenshots where available).
- pfSense interface configuration and full rule-tab screenshots.
- `v5-network-segmentation-planning.md`: trust model and traffic matrix.
- `v6-active-directory-network-preparation.md`: migration and application validation.
- `v6a-pfsense-deployment-and-validation.md`: detailed implementation and rule-level troubleshooting.
- `phase-2-closeout.md`: final evidence inventory, backup/snapshot status and handoff.

## 9. Engineering lessons

- Network segmentation changes service dependencies, not just IP addresses.
- Design each permission around a known workload, source, destination and protocol.
- Verify the real configuration rather than relying on a rule's description.
- Test positive and negative outcomes; both matter to security engineering.
- Preserve recovery checkpoints before moving to new infrastructure.

**Outcome:** A working, routed, security-zoned enterprise lab architecture ready for monitoring integration.

