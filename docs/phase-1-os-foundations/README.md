# Windows Enterprise Integration & Logging

## Overview

Version 3 of the Living Enterprise Environment (LEE) introduced a Windows enterprise environment into the lab.

The objective was to move beyond isolated systems and create a small Active Directory environment where identity, authentication, authorization, resource access, and Windows security telemetry could be observed together.

The environment now includes a Windows Server domain controller and a domain-joined Windows 11 workstation. Active Directory users, organizational units, security groups, privileged identities, shared resources, and Windows auditing were configured to provide a baseline for future security monitoring and attack simulation.

This phase establishes the Windows foundation that later LEE phases will monitor, attack, and defend.

---

## Objectives

The primary objectives were to:

- Deploy Windows Server and Active Directory Domain Services.
- Configure DNS and Kerberos-based domain authentication.
- Create an enterprise-style OU, user, and security group structure.
- Separate standard and privileged administrative identities.
- Deploy and domain-join a Windows 11 client.
- Implement role-based resource authorization using AGDLP.
- Configure a restricted administrative SMB resource.
- Establish Windows Security auditing for authentication and file activity.
- Generate controlled authorized and unauthorized activity.
- Analyze Windows events from a SOC analyst perspective.
- Establish known-good behavior for future detection engineering.

---

## Environment

| System | Role | Address |
|---|---|---|
| `LEE-WIN-SRV-01` | Windows Server / Domain Controller / DNS | `192.168.133.130` |
| `LEE-WIN-CLT-01` | Windows 11 Enterprise Domain Client | `192.168.133.131` |
| `livingenterprise.lab` | Active Directory Domain | — |
| `LEE` | NetBIOS Domain | — |

The Windows systems currently operate on the VMware NAT network used by the Living Enterprise Environment.

---

## Architecture

```text
                     livingenterprise.lab
                              │
                    LEE-WIN-SRV-01
                   192.168.133.130
                              │
              ┌───────────────┼───────────────┐
              │               │               │
             AD DS           DNS          Kerberos
              │               │               │
              └───────────────┼───────────────┘
                              │
                              │ Domain Authentication
                              │
                    LEE-WIN-CLT-01
                   192.168.133.131
                              │
                              │ SMB
                              ▼
                  \\LEE-WIN-SRV-01\
                       ServerTools
