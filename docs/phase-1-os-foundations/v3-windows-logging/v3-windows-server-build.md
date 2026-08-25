# LEE Windows Server Build

## Overview

This document describes the deployment, configuration, and validation of the
Windows Server infrastructure supporting the Living Enterprise Environment (LEE).

LEE-WIN-SRV-01 was deployed as the first Windows Server in the environment and
subsequently promoted as the first Domain Controller for the
`livingenterprise.lab` Active Directory forest.

The build establishes the identity, DNS, authentication, Group Policy, and
Windows logging foundation required for later security monitoring, authentication
investigation, IOC analysis, and attack-simulation exercises.

---

## System Architecture

### Server

| Component | Configuration |
|---|---|
| Hostname | LEE-WIN-SRV-01 |
| Operating System | Windows Server 2025 Standard Evaluation |
| Installation | Desktop Experience |
| IPv4 Address | 192.168.133.130 |
| Subnet | 192.168.133.0/24 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 192.168.133.2 |
| Preferred DNS | 192.168.133.130 |
| AD Forest | livingenterprise.lab |
| AD Domain | livingenterprise.lab |
| NetBIOS Domain | LEE |
| AD Site | Default-First-Site-Name |

LEE-WIN-SRV-01 provides the initial Active Directory Domain Services and
AD-integrated DNS infrastructure for the environment.

---

## Operating System Selection

Windows Server 2025 Standard Evaluation with Desktop Experience was selected
for the initial Windows infrastructure deployment.

Desktop Experience was retained to support:

- Active Directory administration
- DNS administration
- Group Policy management
- Event Viewer analysis
- Security investigation exercises
- Visual documentation of administrative workflows

Although production servers may use reduced-GUI deployment models where
appropriate, Desktop Experience provides additional visibility for the
educational and investigative objectives of LEE.

---

## Network Baseline

Initial connectivity testing validated communication between the Windows,
Ubuntu, and Kali systems on the VMware virtual network.

### Systems

| System | IPv4 Address | Purpose |
|---|---|---|
| LEE-WIN-SRV-01 | 192.168.133.130 | Windows Server / Domain Controller / DNS |
| LEE-Ubuntu-01 | 192.168.133.129 | Linux server |
| Kali Linux | 192.168.133.128 | Security testing system |

During initial deployment, LEE-WIN-SRV-01 obtained network configuration
through DHCP.

Before Domain Controller promotion, the server was transitioned to a static
IPv4 configuration to prevent address changes from disrupting Active Directory
and DNS services.

The final DNS client configuration points LEE-WIN-SRV-01 to its own
AD-integrated DNS service:

`192.168.133.130`

### Network Validation Finding

Early testing identified asymmetric ICMP connectivity:

- Windows Server → Ubuntu Server: Successful
- Ubuntu Server → Windows Server: Failed

Investigation determined that the Windows network profile and host firewall
configuration were responsible for the behavior rather than a failure of
Layer 3 connectivity.

This demonstrated an important troubleshooting distinction:

> Successful network routing does not imply that a host-based firewall permits
> a particular protocol.

---

# Active Directory Deployment

## Design

A new Active Directory forest was created:

`livingenterprise.lab`

The deployment established:

- Forest: livingenterprise.lab
- Domain: livingenterprise.lab
- NetBIOS domain: LEE
- Forest functional level: Windows Server 2025
- Domain functional level: Windows Server 2025
- Global Catalog: Enabled
- DNS Server: Enabled
- DNS Delegation: Not configured

The shorter `LEE` NetBIOS namespace was selected while retaining the more
descriptive `livingenterprise.lab` DNS namespace.

### Domain Controller

`LEE-WIN-SRV-01.livingenterprise.lab`

The server is the first Domain Controller in the forest and therefore initially
holds all FSMO roles:

- Schema Master
- Domain Naming Master
- PDC Emulator
- RID Master
- Infrastructure Master

---

# Active Directory Architecture Validation

PowerShell validation confirmed:

- Domain distinguished name: `DC=livingenterprise,DC=lab`
- Forest root: `livingenterprise.lab`
- Domain mode: Windows2025Domain
- Forest mode: Windows2025Forest
- Global Catalog: LEE-WIN-SRV-01
- Domain Controller IPv4: 192.168.133.130
- LDAP port: TCP/389
- LDAPS port: TCP/636

Core services were also validated as running:

- Active Directory Domain Services (NTDS)
- DNS Server
- Netlogon
- Kerberos Key Distribution Center (KDC)

