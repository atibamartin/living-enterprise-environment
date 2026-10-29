# LEE — Wazuh Deployment and Agent Integration

**Project:** Living Enterprise Environment (LEE)  
**Phase:** 3 — Security Monitoring Foundations  
**Platform:** Wazuh 4.14.9  
**Status:** Deployment complete; telemetry validation in progress  
**Deployment period:** October 2026

## 1. Purpose

Deploy a centralized security monitoring platform capable of collecting, analyzing, and correlating endpoint security events across the segmented Living Enterprise Environment.

The deployment establishes a foundation for SOC investigations, configuration assessment, file integrity monitoring, vulnerability management, and future detection engineering.

## 2. Architecture

| System | Network | Address | Function |
|---|---|---|---|
| Wazuh Server | VMware NAT | 192.168.133.133 | Security monitoring platform |
| LEE-WIN-SRV-01 | SERVER | 192.168.10.10 | Windows Server 2025, Active Directory |
| LEE-Ubuntu-01 | MANAGEMENT | 192.168.40.10 | Ubuntu 26.04 |
| LEE-WIN-CLT-01 | CLIENT | 192.168.20.101 | Windows 11 domain workstation |
| pfSense | Multiple interfaces | Zone gateways | Segmentation and firewall enforcement |

All three endpoints run Wazuh agent version 4.14.9.

## 3. Wazuh Manager Deployment

The Wazuh 4.14.9 virtual appliance was imported into VMware Workstation and configured on the VMware NAT network.

Deployment activities included:

- Adjusting virtual machine compatibility and resources.
- Configuring the manager address as `192.168.133.133`.
- Verifying network connectivity.
- Reviewing and resolving storage capacity constraints.
- Confirming Wazuh manager and supporting services were running.
- Accessing the Wazuh web dashboard.
- Changing the initial administrative credentials.
- Reviewing initial system-generated alerts.

**Result:** Wazuh Manager operational and accessible through the dashboard.

## 4. Network Security and Firewall Integration

The segmented environment uses pfSense to control traffic between endpoint networks and the Wazuh Manager.

Required endpoint communication:

| Protocol | Destination port | Purpose |
|---|---|---|
| TCP | 1514 | Agent-to-manager communication |
| TCP | 1515 | Agent enrollment |
| TCP | 443 | HTTPS access where explicitly required |

Rules were created on the appropriate source interfaces, using scoped source and destination addresses or aliases.

### Troubleshooting observations

Initial communication attempts were blocked by pfSense default-deny rules.

Firewall logs identified the source IP, destination IP, protocol, and destination port. Rules were added or corrected, followed by repeated connectivity testing.

The Windows client changed DHCP addresses from `192.168.20.100` to `192.168.20.101`. An IP-specific rule therefore required updating.

**Lesson learned:** A correctly defined firewall rule can still fail when the endpoint's actual source address no longer matches the rule.

## 5. Windows Server Agent

**Endpoint:** LEE-WIN-SRV-01  
**Address:** 192.168.10.10  
**Operating system:** Windows Server 2025  
**Wazuh agent:** 4.14.9  
**Agent ID:** 001

The Windows installer was obtained and its digital signature verified.

Installation used the Wazuh Manager address and an explicit agent name.

Validation included:

- Confirming the agent service was installed.
- Starting the Wazuh service.
- Verifying network connectivity.
- Confirming the endpoint appeared in the Wazuh dashboard.
- Reviewing initial configuration assessment and vulnerability findings.

**Result:** Agent registered and active.

## 6. Ubuntu Agent

**Endpoint:** LEE-Ubuntu-01  
**Address:** 192.168.40.10  
**Operating system:** Ubuntu 26.04  
**Wazuh agent:** 4.14.9  
**Agent ID:** 002

### DNS troubleshooting

Initial package operations failed because Ubuntu could not resolve repository hostnames.

Investigation identified missing DNS configuration and blocked DNS traffic to the pfSense management interface.

The configuration was updated to use:

`192.168.40.1`

The required TCP/UDP port 53 firewall rule was created, and DNS resolution was validated.

Persistent DNS configuration was added through Netplan.

### Agent installation

The Wazuh repository was configured and package availability verified.

Installation command:

