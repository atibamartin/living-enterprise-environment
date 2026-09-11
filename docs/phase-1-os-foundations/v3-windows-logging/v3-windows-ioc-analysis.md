# Windows Authentication Investigation

## Overview

The Living Enterprise Environment (LEE) uses Active Directory to provide centralized authentication for Windows users, computers, and network resources.

This investigation examined how a domain-joined Windows workstation discovers Active Directory authentication services, how users authenticate to the domain, and how authentication activity appears in the Windows Security log.

The objective was to connect the user experience of signing in and accessing enterprise resources with the underlying DNS, Kerberos, and Windows authentication telemetry available to a security analyst.

---

## Environment

| System | Role | Address |
|---|---|---|
| `LEE-WIN-SRV-01` | Domain Controller / DNS / KDC | `192.168.133.130` |
| `LEE-WIN-CLT-01` | Windows 11 Enterprise Client | `192.168.133.131` |
| `livingenterprise.lab` | Active Directory Domain | — |
| `LEE` | NetBIOS Domain | — |

The Windows client was configured to use `LEE-WIN-SRV-01` as its DNS server.

This allows the workstation to locate Active Directory services through DNS rather than relying on manually configured authentication servers.

---

## Active Directory Service Discovery

Before domain authentication can occur, a Windows client must locate the services responsible for the domain.

DNS SRV records were used to validate service discovery.

LDAP discovery:

```powershell
nslookup -type=SRV _ldap._tcp.dc._msdcs.livingenterprise.lab
