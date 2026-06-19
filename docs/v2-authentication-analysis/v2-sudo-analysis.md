## Project Overview

In this exercise we are evaluating priviledge escalation and activities executed while in that state.  Privilege Escalation is a tactic within the MITRE ATT&CK 
framework that describes techniques used to obtain higher-level permissions on a system. Adversaries often use privilege escalation to gain access to restricted 
resources, execute administrative actions, establish persistence, or move laterally through an environment. I will perform various activities in the privilige
state within the linux system including admin duties.  I will then evaluate the logs parsing information pertaining to escalated privilege activities, document
my findings and the possible IOC's.

It is known that there are many "Single Pane of Glass" applications that are available that can be configured to collect, analyse, correlate and evaluate
if an alert needs to be sent or some automated duty needs to be activated/performed.  The purpose of this exercise is to perform the evaluation on the raw data
found in logs.  This will strengenthen foundational understanding of what has possibly happened when a specific alert is sent from a SIEM.  This leaves the mind
open to other tasks and activities that can take place to verify the fortitude of the systems being monitored.

This analysis was performed on:

`LEE-Ubuntu-01`

The server acts as the foundational Linux system for the Living Enterprise Environment (LEE) and will continue evolving throughout future labs involving:
- SIEM ingestion
- detection engineering
- segmentation
- authentication analysis
- threat simulation

---

# Commands Used


`sudo grep sudo /var/log/auth.log`  -  Documenting current sudo log entries.

`sudo ls /root`  -  Minimal risk privilege escalation.

`sudo cat /etc/shadow`  -  Accessing shadow file containing sensitive information including encrypted password information.

`sudo systemctl status ssh`  -  Administrative service inquiry.

`sudo apt update`  -  Administrative maintenance task.

`sudo useradd -m testuser`  -  Creating new user testuser.

`sudo usermod -aG sudo testuser`  -  Permission modification.

`grep sudo /var/log/auth.log`  -  Pulling logs for investigation purposes.

# Command Output

```bash
atibam@LEE-Ubuntu-01:/$ sudo ls /root
snap

tibam@LEE-Ubuntu-01:/$ sudo cat /etc/shadow
root:*:20566:0:99999:7:::
daemon:*:20566:0:99999:7:::
bin:*:20566:0:99999:7:::
sys:*:20566:0:99999:7:::
.............

tibam@LEE-Ubuntu-01:/$ sudo systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-06-19 12:23:03 EDT; 33min ago
 Invocation: 06e52f3774714b4785a256220f51cd4c
TriggeredBy: ● ssh.socket
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 1779 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 1792 (sshd)
      Tasks: 1 (limit: 3963)
     Memory: 1.6M (peak: 2.2M)
        CPU: 33ms
     CGroup: /system.slice/ssh.service
             └─1792 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Jun 19 12:23:03 LEE-Ubuntu-01 systemd[1]: Starting ssh.service - OpenBSD Secure Shell server...
Jun 19 12:23:03 LEE-Ubuntu-01 sshd[1792]: Server listening on 0.0.0.0 port 22.
Jun 19 12:23:03 LEE-Ubuntu-01 sshd[1792]: Server listening on :: port 22.
Jun 19 12:23:03 LEE-Ubuntu-01 systemd[1]: Started ssh.service - OpenBSD Secure Shell server.

atibam@LEE-Ubuntu-01:/$ sudo apt update
Ign:1 http://archive.ubuntu.com/ubuntu resolute InRelease
Ign:2 http://security.ubuntu.com/ubuntu resolute-security InRelease
Ign:3 http://archive.ubuntu.com/ubuntu resolute-updates InRelease
111 packages can be upgraded. Run 'apt list --upgradable' to see them.
Warning: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/resolute/InRelease  Temporary failure resolving 'archive.ubuntu.com'
Warning: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/resolute-updates/InRelease  Temporary failure resolving 'archive.ubuntu.com'
Warning: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/resolute-backports/InRelease  Temporary failure resolving 'archive.ubuntu.com'
Warning: Failed to fetch http://security.ubuntu.com/ubuntu/dists/resolute-security/InRelease  Temporary failure resolving 'security.ubuntu.com'
Warning: Some index files failed to download. They have been ignored, or old ones used instead.


tibam@LEE-Ubuntu-01:/$ sudo useradd -m testuser
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ sudo usermod -aG sudo testuser
atibam@LEE-Ubuntu-01:/$ 

```
---

## Logging

