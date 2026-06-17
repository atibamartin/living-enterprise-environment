# V1 - Linux Logging + Establishing Baseline for System Logs (kernel.log)

## Objective

Learn Linux system logging and grep analysis while establishing a baseline for normal operational telemetry generated within `/var/log/kernel.log`.

This evaluation focuses on documenting:
- Kernel version, boot sesssion and boot messages
- Storage elements
- Network related information
- Errors and Warnings
- Memory and performance
- Security Related Kernel Events


This analysis was performed on:

`LEE-Ubuntu-01`

The server acts as the foundational Linux system for the Living Enterprise Environment (LEE) and will continue evolving throughout future labs involving:

- SIEM ingestion
- detection engineering
- segmentation
- authentication analysis
- threat simulation

# Identity
---

# Commands

```bash
uname -a
journalctl -b 0 -k | head -n 30
```

## Log Output

```text
Linux LEE-Ubuntu-01 7.0.0-15-generic #15-Ubuntu SMP PREEMPT_DYNAMIC Wed Apr 22 16:06:43 UTC 2026 x86_64 GNU/Linux

atibam@LEE-Ubuntu-01:~$ journalctl -b 0 -k | head -n 30
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Linux version 7.0.0-15-generic (buildd@lcy02-amd64-048) (x86_64-linux-gnu-gcc
(Ubuntu 15.2.0-16ubuntu1) 15.2.0, GNU ld (GNU Binutils for Ubuntu) 2.46) #15-Ubuntu SMP PREEMPT_DYNAMIC Wed Apr 22
16:06:43 UTC 2026 (Ubuntu 7.0.0-15.15-generic 7.0.0)
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Command line: BOOT_IMAGE=/boot/vmlinuz-7.0.0-15-generic
Souroot=UUID=0ed0761d-0447-4c73-8163-ec5a1e141e30 ro quiet splash crashkernel=2G-4G:320M,4G-32G:512M,32G-64G:1024M,64G-128G:2048M,128G-:4096M

```

## Observations

Hostname: `LEE-Ubuntu-01`
Kernel: `7.0.0-15-generic`
Architecture: `x86_64`
Build: `#15-Ubuntu SMP PREEMPT_DYNAMIC`
Boot image: `/boot/vmlinuz-7.0.0-15-generic`
Root device: `UUID=0ed0761d-0447-4c73-8163-ec5a1e141e30`
NX protection: `active`
SMBIOS: `2.7 present`

```

## Kernel / Boot Identity Baseline

The system is identified as `LEE-Ubuntu-01` and is running Linux kernel `7.0.0-15-generic` on an `x86_64` architecture.
The kernel build is listed as `#15-Ubuntu SMP PREEMPT_DYNAMIC Wed Apr 22 16:06:43 UTC 2026`.

The system booted using `/boot/vmlinuz-7.0.0-15-generic` with the root filesystem identified by
UUID `0ed0761d-0447-4c73-8163-ec5a1e141e30`. Kernel boot output confirmed the BIOS-provided memory map, supported CPU vendors, and SMBIOS presence.

NX Execute Disable protection was reported as active, indicating memory execution protection is enabled.

Assessment:
Kernel and boot identity information was successfully captured. No abnormal boot identity issues were observed in the reviewed output.


---

# Identity
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
