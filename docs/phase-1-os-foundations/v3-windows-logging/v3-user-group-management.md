# Active Directory User, Group, and Resource Authorization

## Overview

The Living Enterprise Environment (LEE) uses Active Directory users, security groups, Organizational Units, privileged identities, and resource permissions to model enterprise identity and access management.

The objective of this portion of the Windows environment was to move beyond creating individual domain accounts and establish a structured authorization model that can support administration, security monitoring, and future attack simulation.

The design separates:

- User identity
- Organizational placement
- Role membership
- Privileged identity
- Resource authorization

This provides a baseline for understanding how Active Directory identity and group membership influence access to enterprise resources.

---

## Organizational Unit Structure

Custom Organizational Units were created to separate users, computers, servers, service accounts, administrative identities, and security groups.

```text
livingenterprise.lab
│
├── LEE-Users
│   ├── IT
│   ├── Security
│   ├── Finance
│   ├── Human-Resources
│   └── Operations
│
├── LEE-Groups
│
├── LEE-Computers
│   └── Test-Workstations
│
├── LEE-Servers
│   ├── Member-Servers
│   └── Application-Servers
│
├── LEE-Service-Accounts
│
└── LEE-Admins
    ├── Administrative-Users
    └── Privileged-Groups
