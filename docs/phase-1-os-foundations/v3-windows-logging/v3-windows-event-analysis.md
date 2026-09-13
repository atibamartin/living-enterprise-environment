# Windows Authentication Investigation

## Overview

This lab investigated successful and failed Windows domain authentication within the Living Enterprise Environment (LEE).

A dedicated test account was created in Active Directory and used from a domain-joined Windows 11 workstation to authenticate to the domain controller. A successful network authentication was first established as a baseline. Controlled incorrect-password attempts were then generated and investigated using Windows Security logs and Active Directory account data.

The exercise focused on identifying the user, source system, authentication method, logon type, failure reason, and supporting Kerberos activity associated with the authentication attempts.

---

## Lab Environment

| System         | Role                                            | IP Address      |
| -------------- | ----------------------------------------------- | --------------- |
| LEE-WIN-SRV-01 | Domain Controller / DNS / Authentication Target | 192.168.133.130 |
| LEE-WIN-CLT-01 | Domain-Joined Windows 11 Client                 | 192.168.133.131 |

**Domain:** `livingenterprise.lab`

**Client logged-on user:** `LEE\alicia.carter`

**Dedicated test account:** `LEE\auth.test`

The `auth.test` account was created under:

```text
OU=Security
OU=LEE-Users
DC=livingenterprise
DC=lab
```

The dedicated account allowed authentication activity to be generated without using an administrative or normal production-style user account.

---

# 1. Pre-Investigation Validation

Before generating authentication failures, the domain password and lockout policy was reviewed.

```powershell
Get-ADDefaultDomainPasswordPolicy | Select-Object `
LockoutThreshold,
LockoutDuration,
LockoutObservationWindow,
MinPasswordLength,
PasswordHistoryCount
```

Observed configuration:

```text
LockoutThreshold         : 0
LockoutDuration          : 00:10:00
LockoutObservationWindow : 00:10:00
MinPasswordLength        : 7
PasswordHistoryCount     : 24
```

A `LockoutThreshold` of `0` indicated that automatic account lockout was not currently enabled by the default domain policy.

This was important to establish before deliberately generating incorrect-password attempts.

---

# 2. Test Account Validation

The newly created account was inspected from the domain controller.

```powershell
Get-ADUser auth.test -Properties Enabled,LockedOut,PasswordLastSet,PasswordExpired,BadPwdCount,LastBadPasswordAttempt,UserPrincipalName |
Select-Object SamAccountName,UserPrincipalName,Enabled,LockedOut,PasswordLastSet,PasswordExpired,BadPwdCount,LastBadPasswordAttempt
```

The account was confirmed to be:

```text
SamAccountName    : auth.test
UserPrincipalName : auth.test@livingenterprise.lab
Enabled           : True
LockedOut         : False
PasswordExpired   : False
```

The object's location in Active Directory was also verified:

```powershell
Get-ADUser auth.test |
Select-Object Name,SamAccountName,DistinguishedName
```

Result:

```text
Name               : Auth Test
SamAccountName     : auth.test
DistinguishedName  : CN=Auth Test,OU=Security,OU=LEE-Users,
                     DC=livingenterprise,DC=lab
```

This established that the identity existed, was enabled, and was available for authentication testing.

---

# 3. Client Connectivity Baseline

Authentication testing was performed from `LEE-WIN-CLT-01` while logged on as the normal domain user:

```text
LEE\alicia.carter
```

DNS resolution was tested first:

```cmd
nslookup LEE-WIN-SRV-01
```

Result:

```text
Name:    LEE-WIN-SRV-01.livingenterprise.lab
Address: 192.168.133.130
```

Network connectivity was then validated:

```cmd
ping LEE-WIN-SRV-01
```

Result:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

These checks established that subsequent authentication failures could not immediately be attributed to DNS resolution or basic network connectivity problems.

---

# 4. Successful Authentication Baseline

A network authentication was initiated from `LEE-WIN-CLT-01` to the IPC$ share on `LEE-WIN-SRV-01`.

The User Principal Name (UPN) format was used:

```cmd
net use \\LEE-WIN-SRV-01\IPC$ /user:auth.test@livingenterprise.lab
```