---

# DNS and Domain Controller Discovery

Active Directory depends on DNS not only for hostname resolution but also for
service discovery.

Host resolution successfully returned:

`LEE-WIN-SRV-01.livingenterprise.lab → 192.168.133.130`

LDAP service discovery was validated through:

`_ldap._tcp.dc._msdcs.livingenterprise.lab`

The SRV record identified:

- Target: LEE-WIN-SRV-01.livingenterprise.lab
- Port: TCP/389

Kerberos discovery was validated through:

`_kerberos._tcp.dc._msdcs.livingenterprise.lab`

The SRV record identified:

- Target: LEE-WIN-SRV-01.livingenterprise.lab
- Port: TCP/88

These tests demonstrated the role of DNS SRV records in allowing domain members
to dynamically locate Active Directory services rather than relying on
hard-coded Domain Controller addresses.

---

# Kerberos Authentication Validation

The Kerberos Key Distribution Center service was confirmed:

- Status: Running
- Startup: Automatic
- Dependency on Active Directory Domain Services confirmed

The automatically created `krbtgt` domain account was also identified.

Kerberos functionality was validated using the local ticket cache.

The Administrator session contained:

- A Ticket Granting Ticket (TGT) for `krbtgt/LIVINGENTERPRISE.LAB`
- A service ticket for `host/lee-win-srv-01.livingenterprise.lab`
- AES-256 ticket encryption
- LEE-WIN-SRV-01 identified as the issuing KDC

This provided functional validation beyond confirming that the KDC service was
merely running.

---

# Active Directory Object Structure

Post-promotion inspection identified the automatically generated directory
structure.

The Domain Controller computer object resides in:

`OU=Domain Controllers,DC=livingenterprise,DC=lab`

The distinction between Active Directory containers and Organizational Units
was reviewed as part of the deployment.

Organizational Units provide administrative and Group Policy boundaries,
whereas default containers such as `CN=Users` and `CN=Computers` do not provide
the same Group Policy linking model.

This distinction will guide the custom OU architecture used during subsequent
identity-management phases.

---

# Built-In Security Groups

Domain creation automatically generated privileged and operational groups
including:

- Domain Admins
- Enterprise Admins
- Schema Admins
- Domain Controllers
- Group Policy Creator Owners
- DnsAdmins
- Backup Operators
- Account Operators
- Server Operators
- Event Log Readers
- Remote Management Users
- Protected Users

Reviewing these groups established a baseline for future privilege analysis.

During later security investigations, group membership will provide critical
context when answering:

> What could this account do if compromised?

---

# Group Policy and SYSVOL

Active Directory automatically created:

- Default Domain Policy
- Default Domain Controllers Policy

PowerShell validation returned:

`Default Domain Policy`
GUID: `31b2f340-016d-11d2-945f-00c04fb984f9`

`Default Domain Controllers Policy`
GUID: `6ac1786c-016f-11d2-945f-00c04fb984f9`

The corresponding Group Policy Templates were verified under SYSVOL.

This demonstrated the two-part architecture of a Group Policy Object:

Active Directory stores the directory-side Group Policy Container (GPC), while
SYSVOL stores the file-based Group Policy Template (GPT).

Group Policy Management confirmed:

- Default Domain Policy linked at the domain level
- Default Domain Controllers Policy linked to the Domain Controllers OU

`gpresult` later confirmed both policies were successfully applied to
LEE-WIN-SRV-01.

---

# Authoritative Time Architecture

## Finding

Post-deployment validation identified that the forest-root PDC Emulator was
initially sourcing time from:

`Local CMOS Clock`

Because the PDC Emulator sits at the top of the Active Directory time
hierarchy, relying solely on its local hardware clock was not selected as the
desired long-term design.

Reliable time synchronization is important for:

- Kerberos authentication
- Replay protection
- Security-event correlation
- Incident timeline reconstruction
- Consistent logging across systems

## Remediation

LEE-WIN-SRV-01 was configured as a reliable domain time source using redundant
external NTP peers:

- 0.pool.ntp.org
- 1.pool.ntp.org
- 2.pool.ntp.org
- 3.pool.ntp.org

Windows Time was configured for manual NTP operation on the PDC Emulator.

VMware Tools periodic guest time synchronization was independently verified as
disabled, preventing it from competing with Windows Time for control of the
guest clock.

## Validation

Following configuration:

