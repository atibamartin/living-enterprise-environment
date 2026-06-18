## Project Overview

Now that we have established authentication baselines and developed an understanding of normal authentication log behavior, we will generate controlled failed SSH login attempts from our Kali Linux system against LEE-Ubuntu-01.

The purpose of this exercise is to observe how failed authentication events are recorded within `auth.log`, compare these entries against our established baseline, and identify potential Indicators of Compromise (IOCs) that a security analyst may investigate during an authentication-related security event.

By generating known events in a controlled lab environment, we can better understand how unauthorized access attempts appear in system logs and develop the investigative skills required to distinguish normal activity from suspicious behavior.

---

## Troubleshooting Notes

Initial SSH authentication testing was unsuccessful because the OpenSSH Server service was not installed and running on LEE-Ubuntu-01.

To resolve the issue, the following steps were performed:

```bash
sudo apt update && sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh
```

Network connectivity was validated by confirming that both systems resided on the same subnet:

```text
LEE-Kali-01      192.168.133.128/24
LEE-Ubuntu-01    192.168.133.129/24
```

After installing and enabling the SSH service, both failed and successful authentication events were successfully generated and recorded for analysis.

This troubleshooting process provided additional understanding of SSH service dependencies and authentication logging behavior within Ubuntu Linux.

---

### Post-Authentication Validation

After successfully authenticating to LEE-Ubuntu-01 via SSH, several commands were executed to validate interactive access and confirm successful session establishment.

#### List Directory Contents

```bash
ls -al
```

The command successfully returned the contents of the user's home directory, confirming that the SSH session was fully established and that the authenticated user had access to their assigned resources.

#### Verify Firewall Status

```bash
sudo ufw status
```

Output:

```text
Status: inactive
```

This confirmed that the Ubuntu Uncomplicated Firewall (UFW) was not actively filtering network traffic during testing. The inactive firewall state helped validate that SSH connectivity issues encountered earlier were related to the OpenSSH service configuration rather than host-based firewall restrictions.

### Analyst Observation

Successful SSH authentication was followed by normal command execution and system interaction. This demonstrates the distinction between a successful authentication event and a successful interactive login session, both of which may be relevant during a security investigation.

---

## Artifacts Collected

During the authentication analysis exercise, several artifacts were identified and collected from `auth.log`. These artifacts provide context regarding the source of the activity, the targeted account, the authentication method used, and the outcome of each authentication attempt.

| Artifact                           | Value               |
| ---------------------------------- | ------------------- |
| Source Host                        | LEE-Kali-01         |
| Source IP Address                  | 192.168.133.128     |
| Target Host                        | LEE-Ubuntu-01       |
| Target IP Address                  | 192.168.133.129     |
| Authentication Protocol            | SSHv2               |
| Invalid Username                   | fakeuser            |
| Valid Username                     | atibam              |
| Failed Authentication Attempts     | 6                   |
| Successful Authentication Attempts | 1                   |
| SSH Service Port                   | 22                  |
| Source Connection Ports            | 51806, 58292, 41650 |
| Authentication Log                 | /var/log/auth.log   |

---

## Authentication Timeline

The following timeline summarizes the authentication events generated during testing.

| Timestamp | Event                                                         |
| --------- | ------------------------------------------------------------- |
| 13:05:50  | Invalid user `fakeuser` identified from 192.168.133.128       |
| 13:05:57  | Failed password attempt for invalid user `fakeuser`           |
| 13:06:06  | Failed password attempt for invalid user `fakeuser`           |
| 13:06:17  | Failed password attempt for invalid user `fakeuser`           |
| 13:06:18  | Connection closed for invalid user `fakeuser`                 |
| 13:08:34  | Second SSH connection initiated using invalid user `fakeuser` |
| 13:08:44  | Failed password attempt                                       |
| 13:08:53  | Failed password attempt                                       |
| 13:08:59  | Failed password attempt                                       |
| 13:09:00  | Connection closed for invalid user `fakeuser`                 |
| 13:09:22  | Successful SSH authentication for user `atibam`               |
| 13:09:22  | SSH session opened for user `atibam`                          |
| 13:13:48  | User disconnected from SSH session                            |
| 13:13:48  | SSH session closed for user `atibam`                          |

---

## Potential IOCs Observed

The following Indicators of Compromise (IOCs) were identified during analysis. While these events were intentionally generated in a controlled lab environment, similar activity in a production environment may warrant investigation.

### IOC #1 - Invalid User Enumeration

```text
Invalid user fakeuser from 192.168.133.128
```

An authentication attempt was made using a username that does not exist on the target system.

Potential analyst concern:

* User enumeration activity
* Automated scanning
* Credential stuffing attempts
* Reconnaissance against valid accounts

---

### IOC #2 - Repeated Failed Authentication Attempts

```text
Failed password for invalid user fakeuser
```

Multiple failed password attempts were recorded against the same username.

Potential analyst concern:

* Password guessing
* Brute force activity
* Automated authentication attacks

Observed Count:

* 6 failed authentication attempts

---

### IOC #3 - Multiple SSH Connection Attempts

The source host established multiple SSH sessions using different ephemeral source ports.

Observed Source Ports:

```text
51806
58292
```

Potential analyst concern:

* Repeated connection attempts
* Automated attack tooling
* Persistent unauthorized access attempts

---

### IOC #4 - Remote Authentication Activity

Authentication attempts originated from a remote host rather than the local system.

Observed Source:

```text
192.168.133.128
```

Potential analyst concern:

* External authentication attempts
* Lateral movement activity
* Unauthorized access attempts

---

## Successful Authentication Event

Following the failed authentication testing, a successful SSH login was performed using a valid account.

```text
Accepted password for atibam from 192.168.133.128 port 41650 ssh2
```

The logs confirmed:

* Successful authentication
* Session creation
* Interactive shell access
* Session termination

This event provides a useful comparison against the failed authentication attempts and establishes a baseline for legitimate SSH access.

---

## Analyst Findings

Analysis of the authentication logs demonstrated clear differences between failed and successful SSH authentication events.

Failed authentication attempts generated indicators such as invalid user detection, repeated password failures, and connection termination events. These activities could represent user enumeration or brute force behavior in a production environment.

The successful authentication event generated distinct log entries showing password acceptance, session creation, user activity, and session closure. These entries provide a baseline for normal remote administrative access.

No malicious activity was observed during testing, as all events were intentionally generated within a controlled lab environment. However, the exercise successfully demonstrated how authentication-related security events are recorded, investigated, and analyzed using Linux authentication logs.



