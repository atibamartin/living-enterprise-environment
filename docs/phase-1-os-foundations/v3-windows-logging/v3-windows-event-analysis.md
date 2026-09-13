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
net use \\LEE-WIN-SRV-01\IPC$ /user:auth.test@livingenterprise.lab *
```

The correct password was supplied interactively.

Result:

```text
The command completed successfully.
```

The authenticated SMB connection was then removed:

```cmd
net use \\LEE-WIN-SRV-01\IPC$ /delete
```

Verification:

```cmd
net use
```

Result:

```text
There are no entries in the list.
```

This established a known-good authentication baseline before controlled failures were generated.

---

# 5. Successful Logon Evidence — Event ID 4624

The Security log on `LEE-WIN-SRV-01` was searched for Event ID `4624`.

A matching event was identified for `auth.test`.

Relevant fields included:

```text
Event ID:               4624
Account Name:           auth.test
Account Domain:         LIVINGENTERPRISE.LAB
Logon Type:             3
Source Network Address: 192.168.133.131
Authentication Package: Kerberos
Logon Process:          Kerberos
```

### Interpretation

Event ID `4624` represents a successful logon.

`Logon Type 3` identifies the activity as a network logon, which was consistent with the `net use` authentication performed against the remote server.

The source address:

```text
192.168.133.131
```

corresponded to `LEE-WIN-CLT-01`.

The event therefore established the following relationship:

```text
LEE-WIN-CLT-01
192.168.133.131
        |
        | auth.test
        | Kerberos
        v
LEE-WIN-SRV-01
Successful Network Logon
Event ID 4624
Logon Type 3
```

---

# 6. Controlled Failed Authentication

After confirming that no existing SMB session remained, the same authentication command was executed again:

```cmd
net use \\LEE-WIN-SRV-01\IPC$ /user:auth.test@livingenterprise.lab *
```

An intentionally incorrect password was entered.

Result:

```text
System error 1326 has occurred.

The user name or password is incorrect.
```

The procedure was repeated once more after confirming that no SMB connection existed.

This produced two controlled incorrect-password authentication attempts from the same client and account.

---

# 7. Failed Logon Evidence — Event ID 4625

The Security log on `LEE-WIN-SRV-01` was searched for Event ID `4625`.

Both controlled failures produced events with the same primary characteristics.

### Failure #1

```text
Event ID:               4625
Account Name:           auth.test@livingenterprise.lab
Logon Type:             3

Failure Reason:         Unknown user name or bad password.
Status:                 0xC000006D
Sub Status:             0xC000006A

Workstation Name:       LEE-WIN-CLT-01
Source Network Address: 192.168.133.131
Source Port:            53139

Logon Process:          NtLmSsp
Authentication Package: NTLM
```

### Failure #2

```text
Event ID:               4625
Account Name:           auth.test@livingenterprise.lab
Logon Type:             3

Failure Reason:         Unknown user name or bad password.
Status:                 0xC000006D
Sub Status:             0xC000006A

Workstation Name:       LEE-WIN-CLT-01
Source Network Address: 192.168.133.131
Source Port:            62105

