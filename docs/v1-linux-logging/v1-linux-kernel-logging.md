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

## Environment Details

Host Name: LEE-Ubuntu-01
Operating System: Ubuntu Server 26.04
Virtualization Platform: VMware Workstation Pro
Allocated Memory: 4 GB
Disk Size: 60 GB
Purpose: Living Enterprise Environment (LEE)

The server acts as the foundational Linux system for the Living Enterprise Environment (LEE) and will continue evolving throughout future labs involving:

- SIEM ingestion
- detection engineering
- segmentation
- authentication analysis
- threat simulation

# Boot Identity
---

## Commands

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
---

# Kernel / Boot Identity Baseline

- Hostname: `LEE-Ubuntu-01`
- Kernel: `7.0.0-15-generic`
- Architecture: `x86_64`
- Build: `#15-Ubuntu SMP PREEMPT_DYNAMIC`
- Boot image: `/boot/vmlinuz-7.0.0-15-generic`
- Root device: `UUID=0ed0761d-0447-4c73-8163-ec5a1e141e30`
- NX protection: `active` ( memory execution protection is enabled)
- SMBIOS: `2.7 present`



# Assessment:
Kernel and boot identity information was successfully captured.  Kernel boot output confirmed the BIOS-provided memory map, supported CPU vendors, and SMBIOS presence. No abnormal boot identity issues were observed in the reviewed output.


---

# Disk Storage & Behavior
---

## Commands

```bash
sudo dmesg -T | grep -Ei "sda|vda|nvme|ext4|filesystem|mount|I/O error|blk|disk"
journalctl -b 0 -k | grep -Ei
lsblk

```

## Log Output

