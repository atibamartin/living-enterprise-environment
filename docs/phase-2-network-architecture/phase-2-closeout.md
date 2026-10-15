# LEE — Phase 2 Network Architecture & Segmentation Closeout

**Project:** Living Enterprise Environment (LEE)  
**Phase:** 2 — Network Architecture & Segmentation  
**Closeout date:** October 14, 2026  
**Status:** Implementation and baseline validation complete; monitoring integration is a subsequent phase.

## 1. Executive summary

Phase 2 replaced a flat VMware NAT network with a routed, segmented enterprise lab protected by pfSense 2.9.0. Domain services were restored and validated across the CLIENT–SERVER boundary using explicitly permitted traffic. The ATTACK zone was isolated from protected workloads and MANAGEMENT access was limited to the pfSense administrative interface. The team gathered screenshots, command output, firewall rule inventories, deny/pass logs, and recovery checkpoints. The pfSense configuration backup was saved and renamed; five VM snapshots were taken.

**Security design principle:** *Connectivity is not the goal. Required connectivity is the goal.*

## 2. Final network architecture

| Zone | VMware network | IPv4 network | pfSense interface | Principal system |
|---|---|---|---|---|
| WAN / upstream | VMnet8 (NAT) | 192.168.133.0/24 | WAN (em0), DHCP | VMware NAT gateway 192.168.133.2 |
| SERVER | VMnet10 | 192.168.10.0/24 | 192.168.10.1 (em1) | LEE-WIN-SRV-01 — 192.168.10.10 |
| CLIENT | VMnet11 | 192.168.20.0/24 | 192.168.20.1 (em2) | LEE-WIN-CLT-01 — 192.168.20.100 (DHCP) |
| ATTACK | VMnet12 | 192.168.30.0/24 | 192.168.30.1 (em3) | LEE-Kali-01 — 192.168.30.10 (static) |
| MANAGEMENT | VMnet13 | 192.168.40.0/24 | 192.168.40.1 (em4) | LEE-Ubuntu-01 — 192.168.40.10 (static) |

Custom internal VMnets have VMware DHCP and host adapters disabled. The pfSense CLIENT DHCP pool is 192.168.20.100–192.168.20.199, with gateway 192.168.20.1, DNS 192.168.10.10 and domain `livingenterprise.lab`.

### Future architecture decision

A separate MONITORING zone is planned for Wazuh in Phase 3 (proposed 192.168.50.0/24). **This zone has not yet been implemented.** Define telemetry flows and management boundaries before creating firewall permissions.

## 3. Security policy and observed rule inventory

pfSense evaluates interface rules on the interface where traffic enters. Its stateful firewall permits replies for established allowed connections without requiring symmetric pass rules.

### SERVER interface

- **Block/log** `LEE_DC` → `LEE_ATTACK_NET`, IPv4 any.
- **Block/log** `LEE_DC` → `LEE_MANAGEMENT_NET`, IPv4 any.
- **Block/log** `LEE_DC` → `LEE_CLIENT_NET`, IPv4 any (does not prohibit stateful responses to client-initiated sessions).
- **Pass/log** `LEE_DC` → **not** `NON_PUBLIC_IPV4`, TCP 443 (public HTTPS egress). Destination inversion was explicitly verified in the rule editor.
- **Pass/log** `LEE_DC` → `APPROVED_DNS`, TCP/UDP 53.
- **Pass/log** `LEE_DC` → `APPROVED_NTP`, UDP 123.
- Original default LAN IPv4/IPv6 broad-allow rules **disabled**. Earlier anti-lockout allowance was disabled after explicit management access was verified.

### CLIENT interface

Enabled CLIENT-network → `LEE_DC` permissions:

| Protocol | Destination port | Purpose |
|---|---|---|
| TCP/UDP | 53 | DNS |
| TCP/UDP | 389 | LDAP / CLDAP |
| TCP/UDP | 88 | Kerberos |
| TCP | 445 | SMB / SYSVOL access |
| TCP | 135 | RPC Endpoint Mapper |
| UDP | 123 | Domain NTP |
| TCP | 49152–65535 | Windows dynamic RPC to the domain controller |

