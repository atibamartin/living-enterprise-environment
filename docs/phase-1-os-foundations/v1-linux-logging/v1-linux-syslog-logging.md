# V1 - Linux Logging + Establishing Baseline for System Logs (syslog)

## Objective

Learn Linux system logging and grep analysis while establishing a baseline for normal operational telemetry generated within `/var/log/syslog`.

This evaluation focuses on:
- service lifecycle activity
- network initialization
- virtualization-related telemetry
- AppArmor enforcement events
- socket activity
- NetworkManager behavior
- systemd operational telemetry

This analysis was performed on:

`LEE-Ubuntu-01`

The server acts as the foundational Linux system for the Living Enterprise Environment (LEE) and will continue evolving throughout future labs involving:
- SIEM ingestion
- detection engineering
- segmentation
- authentication analysis
- threat simulation

---

# Command #1

```bash
tail -50 /var/log/syslog
```

## Log Output

```text
2026-05-20T18:33:47.772570-04:00 LEE-Ubuntu-01 ssh-agent[1945]: exiting on signal 15

2026-05-20T18:33:47.773942-04:00 LEE-Ubuntu-01 systemd[1775]: Stopped gcr-ssh-agent.service - GCR ssh-agent wrapper.

2026-05-20T18:33:47.774861-04:00 LEE-Ubuntu-01 systemd[1775]: Stopped ssh-agent.service - OpenSSH Agent.

2026-05-20T18:33:47.825982-04:00 LEE-Ubuntu-01 systemd[1775]: app.slice: Consumed 2.314s CPU time over 34.003s wall clock time, 86.2M memory peak.

2026-05-20T18:33:47.826041-04:00 LEE-Ubuntu-01 systemd[1775]: Reached target shutdown.target - Shutdown.

2026-05-20T18:34:21.337772-04:00 LEE-Ubuntu-01 systemd[1]: geoclue.service: Deactivated successfully.
```

## Observations

- `ssh-agent` terminated gracefully using signal 15 (SIGTERM)
- systemd successfully stopped multiple services and sockets during session teardown
- CPU and memory consumption telemetry was recorded for the session
- services and sockets closed in an orderly sequence
- `geoclue.service` deactivated successfully after inactivity

## Security Significance

These entries represent normal Linux session and service shutdown behavior.

Important operational concepts observed:
- service lifecycle telemetry
- process termination events
- system resource consumption metrics
- socket management
- orderly session teardown

Understanding normal shutdown behavior is critical because:
- attackers may terminate services unexpectedly
- malware may crash or force-stop processes
- unusual shutdown sequences may indicate instability or malicious activity

---

# Command #2

```bash
grep "error" /var/log/syslog
```

## Log Output

```text
2026-05-17T20:21:56.858163+00:00 LEE-Ubuntu-01 alsactl[1691]: alsa-lib main.c:1804:(snd_use_case_mgr_open) [error.ucm] failed to import hw:0 use case configuration -2

2026-05-17T20:22:00.833189+00:00 LEE-Ubuntu-01 kernel: audit: type=1400 audit(1779049320.831:242): apparmor="DENIED" operation="mount" class="mount" info="failed flags match" error=-13 profile="snap-update-ns.desktop-security-center"

2026-05-17T20:26:55.623340+00:00 LEE-Ubuntu-01 kernel: audit: type=1400 audit(1779049615.621:259): apparmor="DENIED" operation="getattr" class="file" info="Failed name lookup - disconnected path" error=-13

2026-05-20T18:33:36.639891-04:00 LEE-Ubuntu-01 gnome-shell[3434]: libinput error: event2 - VirtualPS/2 VMware VMMouse: client bug: event processing lagging behind by 36ms, your system is too slow
```

## Observations

- ALSA audio subsystem generated minor virtualization-related configuration warnings
- AppArmor denied restricted operations and successfully enforced security policy
- VMware virtual mouse generated minor performance lag telemetry
- AppArmor events were recorded through the Linux kernel audit subsystem

## Security Significance

Important lesson:
Not all log entries containing the words:
- error
- failed
- denied

represent malicious activity or compromise.

Key findings:
- AppArmor "DENIED" events often indicate security controls functioning correctly
- virtualization environments commonly generate hardware abstraction noise
- Linux desktop environments generate significant non-security operational telemetry

Operational relevance:
Security analysts must distinguish between:
- normal operational noise
- expected security-control enforcement
- true anomalies requiring investigation

---

# Command #3

```bash
grep "network" /var/log/syslog
```

## Log Output

```text
2026-05-17T20:21:56.739893+00:00 LEE-Ubuntu-01 systemd[1]: Reached target network-pre.target - Preparation for Network.

2026-05-17T20:21:56.751733+00:00 LEE-Ubuntu-01 systemd[1]: Starting networkd-dispatcher.service - Dispatcher daemon for systemd-networkd...

2026-05-17T20:21:57.169881+00:00 LEE-Ubuntu-01 systemd[1]: Reached target network.target - Network.

2026-05-17T20:21:58.801356+00:00 LEE-Ubuntu-01 systemd[1]: Reached target network-online.target - Network is Online.

2026-05-20T17:51:51.058802-04:00 LEE-Ubuntu-01 systemd-networkd[5462]: ens33: Link UP

2026-05-20T17:51:51.059348-04:00 LEE-Ubuntu-01 systemd-networkd[5462]: ens33: Gained carrier

2026-05-20T17:51:51.064853-04:00 LEE-Ubuntu-01 systemd-networkd[5462]: ens33: Gained IPv6LL
```

## Observations

- Linux networking stack initialized successfully during boot
- `ens33` virtual VMware network interface became operational
- network carrier was successfully established
- IPv6 link-local address assignment occurred automatically
- Linux networking services reached operational online state

## Security Significance

Important networking concepts observed:
- network interface lifecycle telemetry
- interface state transitions
- network readiness sequencing
- virtual NIC initialization
- systemd-networkd operational behavior

Operational relevance:
Understanding normal network initialization behavior is critical for identifying:
- interface failures
- link flapping
- DHCP issues
- suspicious network changes
- connectivity anomalies

The interface `ens33` represents the VMware virtual network adapter assigned to the Ubuntu VM.

---

# Baseline Findings

The following baseline behaviors were observed during analysis of `/var/log/syslog`:

- orderly Linux service startup and shutdown activity
- normal systemd operational telemetry
- expected virtualization-related hardware warnings
- successful AppArmor security policy enforcement
- normal Linux networking initialization behavior
- successful VMware virtual NIC activation
- expected desktop environment operational noise
- standard socket and timer activity
- normal resource utilization telemetry

---

# Key Takeaways

- syslog contains broad operational telemetry across the Linux ecosystem
- Linux systems continuously generate large volumes of normal operational noise
- not all "error" or "DENIED" entries indicate compromise activity
- AppArmor enforcement events often represent healthy security controls
- virtualization environments generate expected hardware abstraction warnings
- network initialization produces valuable interface and connectivity telemetry
- establishing baseline operational behavior is essential for anomaly detection
- observability intuition develops through repeated exposure to normal system behavior