```text
[Tue Jun 16 17:30:49 2026] RAMDISK: [mem 0x332c1000-0x35957fff]
[Tue Jun 16 17:30:50 2026] Mount-cache hash table entries: 8192 (order: 4, 65536 bytes, linear)
[Tue Jun 16 17:30:50 2026] Mountpoint-cache hash table entries: 8192 (order: 4, 65536 bytes, linear)
[Tue Jun 16 17:30:50 2026] VFS: Finished mounting rootfs on nullfs
[Tue Jun 16 17:30:52 2026] VFS: Disk quotas dquot_6.6.0
[Tue Jun 16 17:30:52 2026] AppArmor: AppArmor Filesystem Enabled
[Tue Jun 16 17:30:53 2026] systemd[1]: systemd 259.5-0ubuntu3 running in system mode (+PAM +AUDIT +SELINUX +APPARMOR +IMA +IPE +SMACK +SECCOMP +GCRYPT -GNUTLS +OPENSSL +ACL +BLKID +CURL +ELFUTILS +FIDO2 +IDN2 -IDN +KMOD +LIBCRYPTSETUP +LIBCRYPTSETUP_PLUGINS +LIBFDISK +PCRE2 +PWQUALITY +P11KIT +QRENCODE +TPM2 +BZIP2 +LZ4 +XZ +ZLIB +ZSTD +BPF_FRAMEWORK +BTF -XKBCOMMON -UTMP +SYSVINIT +LIBARCHIVE)
[Tue Jun 16 17:30:53 2026] systemd[1]: Expecting device dev-disk-by\x2duuid-0ed0761d\x2d0447\x2d4c73\x2d8163\x2dec5a1e141e30.device - /dev/disk/by-uuid/0ed0761d-0447-4c73-8163-ec5a1e141e30...
[Tue Jun 16 17:30:55 2026] sd 2:0:0:0: [sda] 125829120 512-byte logical blocks: (64.4 GB/60.0 GiB)
[Tue Jun 16 17:30:55 2026] sd 2:0:0:0: [sda] Write Protect is off
[Tue Jun 16 17:30:55 2026] sd 2:0:0:0: [sda] Mode Sense: 61 00 00 00
[Tue Jun 16 17:30:55 2026] sd 2:0:0:0: [sda] Cache data unavailable
[Tue Jun 16 17:30:55 2026] sd 2:0:0:0: [sda] Assuming drive cache: write through
[Tue Jun 16 17:30:55 2026]  sda: sda1 sda2
[Tue Jun 16 17:30:55 2026] sd 2:0:0:0: [sda] Attached SCSI disk
[Tue Jun 16 17:30:56 2026] EXT4-fs (sda2): orphan cleanup on readonly fs
[Tue Jun 16 17:30:56 2026] EXT4-fs (sda2): mounted filesystem 0ed0761d-0447-4c73-8163-ec5a1e141e30 ro with ordered data mode. Quota mode: none.
[Tue Jun 16 17:30:57 2026] systemd[1]: systemd 259.5-0ubuntu3 running in system mode (+PAM +AUDIT +SELINUX +APPARMOR +IMA +IPE +SMACK +SECCOMP +GCRYPT -GNUTLS +OPENSSL +ACL +BLKID +CURL +ELFUTILS +FIDO2 +IDN2 -IDN +KMOD +LIBCRYPTSETUP +LIBCRYPTSETUP_PLUGINS +LIBFDISK +PCRE2 +PWQUALITY +P11KIT +QRENCODE +TPM2 +BZIP2 +LZ4 +XZ +ZLIB +ZSTD +BPF_FRAMEWORK +BTF -XKBCOMMON -UTMP +SYSVINIT +LIBARCHIVE)
[Tue Jun 16 17:30:57 2026] systemd[1]: Set up automount proc-sys-fs-binfmt_misc.automount - Arbitrary Executable File Formats File System Automount Point.
[Tue Jun 16 17:30:57 2026] systemd[1]: Reached target snapd.mounts-pre.target - Mounting snaps.
[Tue Jun 16 17:30:57 2026] systemd[1]: Mounting dev-hugepages.mount - Huge Pages File System...
[Tue Jun 16 17:30:57 2026] systemd[1]: Mounting dev-mqueue.mount - POSIX Message Queue File System...
[Tue Jun 16 17:30:57 2026] systemd[1]: Mounting sys-kernel-debug.mount - Kernel Debug File System...
[Tue Jun 16 17:30:57 2026] systemd[1]: Mounting sys-kernel-tracing.mount - Kernel Trace File System...
[Tue Jun 16 17:30:57 2026] systemd[1]: Mounting sys-fs-fuse-connections.mount - FUSE Control File System...
[Tue Jun 16 17:30:57 2026] systemd[1]: Starting systemd-remount-fs.service - Remount Root and Kernel File Systems...
[Tue Jun 16 17:30:58 2026] EXT4-fs (sda2): re-mounted 0ed0761d-0447-4c73-8163-ec5a1e141e30 r/w.

atibam@LEE-Ubuntu-01:~$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  66.8M  1 loop /snap/core24/1587
loop1    7:1    0     4K  1 loop /snap/bare/5
loop2    7:2    0    20M  1 loop /snap/desktop-security-center/151
loop3    7:3    0 273.7M  1 loop /snap/firefox/8107
loop4    7:4    0  66.8M  1 loop /snap/core24/1643
loop5    7:5    0  19.6M  1 loop /snap/desktop-security-center/150
loop6    7:6    0  16.5M  1 loop /snap/firmware-updater/226
loop7    7:7    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop8    7:8    0   395M  1 loop /snap/mesa-2404/1165
loop9    7:9    0 606.1M  1 loop /snap/gnome-46-2404/153
loop10   7:10   0  18.8M  1 loop /snap/prompting-client/222
loop11   7:11   0  15.7M  1 loop /snap/snap-store/1367
loop12   7:12   0  18.8M  1 loop /snap/prompting-client/204
loop13   7:13   0  49.3M  1 loop /snap/snapd/26865
loop14   7:14   0   580K  1 loop /snap/snapd-desktop-integration/361
sda      8:0    0    60G  0 disk 
├─sda1   8:1    0     1M  0 part 
└─sda2   8:2    0    60G  0 part /
sr0     11:0    1   6.1G  0 rom  /run/media/atibam/Ubuntu 26.04 amd64

```