- Four NTP peers were detected.
- All four peers reported Active status.
- Peer mode was confirmed as NTP Client.
- The PDC synchronized successfully with an external peer.
- Windows reported Stratum 2 synchronization.
- A five-sample stripchart showed a stable offset of approximately 0.56 seconds.

After a subsequent system boot, Windows initially reported `Local CMOS Clock`
while the time service initialized.

Configuration inspection confirmed that:

- `Type: NTP` persisted.
- All four manual peers persisted.
- All four peers were active.
- No configuration reapplication was required.

After the service completed initialization, Windows automatically selected
`0.pool.ntp.org` as its active source.

This established an operational characteristic of the lab:

> Domain Controller startup should be followed by a short convergence period
> before authentication testing, attack simulation, or security telemetry
> collection begins.

---

# Event Log Review

Post-deployment Event Viewer review identified historical Kerberos KDC and
Netlogon events.

Netlogon Event ID 5775 recorded failed attempts to dynamically delete DNS
records associated with:

- `livingenterprise.lab`
- `ForestDnsZones.livingenterprise.lab`

Subsequent DNS, Netlogon, SRV-record, and DCDIAG validation showed no active
service impairment.

Historical KDC Event ID 7 events were also identified. Current-state testing
subsequently demonstrated successful KDC operation and Kerberos ticket issuance.

Windows Time warnings provided additional evidence of the time synchronization
behavior observed during startup. The system reported periods without a usable
time provider and a historical DNS resolution failure involving
`time.windows.com`.

Following the authoritative NTP configuration and startup convergence period,
the server successfully synchronized with the configured NTP pool.

Historical events were retained rather than cleared to preserve investigative
telemetry.

---

# Domain Controller Health Validation

Domain Controller diagnostics were performed using `dcdiag`.

The server passed:

- Connectivity
- Advertising
- FrsEvent
- DFSREvent
- SysVolCheck
- KccEvent
- KnowsOfRoleHolders
- MachineAccount
- NCSecDesc
- NetLogons
- ObjectsReplicated
- Replications
- RidManager
- Services
- SystemLog
- VerifyReferences
- LocatorCheck
- Intersite

Directory partition tests also completed successfully for:

- ForestDnsZones
- DomainDnsZones
- Schema
- Configuration
- livingenterprise

Dedicated DNS diagnostics using:

`dcdiag /test:dns`

passed for both LEE-WIN-SRV-01 and `livingenterprise.lab`.

`repadmin /replsummary` returned no replication partners, which is expected
because LEE-WIN-SRV-01 is currently the environment's only Domain Controller.

---

# Security and Engineering Observations

Several engineering principles were reinforced during the deployment:

1. **Service installation is not service validation.** AD DS and DNS were
   functionally tested after installation rather than assumed healthy because
   their roles installed successfully.

2. **Active Directory depends heavily on DNS.** DNS provides service discovery
   for LDAP, Kerberos, and Domain Controllers through SRV records.

3. **Authentication depends on time.** A functioning KDC alone does not
   guarantee reliable Kerberos authentication if domain clocks become
   excessively skewed.

4. **Configuration state and runtime state are different.** Windows Time
   configuration remained persistent after reboot even though the active source
   temporarily reported the Local CMOS Clock while synchronization converged.

5. **Historical errors require context.** Event Viewer errors were correlated
   against current functional tests before determining whether they represented
   active failures.

6. **Privileged identity must be evaluated by capability.** Built-in group
   membership provides essential context for determining the potential impact
   of account compromise.

7. **Centralized administration changes blast radius.** Active Directory and
   Group Policy provide powerful defensive administration capabilities, but
   compromise of privileged control paths could affect many systems.

---

# Final Validation State

At completion of the Windows Server and Active Directory baseline:

| Component | Status |
|---|---|
| Static network configuration | Validated |
| AD DS | Validated |
| DNS | Validated |
| AD-integrated DNS | Validated |
| LDAP discovery | Validated |
| Kerberos discovery | Validated |
| KDC service | Validated |
| Kerberos ticket issuance | Validated |
| SYSVOL | Validated |
| Group Policy | Validated |
| Netlogon | Validated |
| FSMO roles | Validated |
| Domain Controller diagnostics | Passed |
| DNS diagnostics | Passed |
| External NTP | Validated |
| VMware time synchronization | Disabled as intended |
| Event log review | Completed |

## Baseline Status

**LEE-WIN-SRV-01 Active Directory Baseline: VALIDATED**

This state establishes the trusted Windows identity and authentication
foundation for subsequent Living Enterprise Environment phases.
