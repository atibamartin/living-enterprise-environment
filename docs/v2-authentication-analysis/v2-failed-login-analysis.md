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
## Logging

```bash
tibam@LEE-Ubuntu-01:/$ sudo grep "sshd" /var/log/auth.log
[sudo: authenticate] Password:                
2026-06-18T13:01:04.234184-04:00 LEE-Ubuntu-01 sshd[5532]: Server listening on 0.0.0.0 port 22.
2026-06-18T13:01:04.234300-04:00 LEE-Ubuntu-01 sshd[5532]: Server listening on :: port 22.
2026-06-18T13:05:50.654875-04:00 LEE-Ubuntu-01 sshd-session[6039]: Invalid user fakeuser from 192.168.133.128 port 51806
2026-06-18T13:05:54.719616-04:00 LEE-Ubuntu-01 sshd-session[6039]: pam_unix(sshd:auth): check pass; user unknown
2026-06-18T13:05:54.719801-04:00 LEE-Ubuntu-01 sshd-session[6039]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.133.128 
2026-06-18T13:05:57.006219-04:00 LEE-Ubuntu-01 sshd-session[6039]: Failed password for invalid user fakeuser from 192.168.133.128 port 51806 ssh2
2026-06-18T13:06:04.964874-04:00 LEE-Ubuntu-01 sshd-session[6039]: pam_unix(sshd:auth): check pass; user unknown
2026-06-18T13:06:06.835433-04:00 LEE-Ubuntu-01 sshd-session[6039]: Failed password for invalid user fakeuser from 192.168.133.128 port 51806 ssh2
2026-06-18T13:06:15.043623-04:00 LEE-Ubuntu-01 sshd-session[6039]: pam_unix(sshd:auth): check pass; user unknown
2026-06-18T13:06:17.001779-04:00 LEE-Ubuntu-01 sshd-session[6039]: Failed password for invalid user fakeuser from 192.168.133.128 port 51806 ssh2
2026-06-18T13:06:18.356871-04:00 LEE-Ubuntu-01 sshd-session[6039]: Connection closed by invalid user fakeuser 192.168.133.128 port 51806 [preauth]
2026-06-18T13:06:18.358830-04:00 LEE-Ubuntu-01 sshd-session[6039]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.133.128 
2026-06-18T13:08:34.601419-04:00 LEE-Ubuntu-01 sshd-session[6056]: Invalid user fakeuser from 192.168.133.128 port 58292
2026-06-18T13:08:42.538167-04:00 LEE-Ubuntu-01 sshd-session[6056]: pam_unix(sshd:auth): check pass; user unknown
2026-06-18T13:08:42.538288-04:00 LEE-Ubuntu-01 sshd-session[6056]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.133.128 
2026-06-18T13:08:44.550632-04:00 LEE-Ubuntu-01 sshd-session[6056]: Failed password for invalid user fakeuser from 192.168.133.128 port 58292 ssh2
2026-06-18T13:08:51.848901-04:00 LEE-Ubuntu-01 sshd-session[6056]: pam_unix(sshd:auth): check pass; user unknown
2026-06-18T13:08:53.568231-04:00 LEE-Ubuntu-01 sshd-session[6056]: Failed password for invalid user fakeuser from 192.168.133.128 port 58292 ssh2
2026-06-18T13:08:57.030794-04:00 LEE-Ubuntu-01 sshd-session[6056]: pam_unix(sshd:auth): check pass; user unknown
2026-06-18T13:08:59.643414-04:00 LEE-Ubuntu-01 sshd-session[6056]: Failed password for invalid user fakeuser from 192.168.133.128 port 58292 ssh2
2026-06-18T13:09:00.344767-04:00 LEE-Ubuntu-01 sshd-session[6056]: Connection closed by invalid user fakeuser 192.168.133.128 port 58292 [preauth]
2026-06-18T13:09:00.345456-04:00 LEE-Ubuntu-01 sshd-session[6056]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.133.128 
2026-06-18T13:09:22.516153-04:00 LEE-Ubuntu-01 sshd-session[6079]: Accepted password for atibam from 192.168.133.128 port 41650 ssh2
2026-06-18T13:09:22.518491-04:00 LEE-Ubuntu-01 sshd-session[6079]: pam_unix(sshd:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T13:13:48.841924-04:00 LEE-Ubuntu-01 sshd-session[6079]: syslogin_perform_logout: logout() returned an error
2026-06-18T13:13:48.844257-04:00 LEE-Ubuntu-01 sshd-session[6146]: Received disconnect from 192.168.133.128 port 41650:11: disconnected by user
2026-06-18T13:13:48.845415-04:00 LEE-Ubuntu-01 sshd-session[6146]: Disconnected from user atibam 192.168.133.128 port 41650
2026-06-18T13:13:48.845507-04:00 LEE-Ubuntu-01 sshd-session[6079]: pam_unix(sshd:session): session closed for user atibam
2026-06-18T13:46:18.072759-04:00 LEE-Ubuntu-01 sshd[1745]: Server listening on 0.0.0.0 port 22.
2026-06-18T13:46:18.072856-04:00 LEE-Ubuntu-01 sshd[1745]: Server listening on :: port 22.
2026-06-18T13:47:27.444945-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/grep sshd /var/log/auth.log
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ sudo grep "Failed password" /var/log/auth.log
2026-06-18T13:05:57.006219-04:00 LEE-Ubuntu-01 sshd-session[6039]: Failed password for invalid user fakeuser from 192.168.133.128 port 51806 ssh2
2026-06-18T13:06:06.835433-04:00 LEE-Ubuntu-01 sshd-session[6039]: Failed password for invalid user fakeuser from 192.168.133.128 port 51806 ssh2
2026-06-18T13:06:17.001779-04:00 LEE-Ubuntu-01 sshd-session[6039]: Failed password for invalid user fakeuser from 192.168.133.128 port 51806 ssh2
2026-06-18T13:08:44.550632-04:00 LEE-Ubuntu-01 sshd-session[6056]: Failed password for invalid user fakeuser from 192.168.133.128 port 58292 ssh2
2026-06-18T13:08:53.568231-04:00 LEE-Ubuntu-01 sshd-session[6056]: Failed password for invalid user fakeuser from 192.168.133.128 port 58292 ssh2
2026-06-18T13:08:59.643414-04:00 LEE-Ubuntu-01 sshd-session[6056]: Failed password for invalid user fakeuser from 192.168.133.128 port 58292 ssh2
2026-06-18T13:48:29.311952-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/grep Failed password /var/log/auth.log
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ sudo grep "Invalid user" /var/log/auth.log
2026-06-18T13:05:50.654875-04:00 LEE-Ubuntu-01 sshd-session[6039]: Invalid user fakeuser from 192.168.133.128 port 51806
2026-06-18T13:08:34.601419-04:00 LEE-Ubuntu-01 sshd-session[6056]: Invalid user fakeuser from 192.168.133.128 port 58292
2026-06-18T13:49:02.012296-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/grep Invalid user /var/log/auth.log
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ sudo grep "Accepted" /var/log/auth.log
2026-06-18T11:06:47.235927-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/grep Accepted /var/log/auth.log
2026-06-18T13:09:22.516153-04:00 LEE-Ubuntu-01 sshd-session[6079]: Accepted password for atibam from 192.168.133.128 port 41650 ssh2
2026-06-18T13:49:32.727293-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/grep Accepted /var/log/auth.log
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ sudo grep "session opened" /var/log/auth.log
2026-06-18T11:01:29.044178-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm-greeter(uid=60578) by (uid=0)
2026-06-18T11:01:29.307518-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user gdm-greeter(uid=60578) by gdm-greeter(uid=0)
2026-06-18T11:05:05.942099-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm-greeter(uid=60578) by (uid=0)
2026-06-18T11:05:06.138849-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user gdm-greeter(uid=60578) by gdm-greeter(uid=0)
2026-06-18T11:05:52.671406-04:00 LEE-Ubuntu-01 gdm-password]: pam_unix(gdm-password:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T11:05:52.739071-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T11:06:03.391274-04:00 LEE-Ubuntu-01 pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T11:06:47.235211-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T11:17:01.540439-04:00 LEE-Ubuntu-01 CRON[4780]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2026-06-18T11:30:01.565502-04:00 LEE-Ubuntu-01 CRON[4832]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2026-06-18T12:17:01.630890-04:00 LEE-Ubuntu-01 CRON[5267]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2026-06-18T12:30:01.658212-04:00 LEE-Ubuntu-01 CRON[5293]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2026-06-18T12:54:46.040247-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm-greeter(uid=60578) by (uid=0)
2026-06-18T12:54:46.230539-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user gdm-greeter(uid=60578) by gdm-greeter(uid=0)
2026-06-18T12:58:12.786342-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm-greeter(uid=60578) by (uid=0)
2026-06-18T12:58:13.048598-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user gdm-greeter(uid=60578) by gdm-greeter(uid=0)
2026-06-18T12:58:25.359175-04:00 LEE-Ubuntu-01 gdm-password]: pam_unix(gdm-password:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T12:58:25.456903-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T12:58:35.675082-04:00 LEE-Ubuntu-01 pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:00:16.365672-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:00:24.499277-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:01:04.164285-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:01:11.497858-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:01:32.782871-04:00 LEE-Ubuntu-01 pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:09:22.518491-04:00 LEE-Ubuntu-01 sshd-session[6079]: pam_unix(sshd:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T13:11:17.389004-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:17:01.187499-04:00 LEE-Ubuntu-01 CRON[6284]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2026-06-18T13:30:01.212743-04:00 LEE-Ubuntu-01 CRON[6311]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2026-06-18T13:46:18.603121-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm-greeter(uid=60578) by (uid=0)
2026-06-18T13:46:18.767162-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user gdm-greeter(uid=60578) by gdm-greeter(uid=0)
2026-06-18T13:46:38.556922-04:00 LEE-Ubuntu-01 gdm-password]: pam_unix(gdm-password:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T13:46:38.640579-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T13:46:48.331197-04:00 LEE-Ubuntu-01 pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:47:27.444634-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:48:29.311767-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:49:02.011087-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:49:32.726438-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:49:47.732305-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T13:49:47.733098-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/grep session opened /var/log/auth.log
atibam@LEE-Ubuntu-01:/$ 
```

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