---

# Findings

- VM Disk: `sda`
- Partitions: `sda1 & sda2`
- Main Filesystem: `EXT4 on sda2`
- Disk Size: `64.4 GB / 60.0 GiBHave`
- Boot image: `/boot/vmlinuz-7.0.0-15-generic`
- Root device: `UUID=0ed0761d-0447-4c73-8163-ec5a1e141e30`


### Assessment

The system detected one SCSI disk identified as `/dev/sda` with a size of 64.4 GB / 60.0 GiB. Two partitions were detected: `sda1` and `sda2`.
The root filesystem appears to be mounted on `sda2` using EXT4. During boot, the filesystem was first mounted read-only and later re-mounted read/write, which is normal Linux boot behavior.
No disk I/O errors, filesystem corruption errors, or critical storage failures were observed in the reviewed kernel boot logs.
AppArmor denial events observed for fusermount3. These appear related to mount/FUSE behavior and should be baselined for comparison against future abnormal events

## Questions Answered

- Did the disk get detected?
- What filesystem is used?
- Did it mount correctly?
- Any corruption?
- Any disk failures?


---

# Network Interface Behavior
---

## Commands

```bash
ip addr
journalctl -b 0 -k | grep ens33
lspci | grep -i ethernet

```

## Log Output

```text
atibam@LEE-Ubuntu-01:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:75:c4:58 brd ff:ff:ff:ff:ff:ff
    inet 192.168.133.129/24 brd 192.168.133.255 scope global dynamic noprefixroute ens33
       valid_lft 1635sec preferred_lft 1635sec
    inet6 fe80::20c:29ff:fe75:c458/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever


found the interface name ens33

tibam@LEE-Ubuntu-01:~$ journalctl -b 0 -k | grep ens33
Jun 16 17:30:06 LEE-Ubuntu-01 kernel: e1000 0000:02:01.0 ens33: renamed from eth0
Jun 16 17:30:07 LEE-Ubuntu-01 kernel: e1000: ens33 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None
Jun 16 17:33:43 LEE-Ubuntu-01 kernel: e1000: ens33 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None

atibam@LEE-Ubuntu-01:~$ lspci | grep -i ethernet
02:01.0 Ethernet controller: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) (rev 01)
atibam@LEE-Ubuntu-01:~$ 
tha


```

---

# Findings

- Interface name: `ens33`
- Link state messages: `NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None`
- Errors found: none
- IP address: `192.168.133.129/24`
- MAC address:  `00:0c:29:75:c4:58`
- Driver: `e1000`
- Adapter: `
- Status: `UP`
- Interface Type: `Broadcast / Multicast Ethernet Interface`
- Notes: `No network-related kernel errors observed during initial baseline review.`


### Assessment

We identified the netwok interface name and status.  There were no network driver errors and functionality. We were able to identify 
the interface type, duplex and bandwidth.  The interface initialized as eth0 and was renamed during boot to ens33.  No network related
kernel warnings or errors were observed.

## Questions Answered

- Did the NIC initialize?
- Did the link come up?
- Did the driver load?
- Any network hardware issues?




----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


---

# Warnings and Errors
---

## Commands

```bash
journalctl -b 0 -k -p warning
journalctl -b 0 -k -p err
grep -Ei "error|warn|fail|failed|critical|panic|segfault|BUG|oom" /var/log/kern.log
```

## Log Output

```text
atibam@LEE-Ubuntu-01:~$ journalctl -b 0 -k -p warning
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: [Firmware Bug]: TSC doesn't count with P0 frequency!
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Speculative Return Stack Overflow: IBPB-extending microcode not applied!
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Speculative Return Stack Overflow: WARNING: See https://kernel.org/doc/html/latest/admin-guide/hw-vuln/srso.html for mitigation options.
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: amd_pstate: The CPPC feature is supported but currently disabled by the BIOS.
                                      Please enable it if your BIOS has the CPPC option.
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: amd_pstate: the _CPC object is not present in SBIOS or ACPI disabled
Jun 16 17:29:55 LEE-Ubuntu-01 kernel: sd 2:0:0:0: [sda] Assuming drive cache: write through
Jun 16 17:30:04 LEE-Ubuntu-01 kernel: piix4_smbus 0000:00:07.3: SMBus Host Controller not enabled!
Jun 16 17:30:06 LEE-Ubuntu-01 kernel: kauditd_printk_skb: 230 callbacks suppressed
Jun 16 17:30:07 LEE-Ubuntu-01 kernel: set_capacity_and_notify: 5 callbacks suppressed
Jun 16 17:33:57 LEE-Ubuntu-01 kernel: /dev/sr0: Can't open blockdev
atibam@LEE-Ubuntu-01:~$ 
atibam@LEE-Ubuntu-01:~$