Logon Process:          NtLmSsp
Authentication Package: NTLM
```

---

# 8. Failure Code Analysis

The failed logons contained two particularly useful fields:

```text
Status:     0xC000006D
Sub Status: 0xC000006A
```

`0xC000006D` indicated that the authentication attempt failed because invalid credentials were supplied.

The more specific substatus:

```text
0xC000006A
```

identified an incorrect password.

This distinction was important because the general Windows message:

```text
Unknown user name or bad password.
```

does not by itself distinguish between a nonexistent account and an incorrect password.

The substatus provided the additional context needed to determine the actual reason for failure.

---

# 9. Correlating Repeated Authentication Failures

The two controlled failures shared the following attributes:

| Field          | Failure #1                                                              | Failure #2                                                              |
| -------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Event ID       | 4625                                                                    | 4625                                                                    |
| Account        | [auth.test@livingenterprise.lab](mailto:auth.test@livingenterprise.lab) | [auth.test@livingenterprise.lab](mailto:auth.test@livingenterprise.lab) |
| Logon Type     | 3                                                                       | 3                                                                       |
| Status         | 0xC000006D                                                              | 0xC000006D                                                              |
| Sub Status     | 0xC000006A                                                              | 0xC000006A                                                              |
| Workstation    | LEE-WIN-CLT-01                                                          | LEE-WIN-CLT-01                                                          |
| Source IP      | 192.168.133.131                                                         | 192.168.133.131                                                         |
| Authentication | NTLM                                                                    | NTLM                                                                    |
| Source Port    | 53139                                                                   | 62105                                                                   |

The source port changed between authentication attempts.

This is expected behavior because separate network connections can use different ephemeral source ports.

The changing port did not prevent the events from being correlated because the more significant attributes remained consistent:

```text
Same account
Same source workstation
Same source IP
Same destination
Same logon type
Same failure status
Same failure substatus
Same authentication package
```

Together, these attributes established a repeatable failed-authentication pattern.

---

# 10. Active Directory Account-State Correlation

The state of `auth.test` was checked after the controlled failures:

```powershell
Get-ADUser auth.test -Properties BadPwdCount,LastBadPasswordAttempt,LockedOut |
Select-Object SamAccountName,BadPwdCount,LastBadPasswordAttempt,LockedOut
```

Observed result:

```text
SamAccountName         : auth.test
BadPwdCount            : 2
LastBadPasswordAttempt : 9/12/2026 11:56:45 PM
LockedOut              : False
```

The two failed attempts were therefore reflected not only in Security Event ID `4625`, but also in the Active Directory account's bad-password state.

This provided another source of evidence supporting the investigation.

---

# 11. Kerberos Investigation — Event ID 4768

The successful authentication was investigated further by reviewing Kerberos activity.

Event ID `4768` recorded a Kerberos Ticket Granting Ticket (TGT) request for `auth.test`.

Relevant fields included:

```text
Event ID:                     4768
Account Name:                 auth.test
Realm:                        LIVINGENTERPRISE.LAB
Client Address:               ::ffff:192.168.133.131
Result Code:                  0x0
Ticket Encryption Type:       0x12
Session Encryption Type:      0x12
Pre-Authentication Type:      2
Pre-Authentication Encryption: 0x12
```

The client address:

```text
::ffff:192.168.133.131
```

is an IPv4-mapped IPv6 representation of:

```text
192.168.133.131
```

which again identified `LEE-WIN-CLT-01` as the originating system.

A result code of:

```text
0x0
```

indicated that the Kerberos request succeeded.

Ticket encryption type:

```text
0x12
```

represented:

```text
AES256-CTS-HMAC-SHA1-96
```

Pre-authentication type `2` indicated encrypted-timestamp Kerberos pre-authentication.

---

# 12. Kerberos Service-Ticket Evidence — Event ID 4769

Event ID `4769` was also identified for `auth.test`.

Relevant fields included:

```text
Event ID:               4769
Account Name:           auth.test@LIVINGENTERPRISE.LAB
Service Name:           krbtgt
Client Address:         ::ffff:192.168.133.131
Failure Code:           0x0
Ticket Encryption Type: 0x12
Session Encryption Type: 0x12
```

Most importantly, the event contained:

```text
Logon GUID:
{12eeb89d-87a2-7709-45d9-e13ecf140bd9}
```

The successful Event ID `4624` contained the exact same Logon GUID:

```text
{12eeb89d-87a2-7709-45d9-e13ecf140bd9}
```

This provided direct correlation between the successful Windows network logon and the observed Kerberos ticket activity.

Rather than relying solely on matching timestamps or usernames, the shared Logon GUID provided a stronger relationship between the two events.

---

# 13. CIFS Service-Ticket Check

Because the authentication test targeted an SMB resource, Event ID `4769` records were also searched for a recent CIFS service ticket.

The Security log was searched for:

```text
cifs
```

No CIFS-related 4769 associated with this authentication sequence was identified. The most recent CIFS event present in the log was from an earlier date and was unrelated to this test.

Therefore, the investigation did **not** conclude that a CIFS-specific 4769 was observed during this authentication sequence.

This distinction is important: expected behavior should not be documented as observed behavior unless supporting evidence is actually present.

---

# 14. Kerberos and NTLM Observation

An important difference was identified between the successful and failed authentication attempts.

### Successful Authentication

```text
4624
Logon Type 3
auth.test
192.168.133.131
Kerberos
```

Supporting Kerberos ticket activity was observed through Events `4768` and `4769`.

### Failed Authentication

```text
4625
Logon Type 3
auth.test@livingenterprise.lab
192.168.133.131
NtLmSsp
NTLM
0xC000006D
0xC000006A
```

The successful authentication was therefore observed using Kerberos, while the controlled incorrect-password attempts were recorded using NTLM/NtLmSsp.

The lab confirmed this difference in the observed events but did not conclusively determine the underlying Windows protocol-selection or negotiation mechanism responsible for it.

No unsupported conclusion regarding Kerberos-to-NTLM fallback was made.

---

# 15. Investigation Timeline

The authentication activity can be summarized as:

```text
LEE-WIN-CLT-01
192.168.133.131
Logged on as LEE\alicia.carter
        |
        |
        +---- auth.test + correct password
        |              |
        |              +--> Kerberos TGT activity
        |              |    Event 4768
        |              |
        |              +--> Kerberos ticket activity
        |              |    Event 4769
        |              |
        |              +--> Successful network logon
        |                   Event 4624
        |                   Logon Type 3
        |                   Kerberos
        |
        +---- SMB session removed
        |
        +---- auth.test + incorrect password
        |              |
        |              +--> Event 4625
        |                   Logon Type 3
        |                   NTLM
        |                   0xC000006A
        |
        +---- auth.test + incorrect password
                       |
                       +--> Event 4625
                            Logon Type 3
                            NTLM
                            0xC000006A
