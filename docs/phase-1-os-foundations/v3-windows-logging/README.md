# v3 — Windows Integration

## Living Enterprise Environment

Version 3 expands the Living Enterprise Environment from a Linux-focused authentication lab into a small Windows enterprise environment built around Active Directory.

The purpose of this phase was not only to deploy Windows Server and join a client to a domain, but to build enough realistic enterprise infrastructure to support future authentication analysis, threat simulation, SIEM monitoring, and detection engineering.

By the end of v3, the environment included:

```text
Windows Server Domain Controller
        +
Active Directory Domain Services
        +
DNS
        +
Kerberos
        +
Organizational Units
        +
Security Groups
        +
Privileged Accounts
        +
File Shares and Access Control
        +
Windows 11 Client
        +
Linux/Kali Integration
        +
Windows Authentication Analysis
        +
IOC Investigation
```

This phase became the foundation for the Windows portion of the Living Enterprise Environment.

---

# Environment

## Domain

```text
Domain: livingenterprise.lab
NetBIOS Name: LEE
```

## Windows Server

```text
Hostname: LEE-WIN-SRV-01
IP Address: 192.168.133.130
Role:
- Active Directory Domain Controller
- DNS Server
```

## Windows Client

```text
Hostname: LEE-WIN-CLT-01
IP Address: 192.168.133.131
DNS Server: 192.168.133.130
```

## Kali Linux

```text
Hostname: LEE-Kali-01
IP Address: 192.168.133.128
Role:
- Testing system
- Attack simulation system
- Authentication activity generator
```

## Additional Linux System

```text
LEE-Ubuntu-01
```

The environment runs primarily on VMware Workstation using VMware NAT networking.

---

# v3 Objectives

The primary objectives of v3 were to:

- Deploy a Windows Server system.
- Install Active Directory Domain Services.
- Create the `livingenterprise.lab` forest and domain.
- Deploy Active Directory-integrated DNS.
- Validate Kerberos and domain-controller functionality.
- Configure reliable time synchronization.
- Design an enterprise-style OU hierarchy.
- Create departmental users and security groups.
- Introduce privileged administrative accounts.
- Implement group-based resource access.
- Build and integrate a Windows 11 client.
- Generate and investigate Windows authentication activity.
- Analyze successful and failed network logons.
- Introduce Kali Linux as an external testing system.
- Extract indicators from Windows authentication telemetry.
- Develop initial detection hypotheses.

The broader goal was to build an environment that could later support:

```text
Threat Simulation
        ↓
Centralized Logging
        ↓
SIEM Analysis
        ↓
Detection Engineering
```

---

# 1. Windows Server Deployment

`LEE-WIN-SRV-01` was deployed as the primary Windows Server system for the environment.

The server became the central authentication and directory-services platform for the lab.

Initial build activities included:

- Windows Server installation
- VMware Tools installation
- static network configuration
- hostname configuration
- Windows updates and baseline configuration
- Active Directory role preparation

Snapshots were created throughout major build stages to provide recovery points.

Examples included:

```text
LEE-WIN-SRV-01-BaseInstall
LEE-WIN-SRV-01-Post-Promotion
```

---

# 2. Active Directory Domain Services

Active Directory Domain Services was installed and the server was promoted to the first domain controller in a new forest.

Domain:

```text
livingenterprise.lab
```

NetBIOS name:

```text
LEE
```

The domain controller became:

```text
LEE-WIN-SRV-01.livingenterprise.lab
```

The deployment included:

- Active Directory Domain Services
- DNS
- Global Catalog
- SYSVOL
- NTDS database
- Kerberos Key Distribution Center

The domain was configured using the Windows Server 2025 functional level.

---

# 3. Active Directory Validation

Following domain promotion, multiple validation activities were performed.

## Domain Information

PowerShell was used to inspect the domain:

```powershell
Get-ADDomain
```

This confirmed the domain distinguished name:

```text
DC=livingenterprise,DC=lab
```

and verified the primary Active Directory infrastructure.

---

## DNS Validation

The domain controller registered Active Directory DNS records.

Important records included:

```text
LEE-WIN-SRV-01
→ 192.168.133.130
```