atibam@LEE-Ubuntu-01:~$ journalctl -b 0 -k -p err
Jun 16 17:30:04 LEE-Ubuntu-01 kernel: piix4_smbus 0000:00:07.3: SMBus Host Controller not enabled!
Jun 16 17:33:57 LEE-Ubuntu-01 kernel: /dev/sr0: Can't open blockdev
atibam@LEE-Ubuntu-01:~$ 
atibam@LEE-Ubuntu-01:~$

atibam@LEE-Ubuntu-01:~$ grep -Ei "error|warn|fail|failed|critical|panic|segfault|BUG|oom" /var/log/kern.log
atibam@LEE-Ubuntu-01:~$ 
atibam@LEE-Ubuntu-01:~$ 

```

---

# Warnings/Errors Findings

- No evidence of compromise
- Expected Informational security finding
- Kernel reported SRSO mitigation warning with no indication of active exploitation
- Storage related error:
--   sd 2:0:0:0: [sda] Assuming drive cache: write through (Virtual disk reporting how write caching is handled)
- No alerts/warnings/errors found for the following, Kernel panic, segfault, OOM killer, memory allocation failure, disk I/O error


### Warnings/Errors Assessment

When running a grep command to search for the items that will point to an issue or failure, there were no results/output.
The reviewed warnings appear consistent with expected VMware virtualization behavior and do not indicate active system instability or compromise.







----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Memory Behavior
---

## Commands

```bash
free -h
vmstat
journalctl -b 0 -k | grep -i oom
journalctl -b 0 -k | grep -Ei "memory|swap"
journalctl -b 0 -k | grep -Ei "allocation|out of memory"
```

## Log Output

```text
tibam@LEE-Ubuntu-01:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.3Gi       1.4Gi       654Mi        22Mi       1.5Gi       1.9Gi
Swap:          3.8Gi          0B       3.8Gi
atibam@LEE-Ubuntu-01:~$

atibam@LEE-Ubuntu-01:~$ vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st gu
 2  0      0 670464  38908 1543776    0    0   137     8  232    1  0  1 99  0  0  0
atibam@LEE-Ubuntu-01:~$ 
atibam@LEE-Ubuntu-01:~$