```bash
sudo env WAZUH_MANAGER="192.168.133.133" \
  WAZUH_AGENT_NAME="LEE-Ubuntu-01" \
  apt-get install wazuh-agent=4.14.9-1
```

The agent was enabled and started:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-agent
```

The installed version was verified with `dpkg`.

The package was placed on hold to prevent unintended APT upgrades:

```bash
sudo apt-mark hold wazuh-agent
```

The hold does not disable agent telemetry transmission.

### Validation

The agent log confirmed successful connection to `192.168.133.133:1514/tcp`.

Wazuh displayed the endpoint as active.

### Configuration assessment limitation

The agent loaded an Ubuntu 22.04 CIS assessment policy but skipped execution because the operating system was Ubuntu 26.04.

This is a policy compatibility limitation, not evidence of a successful compliance assessment.

A compatible assessment policy remains an outstanding task.

## 7. Windows Client Agent

**Endpoint:** LEE-WIN-CLT-01  
**Address:** 192.168.20.101  
**Operating system:** Windows 11 Enterprise Evaluation  
**Wazuh agent:** 4.14.9  
**Agent ID:** 003

### Pre-installation validation

Confirmed that the Wazuh service and installation directory did not already exist.

The Wazuh installer was downloaded over HTTPS.

The installer signature was checked using:

```powershell
Get-AuthenticodeSignature $installer
```

Result: `Valid`, with Wazuh, Inc. identified as the signer.

### Installation

```powershell
$arguments = @(
    '/i'
    "`"$installer`""
    '/qn'
    '/norestart'
    'WAZUH_MANAGER=192.168.133.133'
    'WAZUH_AGENT_NAME=LEE-WIN-CLT-01'
)

$process = Start-Process msiexec.exe `
    -ArgumentList $arguments `
    -Wait `
    -PassThru

$process.ExitCode
```

The installer returned exit code `0`.

The Wazuh service and installation directory were confirmed.

The service was subsequently started, and the endpoint appeared in the Wazuh dashboard.

### Firewall validation

Initial tests to TCP ports 1514 and 1515 failed.

Following the scoped CLIENT firewall rule, both ports returned successful connectivity results.

pfSense logs showed permitted traffic from `192.168.20.101` to `192.168.133.133`.

### Configuration validation

The client agent configuration identified:

- Manager: `192.168.133.133`
- Port: `1514`
- Protocol: `tcp`

**Result:** Windows client registered and active.

## 8. Authentication Detection Proof of Concept

A controlled SSH authentication failure was generated locally on the Ubuntu endpoint.

The test used a nonexistent username, `auth.test`, against `127.0.0.1`.

Ubuntu recorded the activity through SSH and PAM.

Wazuh subsequently generated:

| Rule ID | Description | Level |
|---|---|---|
| 5710 | SSH attempt using a nonexistent user | 5 |
| 5503 | PAM user login failed | 5 |

The alert included the Ubuntu agent identity, source address `127.0.0.1`, original log message, decoder, timestamp, and MITRE ATT&CK mapping.

**MITRE ATT&CK:** T1110.001 — Password Guessing.

This demonstrated a successful endpoint-to-SIEM detection workflow.

## 9. Recovery and Baseline Preservation

VMware snapshots were taken to preserve working system states.

A pfSense configuration backup was also downloaded.

These provide rollback and configuration recovery options. Snapshots should not be treated as substitutes for independent backups.

## 10. Outstanding Activities

- Validate security telemetry on both Windows endpoints.
- Confirm Windows security auditing configuration.
- Review Linux sudo and privilege escalation events.
- Verify event field extraction and timestamp consistency.
- Evaluate file integrity monitoring.
- Investigate the unusually large vulnerability counts and distinguish actual exposure from duplicated or otherwise misleading records.
- Establish an Ubuntu 26.04-compatible security configuration assessment policy.
- Review endpoint resource consumption.
- Review DHCP reservation or stable addressing for firewall-scoped clients.
- Complete the living findings register.
- Capture final deployment evidence and diagrams.

## 11. Conclusion

The Wazuh deployment successfully integrated three endpoints across separate network segments into a centralized monitoring platform.

The implementation demonstrated the relationship between endpoint configuration, DNS, package management, firewall enforcement, agent registration, log collection, and security alert generation.

The next objective is to establish reliable telemetry baselines before introducing controlled adversary simulations.