LDAP service record:

```text
_ldap._tcp.dc._msdcs.livingenterprise.lab
```

Kerberos service record:

```text
_kerberos._tcp.dc._msdcs.livingenterprise.lab
```

These records allow domain clients to locate Active Directory services dynamically.

---

## Kerberos Validation

The Kerberos Key Distribution Center service was verified as:

```text
Running
Automatic
```

The built-in `krbtgt` account was also reviewed.

This account is used internally by Active Directory to sign and encrypt Kerberos Ticket Granting Tickets.

---

# 4. Domain Controller Health

Domain-controller health was validated using:

```powershell
dcdiag
```

Tests included:

```text
Connectivity
Advertising
DFSREvent
SysVolCheck
KccEvent
KnowsOfRoleHolders
MachineAccount
NCSecDesc
NetLogons
ObjectsReplicated
Replications
RidManager
Services
SystemLog
VerifyReferences
```

The domain controller successfully passed the validation tests.

This provided a known-good infrastructure baseline before additional systems and activity were introduced.

---

# 5. Time Synchronization

Time synchronization was treated as an important infrastructure component because technologies such as Kerberos depend heavily on accurate time.

Initial Windows Time Service warnings showed that the domain controller was unable to synchronize reliably.

The domain controller was later configured to use public NTP sources:

```text
0.pool.ntp.org
1.pool.ntp.org
2.pool.ntp.org
3.pool.ntp.org
```

The server successfully synchronized with an external source.

Example validated source:

```text
2.pool.ntp.org
```

The system reached:

```text
Stratum 2
```

Reliable time synchronization established a stronger foundation for:

- Kerberos authentication
- event-log correlation
- incident timelines
- future SIEM ingestion

---

# 6. Organizational Unit Design

An enterprise-style Organizational Unit structure was created.

Primary OUs included:

```text
LEE-Users
LEE-Computers
LEE-Servers
LEE-Service-Accounts
LEE-Admins
LEE-Groups
```

Departmental OUs were created beneath `LEE-Users`:

```text
IT
Security
Finance
HR
Operations
```

An administrative user OU was created beneath:

```text
LEE-Admins
```

as:

```text
Administrative-Users
```

This design separates users and systems by role and provides a foundation for future:

- Group Policy
- delegated administration
- access control
- attack simulation
- identity monitoring

---

# 7. Security Groups

Global security groups were created for each business function.

Examples:

```text
GG-IT-Users
GG-Security-Users
GG-Finance-Users
GG-HR-Users
GG-Operations-Users
```

Example user:

```text
Jordan Miles
SamAccountName: jordan.miles
```

was created within the Security OU and added to:

```text
GG-Security-Users
```

Group membership activity later provided useful Windows Security telemetry.

---

# 8. Privileged Account Model

A separate administrative account was created to demonstrate separation between standard and privileged identities.

Example:

```text
Marcus Reed - Admin
SamAccountName: marcus.reed-adm
```

The account was stored under:

```text
OU=Administrative-Users
OU=LEE-Admins
```

This structure models a common enterprise practice:

```text
Standard User Account
        ≠
Privileged Administrative Account
```

Using separate accounts reduces unnecessary privileged access and creates clearer audit trails.

Windows Security logs captured group membership changes involving privileged accounts.

Important events included:

```text
4728
Member added to a security-enabled global group

4729
Member removed from a security-enabled global group
```

These events will become useful during future identity-monitoring exercises.

---

# 9. Resource Access Model

A controlled file resource was created:

```text
C:\LEE-Shares\ServerTools
```

An SMB share named:

```text
ServerTools
```

was created.

The share was configured with:

```text
SMB encryption enabled
Access-Based Enumeration enabled
Offline caching disabled
```

A domain-local security group was created:

```text
DL-ServerTools-RW
```

NTFS permissions were configured so that the group received:

```text
Modify
```

access.

The final NTFS access model included:

```text
SYSTEM                    Full Control
BUILTIN\Administrators    Full Control
LEE\DL-ServerTools-RW     Modify
```

Inheritance was disabled to create a controlled permissions model.

This exercise introduced the relationship between:

