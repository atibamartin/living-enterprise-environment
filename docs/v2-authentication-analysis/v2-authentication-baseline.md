# V2 - Authentication Analysis

## Project Overview

This phase focuses on understanding how authentication
events are generated, logged, analyzed, and detected
within a Linux environment.

The objective is to establish a baseline for normal
authentication behavior and identify indicators
associated with failed logins, SSH activity,
privilege escalation, and brute-force attacks.

---

## Learning Objectives

- Understand successful authentication events
- Analyze failed login attempts
- Monitor SSH activity
- Track sudo and privilege escalation actions
- Identify brute-force behavior
- Develop basic detection logic

---

## Environment

| System | Role |
|----------|----------|
| Ubuntu Server | Log Source |
| Kali Linux | Attack Simulation |
| auth.log | Evidence Source |

---

## Success Criteria

- Authentication baseline documented
- Failed login events analyzed
- SSH activity reviewed
- Sudo events documented
- Brute-force activity simulated
- Detection logic created

---

# Capturing Authentication Baseline
Our goal here is to establish the baseline of authentication events including login behavior, timestamps, usernames and source IPs
to have a defined level of authentication behavior that we can compare to any future events.  This will allow us to identify
deviations from the norm, which will serve as indicators of compromise.

### Commands used

```bash
grep "Accepted" /var/log/auth.log
grep "session opened" /var/log/auth.log
grep "session closed" /var/log/auth.log
```
---
### ip address
`192.168.133.129/24`
---

## Log Output

```text
atibam@LEE-Ubuntu-01:/$ sudo grep "Accepted" /var/log/auth.log
[sudo: authenticate] Password:

atibam@LEE-Ubuntu-01:/$ grep "session opened" /var/log/auth.log
2026-06-18T11:01:29.044178-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm-greeter(uid=60578) by (uid=0)
2026-06-18T11:01:29.307518-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user gdm-greeter(uid=60578) by gdm-greeter(uid=0)
2026-06-18T11:05:05.942099-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session opened for user gdm-greeter(uid=60578) by (uid=0)
2026-06-18T11:05:06.138849-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user gdm-greeter(uid=60578) by gdm-greeter(uid=0)
2026-06-18T11:05:52.671406-04:00 LEE-Ubuntu-01 gdm-password]: pam_unix(gdm-password:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T11:05:52.739071-04:00 LEE-Ubuntu-01 (systemd): pam_unix(systemd-user:session): session opened for user atibam(uid=1000) by atibam(uid=0)
2026-06-18T11:06:03.391274-04:00 LEE-Ubuntu-01 pkexec: pam_unix(polkit-1:session): session opened for user root(uid=0) by atibam(uid=1000)
2026-06-18T11:06:47.235211-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by atibam(uid=1000)
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ 
atibam@LEE-Ubuntu-01:/$ grep "session closed" /var/log/auth.log
2026-06-18T11:01:02.565421-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm-greeter
2026-06-18T11:03:00.973616-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm-greeter
2026-06-18T11:06:01.976768-04:00 LEE-Ubuntu-01 gdm-launch-environment]: pam_unix(gdm-launch-environment:session): session closed for user gdm-greeter
2026-06-18T11:06:47.247723-04:00 LEE-Ubuntu-01 sudo: pam_unix(sudo:session): session closed for user root
2026-06-18T11:17:01.545214-04:00 LEE-Ubuntu-01 CRON[4780]: pam_unix(cron:session): session closed for user root

```

## Authentication Baseline Findings

### Command: Search for Accepted Authentication Events

```bash
sudo grep "Accepted" /var/log/auth.log
```

### Result

No output was returned.

### Analysis

No `Accepted` authentication events were present in the current `auth.log` sample. This indicates that there were no recorded successful SSH-style login events during this baseline window.

This is expected because the system was accessed locally through the graphical login process rather than through SSH.

---

## Session Open Events

### Command

```bash
grep "session opened" /var/log/auth.log
```

### Key Observations

The logs showed session activity for the following users:

* `gdm-greeter`
* `atibam`
* `root`

### gdm-greeter

The `gdm-greeter` session represents the GNOME Display Manager login screen. This activity occurs before the main user logs into the system.

Example:

```text
session opened for user gdm-greeter(uid=60578)
```

This is normal baseline behavior for an Ubuntu system using a graphical login screen.

### atibam

The `atibam` user session represents the primary local user login.

Example:

```text
gdm-password:session): session opened for user atibam(uid=1000)
systemd-user:session): session opened for user atibam(uid=1000)
```

This confirms that the local user successfully logged into the system.

### root

The `root` session entries were created when the `atibam` user performed privileged actions using tools such as `pkexec` and `sudo`.

Example:

```text
session opened for user root(uid=0) by atibam(uid=1000)
```

This is expected behavior when a standard user performs administrative actions.

---

## Session Close Events

### Command

```bash
grep "session closed" /var/log/auth.log
```

### Key Observations

The logs showed closed sessions for:

* `gdm-greeter`
* `root`
* `cron`

The `root` session close event corresponds to a completed privileged action.

Example:

```text
sudo: pam_unix(sudo:session): session closed for user root
```

No closed session for `atibam` was observed in this sample because the user session was likely still active at the time the command was run.

---

## Baseline Summary

During this authentication baseline review, the system showed normal local login behavior. No successful SSH authentication events were observed. The logs showed expected graphical login activity from `gdm-greeter`, a successful local session for user `atibam`, and temporary privileged sessions for `root` created through `sudo` and `pkexec`.

No suspicious authentication activity was identified in this sample.