```

---

# 16. SOC Analyst Interpretation

If this activity appeared during an investigation rather than a controlled lab, an analyst could report:

> Multiple failed network authentication attempts were identified for `auth.test@livingenterprise.lab` from `LEE-WIN-CLT-01` (`192.168.133.131`) against `LEE-WIN-SRV-01`. The failures were recorded as Windows Security Event ID 4625, Logon Type 3, using NTLM authentication. Status `0xC000006D` indicated failed authentication and Sub Status `0xC000006A` identified an incorrect password as the cause. Repeated events originated from the same workstation and IP address while using different ephemeral source ports. A successful network authentication for the same account and source was separately observed as Event ID 4624 using Kerberos, with supporting Kerberos ticket activity in Events 4768 and 4769.

Additional investigation would then determine whether the failures represented:

* normal user error,
* stale credentials,
* a misconfigured service or scheduled task,
* password spraying,
* brute-force authentication,
* credential misuse, or
* other unauthorized activity.

Context, frequency, time window, affected accounts, source systems, and surrounding events would be required before assigning malicious intent.

---

# 17. Key Findings

This investigation demonstrated that a Windows authentication event should not be evaluated using Event ID alone.

Useful investigative fields included:

```text
Event ID
Account Name
Account Domain
Logon Type
Source Workstation
Source IP Address
Source Port
Authentication Package
Logon Process
Failure Reason
Status
Sub Status
Logon GUID
Kerberos Result Code
Ticket Encryption Type
```

The exercise also demonstrated the value of correlating multiple data points rather than relying on a single log entry.

A successful `4624`, failed `4625` events, Kerberos `4768/4769` activity, Active Directory account state, source addresses, and authentication metadata together provided a substantially clearer picture of what occurred.

---

# 18. Detection Engineering Relevance

The manual investigation performed in Event Viewer represents the same basic analytical process that can later be performed at scale using a SIEM.

Instead of manually searching individual Windows events, a detection platform can query and correlate authentication telemetry by:

```text
Authentication result
        ↓
Account
        ↓
Source workstation / IP
        ↓
Time window
        ↓
Logon type
        ↓
Failure status
        ↓
Authentication protocol
        ↓
Frequency / pattern
```

This provides the foundation for future detections such as:

* repeated failed logons from one host,
* one source attempting authentication against many accounts,
* one account receiving failures from many sources,
* successful authentication following repeated failures,
* unusual NTLM usage,
* privileged-account authentication anomalies, and
* abnormal Kerberos ticket activity.

These concepts will be revisited during later SIEM and detection-engineering phases of the Living Enterprise Environment.

---

# Conclusion

The controlled authentication exercise successfully produced and investigated both successful and failed Windows domain authentication activity.

The investigation established a known-good network authentication baseline, generated repeatable incorrect-password failures, identified the source system and account responsible for the activity, interpreted Windows authentication status codes, correlated Active Directory account state with Security events, and examined supporting Kerberos ticket activity.

Most importantly, the exercise demonstrated that authentication analysis requires correlation rather than simply identifying an Event ID.

The combination of identity, source, destination, authentication protocol, logon type, result codes, timestamps, and related events provides the context necessary to distinguish ordinary authentication failures from behavior that may require further security investigation.

# Troubleshooting and Lessons Learned

During the initial authentication baseline, the expected successful connection did not immediately occur. This created an additional troubleshooting opportunity before the controlled authentication investigation began.

## Initial Authentication Failure

The first authentication attempts used the NetBIOS-style domain credential format:

```cmd
net use \\LEE-WIN-SRV-01\IPC$ /user:LEE\auth.test *
```

Despite resetting and verifying the password, the authentication attempt returned:

```text
System error 1326 has occurred.

The user name or password is incorrect.
```

Rather than continuing to reset the password or modifying domain configuration, the investigation shifted toward validating each component involved in authentication.

---

## Active Directory Account Validation

The `auth.test` account was inspected directly from the domain controller.

The account was confirmed to:

* exist in Active Directory,
* be enabled,
* not be locked,
* have a current password,
* have a valid User Principal Name (UPN), and
* record the unsuccessful password attempts through `BadPwdCount` and `LastBadPasswordAttempt`.

This demonstrated that the failed authentication attempts were reaching the domain and affecting the expected Active Directory account state.

---

## Local Logon Test and Error 1385

A separate credential test was attempted on the domain controller using:

```cmd
runas /user:LEE\auth.test cmd
```

The result was:

```text
RUNAS ERROR: Unable to run - cmd

1385: Logon failure: the user has not been granted the requested logon type at this computer.
```

This was significantly different from error `1326`.

Error `1385` indicated that Windows denied the requested logon type rather than simply reporting invalid credentials.

Because `LEE-WIN-SRV-01` is a domain controller, the test account did not need loc