atibam@LEE-Ubuntu-01:~$ journalctl -b 0 -k | grep -i oom
Jun 16 17:29:58 LEE-Ubuntu-01 systemd[1]: Listening on systemd-oomd.socket - Userspace Out-Of-Memory (OOM) Killer Socket.
atibam@LEE-Ubuntu-01:~$
atibam@LEE-Ubuntu-01:~$ journalctl -b 0 -k | grep -Ei "memory|swap"
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: DMI: Memory slots populated: 1/128
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Early memory node ranges
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Freeing SMP alternatives memory: 56K
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Memory: 1313528K/4193716K available (23771K kernel code, 4949K rwdata, 16748K rodata, 5292K init, 5860K bss, 2853784K reserved, 0K cma-reserved)
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: x86/mm: Memory block size: 128MB
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Freeing initrd memory: 39516K
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Freeing unused decrypted memory: 2028K
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Freeing unused kernel image (initmem) memory: 5292K
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Freeing unused kernel image (text/rodata gap) memory: 804K
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: Freeing unused kernel image (rodata/data gap) memory: 1684K
Jun 16 17:29:54 LEE-Ubuntu-01 systemd[1]: Reached target swap.target - Swaps.
Jun 16 17:29:58 LEE-Ubuntu-01 systemd[1]: Listening on systemd-oomd.socket - Userspace Out-Of-Memory (OOM) Killer Socket.
Jun 16 17:29:58 LEE-Ubuntu-01 kernel: Adding 3954684k swap on /swap.img.  Priority:-1 extents:4 across:4478972k 
Jun 16 17:29:58 LEE-Ubuntu-01 kernel: vmwgfx 0000:00:0f.0: [drm] Legacy memory limits: VRAM = 4096 KiB, FIFO = 256 KiB, surface = 0 KiB
Jun 16 17:29:58 LEE-Ubuntu-01 kernel: vmwgfx 0000:00:0f.0: [drm] Maximum display memory size is 262144 KiB

atibam@LEE-Ubuntu-01:~$ journalctl -b 0 -k | grep -Ei "allocation|out of memory"
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: DMA: preallocated 256 KiB GFP_KERNEL pool for atomic allocations
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: DMA: preallocated 256 KiB GFP_KERNEL|GFP_DMA pool for atomic allocations
Jun 16 17:29:54 LEE-Ubuntu-01 kernel: DMA: preallocated 256 KiB GFP_KERNEL|GFP_DMA32 pool for atomic allocations

```

---

# Memory Findings

- Total Memory: `3.3Gi`
- Available Memory: `1.9Gi`
- Swap: `3.8Gi`
- Swap Used: `0B`
- CPU idle: `99%`
- I/O wait: `0%`
- Interface Type: `Broadcast / Multicast Ethernet Interface`
- Notes: `No network-related kernel errors observed during initial baseline review.



### Memory Baseline

The system has 3.3 GiB of available RAM and 3.8 GiB of configured swap. At the time of review, 1.4 GiB of RAM was used, 1.9 GiB was available, and swap usage was 0B.
`vmstat` showed no swap-in or swap-out activity, with CPU idle at 99% and I/O wait at 0%, indicating no observable memory or performance pressure during baseline review.
Kernel logs showed memory reservation, initialization, and swap activation events during boot. No Out-of-Memory kills, memory allocation failures, kernel panics, or memory-related critical errors were observed.

### Assessment:
Memory state appears healthy for the current VM workload. No evidence of memory exhaustion, swap pressure, or OOM activity was observed.

### Questions answered:

- Did the system run out of memory?
- Were processes killed?
- Any memory pressure?

---


# Security Behavior
---

## Commands

```bash
grep -Ei "apparmor|audit|denied|blocked|drop|reject|iptables|nftables|ufw" /var/log/kern.log
sudo aa-status

```

## Log Output