A former **temporary** client `192.168.20.100` → DC TCP 49679 validation rule remains **disabled**, superseded by the dynamic RPC rule. The broad dynamic RPC range is a functional AD accommodation scoped to the DC, not a claim that every port in that range is always necessary. Revisit it if tighter RPC allocation is configured in a later hardening exercise.

### MANAGEMENT interface

- **Pass:** Ubuntu `192.168.40.10` → `This Firewall (self)` TCP 443 for pfSense WebGUI administration.
- Other unapproved MANAGEMENT-originating traffic is default-denied; pfSense WebGUI access was verified.

### ATTACK interface

- **Pass/log:** Kali `192.168.30.10` → ATTACK interface `192.168.30.1`, ICMP echo request only. This optional diagnostic allowance is not a route into other security zones.
- All other unapproved ATTACK-originating flows are default-denied.

### WAN and Floating

- WAN displays automatically generated block-private and block-bogon controls; no user-defined WAN pass rules were present in the captured view.
- **No floating rules configured.**

### Aliases

| Alias | Definition |
|---|---|
| `LEE_DC` | 192.168.10.10 |
| `LEE_CLIENT_NET` | 192.168.20.0/24 |
| `LEE_ATTACK_NET` | 192.168.30.0/24 |
| `LEE_MANAGEMENT_NET` | 192.168.40.0/24 |
| `APPROVED_DNS` | 8.8.8.8; 8.8.4.4 |
| `APPROVED_NTP` | 129.6.15.28; 132.163.96.3 |
| `NON_PUBLIC_IPV4` | 10.0.0.0/8; 172.16.0.0/12; 192.168.0.0/16; 127.0.0.0/8; 169.254.0.0/16; 100.64.0.0/10; 0.0.0.0/8; 192.0.0.0/24; 192.0.2.0/24; 198.18.0.0/15; 198.51.100.0/24; 203.0.113.0/24; 224.0.0.0/4; 240.0.0.0/4 |

**Review note:** `NON_PUBLIC_IPV4` is a custom exclusion list, not a guarantee that every remaining destination is publicly routable or safe. Maintain it as network requirements evolve.

## 4. Validation evidence

### 4.1 Domain controller health

Following migration to 192.168.10.10, Active Directory Domain Services, DNS and KDC were operational. `dcdiag` DNS/connectivity tests passed, DC location and LDAP/Kerberos DNS records resolved, and external forwarding to approved DNS resolvers worked. The DC synchronized with configured NIST time sources.

### 4.2 Domain client functionality

- `nltest /dsgetdc:livingenterprise.lab /force` found `LEE-WIN-SRV-01` at 192.168.10.10.
- `nltest /sc_query:livingenterprise.lab` returned `NERR_Success` after required traffic was permitted.
- Client-to-DC tests confirmed TCP 389, 88, 445 and a dynamic RPC endpoint (TCP 49679) reachable when the corresponding policies were enabled.
- `gpupdate /target:computer /force` completed successfully.
- `gpresult /scope computer /r` confirmed `Default Domain Policy` applied from `LEE-WIN-SRV-01.livingenterprise.lab` on October 14.
- Domain user `LEE\alicia.carter` signed in successfully. `klist` showed five Kerberos tickets, including a TGT and LDAP/CIFS service tickets using AES-256.
- `gpresult /scope user /r` identified the domain user and her Finance group. No user GPO was listed as applied; this is not, by itself, a failure.
- The client initially reported `Local CMOS Clock` and unsynchronized time. After `w32tm /resync /rediscover`, its source became `LEE-WIN-SRV-01.livingenterprise.lab`, stratum 3, with a successful sync to 192.168.10.10.
- `gpresult /scope computer /r` returned Access Denied in an unelevated domain-user session and succeeded in an elevated session; this reflected access rights, not a Group Policy connectivity failure.

