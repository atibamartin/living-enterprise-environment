# Windows Client Build and Domain Integration

## Overview

`LEE-WIN-CLT-01` was deployed as the first Windows workstation in the Living Enterprise Environment (LEE).

The purpose of the system is to provide a realistic enterprise endpoint that can authenticate against Active Directory, receive domain services, access network resources, generate Windows security telemetry, and later serve as a target and source for controlled security testing.

The workstation extends LEE from a server-focused Active Directory environment into a client/server enterprise environment.

---

## System Role

| Setting | Configuration |
|---|---|
| Hostname | `LEE-WIN-CLT-01` |
| Operating System | Windows 11 Enterprise |
| Domain | `livingenterprise.lab` |
| Domain Controller | `LEE-WIN-SRV-01` |
| Network | VMware NAT / VMnet8 |
| IPv4 Address | `192.168.133.131/24` |
| Default Gateway | `192.168.133.2` |
| DNS Server | `192.168.133.130` |
| DHCP Server | `192.168.133.254` |

The workstation receives its network configuration through DHCP while using the LEE domain controller as its DNS server.

---

## Virtual Machine Deployment

The Windows 11 virtual machine was created in VMware Workstation Pro as a standard enterprise workstation.

The initial virtual machine configuration included:

- Windows 11 Enterprise
- 64 GB virtual disk
- Split virtual disk files
- VMware NAT networking
- Virtual TPM support
- VMware Tools
- Intel 82574L virtual network adapter

A local administrative account was created during Windows setup to provide initial workstation administration before domain integration.

VMware Tools was installed and validated after the operating system installation.

```powershell
Get-Service VMTools