```text
atibam@LEE-Ubuntu-01:~$ grep -Ei "apparmor|audit|denied|blocked|drop|reject|iptables|nftables|ufw" /var/log/kern.log
2026-06-16T18:00:59.967880-04:00 LEE-Ubuntu-01 kernel: audit: type=1400 audit(1781647259.965:263): apparmor="DENIED" operation="open" class="file"
b-aprofile="snap.firmware-updater.firmware-notifier" name="/proc/sys/vm/max_map_count" pid=5452 comm="firmware-notifi" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0


atibam@LEE-Ubuntu-01:~$ sudo aa-status
3 profiles are in complain mode.
   /usr/sbin/sssd
   Xorg
   Xorg_wrap
0 profiles are in prompt mode.
0 profiles are in kill mode.
75 profiles are in unconfined mode.
9 processes have profiles defined.
8 processes are in enforce mode.
   /usr/sbin/chronyd (1676) 
   /usr/sbin/chronyd (1687) 
   /usr/sbin/cups-browsed (5434) 
   /usr/sbin/cupsd (5433) 
   /usr/bin/fusermount3 (3722) fusermount3
   /usr/sbin/rsyslogd (1581) rsyslogd
   /snap/snapd-desktop-integration/361/usr/bin/snapd-desktop-integration (4587) snap.snapd-desktop-integration.snapd-desktop-integration
   /snap/snapd-desktop-integration/361/usr/bin/snapd-desktop-integration (4692) snap.snapd-desktop-integration.snapd-desktop-integration
0 processes are in complain mode.
0 processes are in prompt mode.
0 processes are in kill mode.
1 processes are unconfined but have a profile defined.
   /usr/bin/gjs-console (6828) desktop-icons-ng
0 processes are in mixed mode.


```

---

## Observed Event:

- AppArmor denied read access to `/proc/sys/vm/max_map_count`
- Process: `firmware-notifier`
- Profile: `snap.firmware-updater.firmware-notifier`

### AppArmor Baseline

AppArmor is enabled and actively enforcing security policies.

### Findings:

- 234 profiles loaded
- 156 profiles operating in enforce mode
- 3 profiles operating in complain mode
- 75 profiles unconfined

### Observed Protected Services:
- rsyslogd
- chronyd
- cupsd
- fusermount3

### Security Events:

A single AppArmor DENIED event was observed involving the Snap firmware updater attempting to access `/proc/sys/vm/max_map_count`. The request was successfully blocked according to policy.

### Assessment:

AppArmor is functioning as expected and actively enforcing security controls. No evidence of unauthorized access attempts, privilege escalation, firewall denials, or malicious activity was observed.

### Questions answered:

- Was something denied?
- Was security software triggered?
- Were capabilities blocked?

---
### Log Rotation

I evaluated the current configuration for log rotation.  It was set at it's default setting of saving logs weekly keeping the 
last 4 logs.  I have updated so that logs are saved daily and the last 30 files are kept.


## Configuration
```text
atibam@LEE-Ubuntu-01:/$ cat /etc/logrotate.d/rsyslog
/var/log/syslog
/var/log/mail.log
/var/log/kern.log
/var/log/auth.log
/var/log/user.log
/var/log/cron.log
{
	rotate 30
	daily
	missingok
	notifempty
	compress
	delaycompress
	sharedscripts
	postrotate
		/usr/lib/rsyslog/rsyslog-rotate
	endscript
}
```

---

## Kernel Log Baseline Summary

The purpose of this review was to establish a baseline of normal kernel activity for LEE-Ubuntu-01. During the evaluation of kernel log data, the system was found to be operating in a healthy state with behavior consistent with a newly deployed Ubuntu virtual machine that has had limited exposure to external systems and minimal software installed beyond the default operating system components.

The analysis focused on the following areas:

* Hardware and Virtualization
* Storage
* Filesystem
* Network
* Memory
* Warnings and Errors
* Security Controls (AppArmor)

Across the areas reviewed, no critical kernel errors, kernel panics, memory exhaustion events, storage failures, or network-related issues were identified. AppArmor was confirmed to be active and enforcing security policies as expected.

The sections that follow document the questions used to guide the analysis, the commands executed to gather evidence, the relevant log entries identified, and a summary of the findings for each category. This baseline will serve as a point of comparison for future investigations, system changes, threat simulations, and security monitoring activities within the Living Enterprise Environment (LEE).

