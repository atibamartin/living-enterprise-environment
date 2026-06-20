# V1 - Linux Logging + Establishing Baseline for Authentication Logs

## Objective
Learn Linux system logging and grep analysis while establishing a baseline for normal operational telemetry generated within `/var/log/auth.log`.


This analysis was performed on:

`LEE-Ubuntu-01`

The server acts as the foundational Linux system for the Living Enterprise Environment (LEE) and will continue evolving throughout future labs involving:
- SIEM ingestion
- detection engineering
- segmentation
- authentication analysis
- threat simulation


---


## auth.log Evaluation

### Command #1

```bash
grep "sudo" /var/log/auth.log
```

---

## Log Output

```markdown id="jlwm3h"
grep "sudo" /var/log/auth.log

2026-05-19T00:22:17.487386-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)

### Observation

A privileged sudo session was opened for the root account by user `atibam`.

### Significance

This represents a successful privilege escalation event using sudo.

2026-05-19T00:22:17.490304-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/home/atibam ; USER=root ; COMMAND=/usr/bin/cat /var/log/auth.log

## Command that was run with sudo privilege

2026-05-19T00:22:17.494169-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root

## Elevated/root session ended normally.

```
### Command #2

```bash
cat /var/log/auth.log
```
## Log Output 

```markdown id="jlwm3h"
cat /var/log/auth.log
2026-05-19T00:03:50.171024-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm-greeter(uid=60578) by (uid=0)

## gdm-greeter - equivalent to windows login screen - graphical authentication interface

2026-05-19T00:03:50.214936-04:00 LEE-Ubuntu-01 systemd-logind[1541]: New session 'c1' of user 'gdm-greeter' with class 'greeter' and type 'wayland'.

## Ubuntu launched a graphical desktop session.

2026-05-19T00:04:29.292432-04:00 LEE-Ubuntu-01 pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by atibam(uid=1000)

##`pkexec` provides privilege escalation functionality similar to "Run as Administrator" in Windows environments.

2026-05-19T00:17:01.078797-04:00 LEE-Ubuntu-01 CRON[5539]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)

## Ubuntu equivalent of windows task scheduler
## Attackers often use cron persistence.  Its a sign that the malware is attempting to maintain
## it's foothold on a system by using scheduled tasks to ensure it runs regularly.
```
---
### Command #3
```bash
 grep "Failed" /var/log/auth.log
```

## Log Output 

```markdown id="jlwm3h"
atibam@LEE-Ubuntu-01:~$ grep "Failed" /var/log/auth.log
2026-05-19T00:13:13.626576-04:00 LEE-Ubuntu-01 dbus-daemon[1391]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)

## Ubuntu attempted to activate Bluetooth-related service and there were none present.
```

---

## Security significance - Authentication logs can reveal:
- privilege escalation
- administrative activity
- suspicious command execution
- attacker persistence behavior

---

## Key Takeaways

- Linux authentication activity is heavily logged through PAM and auth.log
- sudo activity records both privilege escalation and executed commands
- pkexec represents GUI-based privilege escalation
- cron jobs generate authentication session telemetry
- not all "Failed" log entries indicate malicious activity
- establishing a baseline is critical for anomaly detection


