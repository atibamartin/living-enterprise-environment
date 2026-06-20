# living-enterprise-environment
Living Enterprise Environment — A continuously evolving cloud, cybersecurity, virtualization, and automation lab environment focused on building real-world enterprise skills through hands-on projects, infrastructure design, documentation, security analysis, and DevSecOps workflows. This repository will document the development of labs, scripts, architectures, troubleshooting processes, and operational knowledge across VMware, Linux, networking, cloud platforms, SIEM/SOAR tooling, Terraform, automation, and defensive security practices.

# Living Enterprise Environment (LEE)

A continuously evolving cybersecurity,
cloud, virtualization, automation,
and detection engineering lab.

---

## Environment Overview

### Core Objectives

- Linux Administration
- Security Operations
- Threat Detection
- SIEM Engineering
- Cloud Security
- Automation
- Incident Response

---

## Current Environment

| System | Purpose |
|----------|----------|
| Ubuntu Server | Logging, Linux Administration |
| Kali Linux | Attack Simulation |
| Windows Server | Authentication Analysis |
| Wazuh | SIEM |
| pfSense | Segmentation |
| Security Onion | Detection Engineering |

---

# Project Roadmap

# Phase 1 - OS Foundations

###  V1 – Linux Logging ✅

Status: Completed

- auth.log
- syslog
- kern.log

Documentation:

- v1-linux-auth-logging.md
- v1-linux-syslog-logging.md
- v1-linux-kernel-logging.md


### V2 – Authentication Analysis ✅

Status: Completed

Focus Areas:

- SSH Authentication
- Failed Logins
- Brute Force Detection
- Account Lockouts
- Privilege Escalation

Documentation:

- v2-authentication-baseline.md
- v2-linux-authentication-baseline.md
- v2-sudo-analysis.md


### V3 – Windows Integration 🚧

Status: In Progress

Focus Areas:

- Windows Event Viewer
- Security Logs
- System Logs
- Application Logs
- Privilege Escalation

Documentation:

- v3-windows-server-build.md
- v3-linux-syslog-logging.md
- v1-linux-kernel-logging.md

---

# Phase 2 - Network Architecture 🚧

### Objective

Design and Build out a network to support an enterprise.

### V4 – Enterprise Network Design 🚧

Focus:

- Current LEE network
- Future-state architecture
- IP addressing plan
- Network roles

Skills:

- Network documentation
- Security architecture
- Systems planning

Documentation:

LEE Network Design Document

### V5 – Network Segmentation Planning 🚧

Focus:

- Server network
- User network
- Security network
- Management network

Skills:

- Segmentation concepts
- Trust boundaries
- Traffic flow analysis

Deliverable:
Network Segmentiation Plan

---

# Phase 3 - Security Monitoring Foundations 🚧

Status:  Planned

### Objective

Create centralized visibility across the environment.

### V6 - Wazuh Deployment

Focus:

- Wazuh Manager
- Linux Agent
- Windows Agent

Skills:

- SIEM fundamentals
- Agent deployment
- Log collection

Deliverable:

Centralized logging environment

### V7 - Controlled Telemetry Generation

Generate known events:

Examples:

- Failed SSH logins
- Failed Windows logins
- File modifications
- User creation
- Service changes

Skills:

- Detection validation
- Log correlation

Deliverable:

Known Event Catalog

---

# Phase 4 - Threat Simulation 🚧

Status:  Planned

### Objective

Create attack telemetry.

### V8 - Linux Attack Simulation

Examples:

- Brute force attempts
- Privilege escalation
- Suspicious commands
- Persistence mechanisms

Skills:

- Threat analysis
- Log investigation

Deliverable:

Attack Detection Report


### V9 - Windows Attack Simulation

Examples:

- Failed logins
- PowerShell activity
- User creation
- Scheduled task persistence

Skills:

- Windows threat detection

Deliverable:

Windows Threat Detection Report


# Phase 5 – Network Security 🚧

Status:  Planned

### Objective

Understand enterprise network controls.

### V10 - pfSense Deployment

Focus:

- Firewall rules
- VLANs
- Network segmentation

Skills:

- Enterprise networking
- Security architecture

Deliverable:

Network Architecture Diagram


### V11 - Segmented Environment

Networks:

- User Network
- Server Network
- Management Network
- Security Network

Skills:

- Segmentation
- Traffic control

Deliverable:

LEE Network Design

---

# Phase 6 – SOC Operations 🚧

Status:  Planned


### Objective

Operate like a SOC analyst.


### V12 - Detection Tuning

Focus:

- Reduce false positives
- Improve alert quality


### V13 - Threat Hunting Exercises

Hunt for:

- Failed logins
- New users
- Privilege escalation
- Persistence
- Malware indicators

Skills:

- Threat hunting
- Investigation

Deliverable:

Threat Hunt Reports

### V14 - Incident Response Exercises

Scenarios:

- Account compromise
- Malware infection
- Insider activity
- Unauthorized access

Skills:

- Incident response
- Containment
- Documentation

Deliverable:

Incident Response Reports

---

# Phase 7 – Enterprise Security engineer 🚧

Status:  Planned

### Objective

Design security controls rather than simply monitor them.


### V15 - Vulnerability Management

Tools:

- OpenVAS
- Nessus Essentials

Skills:

- Vulnerability assessment
- Risk prioritization

Deliverable:

Vulnerability Assessment Report


### V16 - Security Hardening

Linux:

- SSH
- AppArmor
- Auditd

Windows:

- Defender
- Local Policy
- Event Auditing

Deliverable:

Hardening Checklist

---

### V17 - Final Enterprise Assessment

Objective:

Evaluate the entire Living Enterprise Environment as if it were a production enterprise.

Deliverables:

- Asset Inventory
- Network Diagram
- Security Architecture
- Baseline Documentation
- Detection Library
- Incident Response Procedures
- Lessons Learned

Outcome:

A complete portfolio demonstrating enterprise operations, security monitoring, threat detection, and incident response capabilities.