```bash
atibam@LEE-Ubuntu-01:/$ grep sudo /var/log/auth.log
2026-06-19T12:47:23.830515-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-19T12:47:23.830731-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/grep sudo /var/log/auth.log
2026-06-19T12:47:23.834334-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
2026-06-19T12:55:00.966772-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-19T12:55:00.969170-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/ls /root
2026-06-19T12:55:00.978480-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
2026-06-19T12:55:16.612104-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-19T12:55:16.612345-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/cat /etc/shadow
2026-06-19T12:55:16.622407-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
2026-06-19T12:55:46.992167-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-19T12:55:46.993088-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/systemctl statusssh
2026-06-19T12:55:47.002761-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
2026-06-19T12:56:43.409618-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-19T12:56:43.410157-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/systemctl status ssh
2026-06-19T12:56:43.437283-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
2026-06-19T12:57:39.751925-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-19T12:57:39.753394-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/apt update
2026-06-19T12:57:49.049760-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
2026-06-19T13:00:50.775005-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-19T13:00:50.777695-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/sbin/usermod -aG sudo testuser
2026-06-19T13:00:50.801223-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
2026-06-19T13:02:49.667524-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-19T13:02:49.667862-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/sbin/useradd -m testuser
2026-06-19T13:02:49.730876-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
2026-06-19T13:03:01.557165-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-19T13:03:01.557344-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/sbin/usermod -aG sudo testuser
2026-06-19T13:03:01.579305-04:00 LEE-Ubuntu-01 usermod[5288]: add 'testuser' to group 'sudo'
2026-06-19T13:03:01.579466-04:00 LEE-Ubuntu-01 usermod[5288]: add 'testuser' to shadow group 'sudo'
2026-06-19T13:03:01.588977-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root

```

---

## Findings

During the privilege escalation exercise, several artifacts were identified and collected from `auth.log`. These artifacts provide context regarding the source of the activity, the time of said activities, commands executed, the user and the outcome of each attempt.

| Artifact          | Observed Value      | Purpose                            |
| ----------------- | ------------------- | ---------------------------------- |
| User              | atibam              | Who executed the command           |
| TTY               | /dev/pts/0          | Interactive terminal used          |
| USER              | root                | Working directory at execution     |
| PWD               | /usr/bin/           | Working directory at execution     |
| Host              | LEE-Ubuntu-01       | Working directory at execution     |

---
| Command                          | Date/Time Executed          | User       | Date/Time Executed                     | Successful (Y/N) |       
| -------------------------------- | --------------------------- | ---------- | -------------------------------------- | ---------------- |
| sudo ls /root                    | 2026-06-19 12:55:00         | atibam     | list root directory entries            | Y                |
| sudo cat /etc/shadow             | 2026-06-19 12:55:16         | atibam     | display encrypted passwords            | Y                |
| sudo systemctl status ssh        | 2026-06-19 12:56:43         | atibam     | display current ssh service status     | Y                |
| sudo apt update                  | 2026-06-19 12:57:39         | atibam     | display latest packages                | Y                |
| sudo useradd -m testuser         | 2026-06-19 13:02:49         | atibam     | add new user                           | Y                |
| sudo usermod-aG sudo testuser    | 2026-06-19T13:03:01         | atibam     | add new user to sudo and shadow group  | Y                |


For each sudo event we ask ourselves the following question:

- Who was attemting escalation?
- When was the activity happening?
- What commands were executed?
- Where or what host/s what this activity taking place?
- What was authorized and was it expected activity or potential misuse?

---

## Potential Indicators of Compromise (IOCs)

No confirmed Indicators of Compromise were observed during this exercise. All privileged activity originated from the authorized administrative account (`atibam`) and was intentionally generated for lab validation purposes.

The following actions would warrant additional investigation if observed in a production environment without a corresponding change request, maintenance window, or approved administrative task:

| Activity                                                          | Security Concern                                             |
| ----------------------------------------------------------------- | ------------------------------------------------------------ |
| Accessing `/etc/shadow`                                           | Possible credential harvesting or password auditing activity |
| Creation of new user accounts                                     | Potential persistence mechanism                              |
| Adding users to the `sudo` group                                  | Privilege escalation or unauthorized administrative access   |
| Repeated execution of privileged commands                         | Potential misuse of administrative privileges                |
| Unexpected `sudo apt` activity                                    | Unauthorized software installation or system modification    |
| Privileged commands executed outside approved maintenance windows | Potential insider threat or compromised account activity     |

All observed activity during this exercise was validated as expected administrative behavior and successfully correlated to authorized testing performed within the Living Enterprise Environment (LEE).
---
## Key Observations

- All privileged activity originated from the authorized account `atibam`.
- Every sudo session successfully opened and closed.
- No failed sudo attempts were observed.
- No unauthorized privilege escalation activity was detected.
- User creation and privilege modification events were successfully logged.
- Commands executed through sudo were fully recorded within auth.log.
---

- ## Assessment

The ability to distinguish legitimate administrative activity from potentially malicious behavior is a fundamental responsibility of security operations personnel.
Reviewing privileged activity in context, correlating events with approved change requests, maintenance windows, or service tickets helps reduce false positives and allows analysts to focus on genuine threats.

