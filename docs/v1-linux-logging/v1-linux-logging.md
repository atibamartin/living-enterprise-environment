# V1 - Linux Logging + Establishing Baseline

## Objective
Learn Linux logging and grep analysis, while creating
a baseline of these three essential logs:

- auth.log
- syslog
- kern.log

We will be performing this evaluation on LEE-Ubuntu-01 server.  It is my initial
server that will be used for early labs.  It will also be used as the Living Enterprise
evolves.

---


## auth.log Evaluation

## Commands Used

```bash
grep "sudo"/var/auth.log
grep "failed" /var/log/auth.log
cat /var/log/auth.log
```

### Command

```bash
grep "sudo" /var/log/auth.log
```

---

## Log Output

```markdown id="jlwm3h"
grep "sudo" /var/log/auth.log

2026-05-19T00:22:17.487386-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)

2026-05-19T00:22:17.490304-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/home/atibam ; USER=root ; COMMAND=/usr/bin/cat /var/log/auth.log

2026-05-19T00:22:17.494169-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root

```
### Command

```bash
cat /var/log/auth.log
```

---

## Log Output

```markdown id="jlwm3h"
grep "sudo" /var/log/auth.log

2026-05-19T00:22:17.487386-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)

2026-05-19T00:22:17.490304-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/home/atibam ; USER=root ; COMMAND=/usr/bin/cat /var/log/auth.log

2026-05-19T00:22:17.494169-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
```
```markdown id="jlwm3h"
atibam@LEE-Ubuntu-01:~$ grep "Failed" /var/log/auth.log
2026-05-19T00:13:13.626576-04:00 LEE-Ubuntu-01 dbus-daemon[1391]: [system] Failed to activate service 'org.bluez': timed out (service_start_timeout=25000ms)
## Ubuntu attempted to activate Bluetooth-related service and there was none
```
```markdown id="jlwm3h"
cat /var/log/auth.log
2026-05-19T00:03:50.171024-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm-greeter(uid=60578) by (uid=0)
## gdm-greeter - equivalent to windows login screen - graphical authentication interface

2026-05-19T00:03:50.214936-04:00 LEE-Ubuntu-01 systemd-logind[1541]: New session 'c1' of user 'gdm-greeter' with class 'greeter' and type 'wayland'.
## Ubuntu launched a graphical desktop session.
## user successful login

2026-05-19T00:04:29.292432-04:00 LEE-Ubuntu-01 pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by atibam(uid=1000)
## Ubuntu equivalent of run as administrator

2026-05-19T00:17:01.078797-04:00 LEE-Ubuntu-01 CRON[5539]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
## Ubuntu equivalent of windows task scheduler
## Attackers often use cron persistence.  Its a sign that the malware is attempting to maintain
## it's foothold on a system by using scheduled tasks to ensure it runs regularly.
```
---

## Analysis / Significance

### Analysis

This log sequence shows a successful sudo privilege escalation event.

User `atibam` temporarily elevated privileges to root in order to read `/var/log/auth.log`.

## Important observations:
- sudo activity is fully logged
- executed commands are visible
- session open and close events are recorded
- this behavior forms part of the system authentication baseline

## Security significance - Authentication logs can reveal:
- privilege escalation
- administrative activity
- suspicious command execution
- attacker persistence behavior

---

## Baseline Observations

- Successful graphical login events observed for user atibam
- sudo usage correctly logged with command execution details
- session open/close events visible through PAM
- cron automated tasks generate authentication session logs
- pkexec entries indicate GUI privilege escalation activity
- Bluetooth service startup failure observed but determined non-security related