```text
User
  ↓
Global Group
  ↓
Domain Local Group
  ↓
Resource Permission
```

and demonstrated a scalable enterprise access-control pattern.

---

# 10. Windows 11 Client Deployment

A Windows 11 workstation was created:

```text
LEE-WIN-CLT-01
```

The client received:

```text
IP Address:      192.168.133.131
Subnet:          /24
Gateway:         192.168.133.2
DNS Server:      192.168.133.130
```

VMware Tools was installed and validated.

Connectivity to the domain controller was tested using:

```text
ping
nslookup
```

The client successfully resolved:

```text
LEE-WIN-SRV-01.livingenterprise.lab
```

using Active Directory DNS.

This created a domain-aware Windows endpoint for later user authentication, policy, file-access, and detection exercises.

---

# 11. Windows Authentication Investigation

After the Windows environment was built, v3 moved from infrastructure deployment into security analysis.

The `auth.test` account was used to generate controlled authentication activity.

Kerberos authentication activity was observed including:

```text
A Kerberos authentication ticket (TGT) was requested.
```

with:

```text
Account Name:
auth.test

Supplied Realm:
LIVINGENTERPRISE.LAB
```

This provided hands-on exposure to Windows domain authentication telemetry.

The investigation expanded into:

- successful authentication
- failed authentication
- network logons
- NTLM
- Kerberos
- credential validation
- source identification
- event correlation

---

# 12. Kali Integration

Kali Linux was introduced as an external testing system.

System:

```text
LEE-Kali-01
192.168.133.128
```

Connectivity to the domain controller was confirmed.

A targeted Nmap scan identified expected Active Directory services:

```text
53    DNS
88    Kerberos
135   RPC
139   NetBIOS
389   LDAP
445   SMB
464   Kerberos password service
636   LDAPS
3268  Global Catalog
3269  Global Catalog SSL
```

This established Kali as the external activity-generation platform for future threat simulation.

---

# 13. Kali DNS Investigation

An initial DNS lookup from Kali failed.

```bash
nslookup LEE-WIN-SRV-01.livingenterprise.lab
```

Kali was using:

```text
192.168.133.2
```

which was the VMware NAT resolver.

That DNS server did not know the internal Active Directory domain.

The Active Directory DNS server was queried directly:

```bash
nslookup LEE-WIN-SRV-01.livingenterprise.lab 192.168.133.130
```

and successfully returned:

```text
192.168.133.130
```

The Active Directory LDAP SRV record was also successfully resolved.

This demonstrated:

```text
Connectivity
≠
Service Availability
≠
Name Resolution
```

and became an important troubleshooting lesson.

---

# 14. SMB Authentication Analysis

Kali was used to authenticate to the Windows Server SMB service.

The original goal was to generate a failed login.

However, the first authentication unexpectedly succeeded.

The command:

```bash
smbclient -L //192.168.133.130 -U 'LIVINGENTERPRISE\auth.test'
```

returned available SMB shares.

Windows telemetry confirmed:

```text
Event ID: 4624
Logon Type: 3
Account: auth.test
Source: LEE-KALI-01
Source IP: 192.168.133.128
Authentication: NTLMv2
```

Credential validation was also recorded:

```text
Event ID: 4776
Error Code: 0x0
```

This proved that Windows accepted the supplied credentials.

The unexpected result became part of the investigation rather than being discarded.

---

# 15. Failed Authentication Analysis

A known-invalid password was later supplied from Kali.

Kali returned:

```text
NT_STATUS_LOGON_FAILURE
```

Windows generated:

```text
Event ID: 4625
```

Relevant information included:

```text
Logon Type: 3
Account: auth.test
Source Workstation: LEE-KALI-01
Source IP: 192.168.133.128
Authentication: NTLM

Status:
0xC000006D

Substatus:
0xC000006A
```

Credential validation produced:

```text
Event ID: 4776
Error Code: 0xC000006A
```

The failed authentication could therefore be correlated across:

```text
Kali
   ↓
Windows Authentication
   ↓
Credential Validation
   ↓
Active Directory Account State
```

---

# 16. Active Directory Account-State Analysis

The `auth.test` account was examined using:

```powershell
Get-ADUser
