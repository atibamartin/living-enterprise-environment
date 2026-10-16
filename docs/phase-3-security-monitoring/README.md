# Phase 3 — Security Monitoring Foundations

**Project:** Living Enterprise Environment (LEE)  
**Status:** In Progress  
**Primary Platform:** Wazuh 4.14.9  
**Hypervisor:** VMware Workstation Pro  
**Network Security:** pfSense

## 1. Overview

Phase 3 introduces centralized security monitoring into the Living Enterprise Environment.

Previous phases established Windows and Linux infrastructure, Active Directory authentication, network connectivity, segmentation, and firewall policies.

This phase builds on those foundations by deploying Wazuh as a centralized platform for collecting, analyzing, and investigating security telemetry.

The objective is not simply to install monitoring software, but to understand how endpoint activity becomes actionable security information.

## 2. Objectives

- Deploy and configure a Wazuh monitoring appliance.
- Integrate Wazuh into the existing LEE network architecture.
- Establish secure communication between monitored endpoints and Wazuh.
- Collect Windows and Linux security events.
- Validate event ingestion and agent connectivity.
- Generate controlled security activity.
- Investigate authentication failures and other security events.
- Establish a foundation for future detection engineering.

## 3. Planned Architecture

**Monitoring Server:** LEE-WAZUH-01

**Operating System:** Amazon Linux 2023

**Wazuh Components:**
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

**Initial Network:** VMware VMnet8 (temporary deployment)

**Target Network:** Management / Security Monitoring Zone, subject to validation of the Phase 2 pfSense architecture.

**Planned Monitoring Sources:**
- LEE-WIN-SRV-01 — Windows Server / Domain Controller
- LEE-WIN-CLT-01 — Windows 11 Client
- LEE-Ubuntu-01 — Linux Management System
- pfSense — Firewall logs (future integration)

Kali Linux will primarily function as a controlled activity-generation system rather than an initial monitoring endpoint.

## 4. Version 7 — Wazuh Deployment

Activities:

1. Import and configure the Wazuh OVA.
2. Validate operating system and Wazuh services.
3. Configure network connectivity.
4. Secure default credentials.
5. Establish dashboard access.
6. Integrate with the segmented LEE network.
7. Deploy endpoint agents.
8. Validate agent enrollment and event ingestion.
9. Capture evidence and create a baseline snapshot.

**Completion criteria:** Wazuh receives and displays security telemetry from designated LEE endpoints through approved network paths.

## 5. Version 8 — Controlled Telemetry Generation

Activities:

1. Establish normal activity baselines.
2. Generate controlled authentication failures.
3. Examine Windows security event ingestion.
4. Investigate Linux authentication events.
5. Review Wazuh alert rules and severity levels.
6. Correlate source activity with generated alerts.
7. Identify gaps in visibility and detection.
8. Document investigation findings and lessons learned.

**Completion criteria:** Controlled activity is traceable from the originating system to collected telemetry and, where applicable, a corresponding Wazuh alert.

## 6. Engineering Principles

- Required connectivity rather than unrestricted connectivity.
- Least-privilege access and controlled network paths.
- Validate functionality before introducing complexity.
- Maintain separation between attack, client, server, and management roles.
- Document configuration decisions and troubleshooting.
- Preserve repeatable deployment and investigation procedures.

## 7. Future Integration

Phase 3 prepares the environment for later threat simulation, advanced security controls, and detection engineering.

Future work may include:

- Custom Wazuh detection rules.
- Sysmon telemetry.
- MITRE ATT&CK mapping.
- Firewall log correlation.
- File integrity monitoring.
- Alert tuning and investigation workflows.

## 8. Documentation

- `v7-wazuh-deployment.md`
- `v8-controlled-telemetry-generation.md`

Detailed implementation commands, configuration decisions, validation evidence, troubleshooting, and lessons learned will be maintained in the individual version documents.

---

**Phase Status:** Initial architecture and deployment planning. Configuration details will be updated as implementation is validated.