### 4.3 Attack zone isolation

Kali was configured with NetworkManager profile `LEE-ATTACK-STATIC`, address 192.168.30.10/24 and default route via 192.168.30.1. Neighbor discovery resolved the pfSense gateway MAC, establishing local link reachability even before ICMP was permitted.

From Kali, connection attempts timed out to:

- SERVER 192.168.10.10:445
- CLIENT 192.168.20.100:445
- MANAGEMENT 192.168.40.10:22

Matching pfSense ATTACK default-deny entries were captured for each protected destination. Kali ICMP echo to pfSense 192.168.30.1 initially failed with matching denies; after the narrowly scoped ICMP rule, ping succeeded (4/4 replies), and the firewall logged the pass.

### 4.4 Management zone isolation

From Ubuntu 192.168.40.10, ICMP ping and TCP 445 to DC 192.168.10.10 failed, with matching MANAGEMENT interface default-deny logs. Explicit Ubuntu → pfSense TCP 443 management access continued to work.

### 4.5 Background traffic and DNS note

Kali repeatedly attempted DNS to VMware NAT gateway 192.168.133.2:53 and was blocked on ATTACK. NetworkManager profile `LEE-ATTACK-STATIC` showed no explicit IPv4 DNS and no device-reported IP4.DNS; `/etc/resolv.conf` nevertheless contained `nameserver 192.168.133.2` and `search localdomain`. The file was generated by NetworkManager but appeared as a regular file. The team deliberately **did not modify DNS** without a demonstrated need. Kali's successful IP-based network tests do not depend on DNS resolution.

The DC also generated blocked outbound TCP 80 attempts. These were recorded but not treated as a reason to widen egress without a service requirement.

## 5. Troubleshooting and engineering decisions

1. **Flat network migration required rebuilding service assumptions.** The DC's previous address and the client's previous DNS configuration had to be reconciled with the new networks.
2. **Port reachability is not application success.** TCP 445 connectivity alone did not prove SYSVOL/GPO health. Application-level policy processing was validated later.
3. **Dynamic RPC is distinct from RPC Endpoint Mapper.** TCP 135 enables endpoint discovery; the service can then direct the client to a dynamic high TCP port. A temporary TCP 49679 test helped isolate the missing dependency. The temporary rule was subsequently disabled after the DC-scoped dynamic range was enabled.
4. **Local and domain sessions yield different identity evidence.** The local administrator had no Kerberos tickets in that session; the domain user later obtained the expected Kerberos TGT and service tickets.
5. **Time synchronization matters for domain authentication.** The client recovered its domain time source after resynchronization; successful `w32tm` output was captured.
6. **A failed ping does not establish a failed network.** Neighbor resolution and pfSense logs distinguished physical/link connectivity from firewall policy decisions.
7. **Default-deny logs can prove intended isolation.** Source, destination, protocol, interface and matched rule were correlated with the initiated tests.
8. **Avoid unnecessary permissions.** Unsolicited DNS attempts, Internet TCP 80/443 and other background traffic were not used to justify blanket allow rules.
9. **Firewall descriptions are documentation, not enforcement.** Source, destination, protocol, inversion, port, enabled state and ordering were inspected directly.

## 6. Evidence inventory

The following artifacts were captured during the phase and should be stored in the project's evidence directory or linked from `v6a`:

- Screenshots of pfSense SERVER, CLIENT, MANAGEMENT, ATTACK, WAN and Floating rule tabs.
- Screenshot of SERVER public-HTTPS rule editor verifying destination inversion (`! NON_PUBLIC_IPV4`).
- Screenshot of MANAGEMENT TCP 443 rule editor.
- Screenshot of firewall aliases and their definitions.
- Screenshot showing temporary TCP 49679 validation rule **disabled**.
- Screenshots of ATTACK → SERVER, CLIENT and MANAGEMENT deny logs.
- Screenshot of ATTACK → pfSense ICMP pass log.
- Screenshots of MANAGEMENT → SERVER ICMP and TCP 445 deny logs.
- CLI evidence of client computer GPO, domain-user Kerberos tickets, domain time synchronization, and Kali address/route/neighbor information.

