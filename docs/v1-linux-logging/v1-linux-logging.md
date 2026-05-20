# V1 - Linux Logging + grep exercise

## Objective
Learn Linux logging and grep analysis, while creating
a baseline of these three essential logs:

- auth.log
- syslog
- kern.log

## auth.log Evaluation

## Commands Used

```bash
cat /var/log/auth.log
grep "sudo"/var/auth.log
grep "failed" /var/log/auth.log
```

### Command

```bash
grep "sudo" /var/log/auth.log
```

---

## 3. Log Output

```markdown id="jlwm3h"
### Log Output

```text
2026-05-19T00:22:17.487386-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)

2026-05-19T00:22:17.490304-04:00 LEE-Ubuntu-01 sudo: atibam : TTY=/dev/pts/0 ; PWD=/home/atibam ; USER=root ; COMMAND=/usr/bin/cat /var/log/auth.log

2026-05-19T00:22:17.494169-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root

```

---

## 4. Analysis / Significance

```markdown id="jlwm4h"
```
### Analysis

This log sequence shows a successful sudo privilege escalation event.

User `atibam` temporarily elevated privileges to root in order to read `/var/log/auth.log`.

Important observations:
- sudo activity is fully logged
- executed commands are visible
- session open and close events are recorded
- this behavior forms part of the system authentication baseline

Security significance:
Authentication logs can reveal:
- privilege escalation
- administrative activity
- suspicious command execution
- attacker persistence behavior