**Evidence hygiene:** Screenshots and logs should retain timestamps, IPs, protocol/port, rule names and outcomes. Redact secrets and unrelated private details before publication.

## 7. Recovery baseline

- pfSense configuration backup **saved and renamed**. Its long-term second-copy location is not yet finalized.
- VM snapshots **completed** for pfSense, Windows Server/DC, Windows client, Ubuntu management, and Kali attack host.
- Keep configuration XML private; it can contain sensitive settings or password material. Do not commit it to the public repository.
- Snapshots are lab rollback points, **not** a replacement for supported Active Directory/system-state backups. Restoring a domain controller snapshot requires special care, especially after other VMs have continued changing state.

## 8. Closeout checklist

- [x] Define segmented target-state topology and zone addressing.
- [x] Deploy and configure pfSense interfaces.
- [x] Migrate DC, client, Ubuntu and Kali into intended zones.
- [x] Validate DNS, AD discovery, Kerberos, secure channel, Group Policy and domain time.
- [x] Implement narrowly scoped client-to-DC access and DC egress rules.
- [x] Validate management WebGUI access and protected-zone denials.
- [x] Validate Kali isolation with corresponding pfSense logs.
- [x] Capture all interface-rule tabs, Floating status and alias definitions.
- [x] Disable temporary TCP 49679 validation rule.
- [x] Save and rename pfSense configuration backup.
- [x] Take snapshots of all five virtual machines.
- [ ] Store an additional encrypted copy of the pfSense backup outside the VM environment.
- [ ] Create a supported DC backup strategy before future major AD changes.

**Closeout decision:** Phase 2 implementation and baseline evidence are complete. The two unchecked resilience improvements are tracked follow-ups, not blockers to proceeding with Phase 3.

## 9. Relationship to original phase plan

The original repository contains Phase 5 `v11-pfsense-deployment.md` and `v12-segmented-environment.md`. Their implementation objectives were accomplished during the Phase 2 architecture/segmentation work. Keep those documents as historical planning references and annotate that the implemented design and evidence live in Phase 2 `v6a` and this closeout. Do **not** claim Phase 3 Wazuh or Phase 4 simulations have already occurred.

Suggested navigation:

- Phase 2 README → `v4-enterprise-network-design.md`, `v5-network-segmentation-planning.md`, `v6-active-directory-network-preparation.md`, `v6a-...` implementation narrative, **this closeout**.
- Phase 5 README → historical v11/v12 cross-reference to Phase 2, rather than duplicating instructions.
- Main README → revised phase roadmap and next-stage link.

## 10. Handoff to Phase 3 — Security Monitoring

**Next planned milestone:** deploy Wazuh in a **dedicated MONITORING zone**, not MANAGEMENT.

Before provisioning the monitoring VM:

1. Define its address space, pfSense interface/VMnet, resource allocation and access model.
2. Specify which systems send telemetry (DC, client, Ubuntu, pfSense, later Kali), over which exact ports and protocols.
3. Define administration access to the Wazuh dashboard/API and keep it separate from agent ingestion.
4. Add only required, logged firewall exceptions and verify negative tests remain blocked.
5. Establish time normalization (UTC for SIEM correlation), retention, log integrity and an initial healthy-telemetry baseline.
6. Proceed to controlled event generation, analysis and detection engineering in later versions.

## 11. Final engineering statement

Phase 2 established a functioning, segmented enterprise foundation in which domain dependencies are explicitly permitted, management access is scoped, attacker-originated connections to protected zones are denied, and firewall decisions can be demonstrated through correlated logs. The environment has been documented and checkpointed for the next stage: telemetry collection and detection.
