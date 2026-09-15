# Phase 2 — Network Architecture

## Living Enterprise Environment (LEE)

Phase 2 of the Living Enterprise Environment focuses on transforming the lab from a collection of connected enterprise systems into a deliberately designed **enterprise network architecture**.

Phase 1 established the systems that make up the environment:

- Linux infrastructure
- Windows Server
- Active Directory Domain Services
- DNS
- Kerberos authentication
- Organizational Units
- Users and security groups
- Privileged administrative accounts
- Windows client integration
- Enterprise resource access
- Authentication investigation
- Windows event analysis
- Indicator of Compromise (IOC) investigation

Those systems currently communicate across a relatively simple VMware virtual network.

Phase 2 introduces the network architecture that will eventually allow the environment to support **segmentation, traffic inspection, centralized monitoring, attack simulation, and detection engineering**.

The objective is not simply to make the network more complicated.

The objective is to make the network **intentional**.

---

# Phase 2 Objective

The primary goal of Phase 2 is to design and implement a network architecture that more closely represents how enterprise systems are separated, controlled, monitored, and protected.

The existing environment has already demonstrated that the individual systems function correctly.

Phase 2 asks a different set of questions:

- Which systems should be able to communicate?
- Which systems should be separated?
- What network services are required for Active Directory?
- Where should security controls eventually be placed?
- What traffic should be observable?
- How can future attacker activity be isolated from the rest of the lab?
- How can network activity later be correlated with endpoint and authentication telemetry?

These questions establish the foundation for the security monitoring and attack simulation phases that follow.

---

# Why Phase 2 Exists

A flat network is useful during initial infrastructure deployment because it reduces complexity.

It is not ideal for demonstrating enterprise security architecture.

If every system exists on the same network, there are fewer opportunities to observe or control:

- lateral movement
- network segmentation
- firewall policy
- inter-network communication
- trust boundaries
- administrative traffic
- attacker traffic
- security monitoring points

Phase 2 begins addressing these limitations.

The network becomes part of the security architecture rather than simply the mechanism connecting the virtual machines.

---

# Starting Architecture

At the beginning of Phase 2, the Living Enterprise Environment contains the following primary systems:

### LEE-WIN-SRV-01

Windows Server and Active Directory Domain Controller.

Primary services include:

- Active Directory Domain Services
- DNS
- Kerberos / KDC
- LDAP
- Group Policy
- SYSVOL
- SMB resources

Current address:

`192.168.133.130`

---

### LEE-WIN-CLT-01

Windows 11 enterprise workstation used for:

- domain authentication
- user activity
- administrative testing
- resource access
- Windows telemetry generation
- future attack simulation

Current address:

`192.168.133.131`

---

### LEE-Ubuntu-01

Linux server used for:

- Linux administration
- SSH authentication
- system logging
- Linux telemetry
- future monitoring services and attack simulation

---

### Kali Linux

Attack and security testing workstation.

Kali will eventually be used to generate controlled malicious or suspicious activity against systems inside the Living Enterprise Environment.

Examples may include:

- reconnaissance
- authentication attacks
- network scanning
- service enumeration
- credential attacks
- lateral movement testing
- command-and-control simulations
- other controlled adversary behavior

All attack activity will remain isolated within the lab environment.

---

# Existing Network

The environment currently relies primarily on VMware virtual networking.

The Windows systems have been operating on the:

`192.168.133.0/24`

network.

The environment has already demonstrated successful communication between systems, including:

- ICMP connectivity
- DNS resolution
- Active Directory discovery
- LDAP SRV record resolution
- Kerberos authentication
- domain communication
- SMB resource access

This validates the existing environment before introducing additional network complexity.

That baseline is important.

If connectivity problems occur after the network redesign, Phase 1 provides a known-good reference point for troubleshooting.

---

# Network Architecture Goals

Phase 2 will introduce several core networking concepts into the Living Enterprise Environment.

## 1. Network Documentation

Before changing the architecture, the existing network will be documented.

This includes identifying:

- virtual machines
- network interfaces
- IP addresses
- VMware virtual networks
- default gateways
- DNS configuration
- DHCP configuration
- critical services
- communication dependencies

This creates a baseline of the environment before segmentation begins.

---

## 2. Network Diagram

A formal network diagram will be created to document the architecture.

The diagram will evolve as the environment evolves.

It should identify:

- Windows Server
- Windows client
- Linux server
- Kali attacker system
- VMware virtual networks
- subnets
- gateways
- security boundaries
- firewall/router placement
- monitoring infrastructure

The diagram becomes both operational documentation and part of the LEE portfolio.

---

# 3. IP Addressing Strategy

The existing lab initially relied heavily on VMware DHCP.

Phase 2 will establish a more intentional addressing strategy.

Infrastructure systems that provide critical services should have predictable addresses.

Examples include:

- Domain Controllers
- DNS servers
- firewall interfaces
- monitoring servers
- SIEM infrastructure
- security appliances

Client systems may continue using DHCP where appropriate.

The objective is to make network addressing predictable enough to support firewall rules, monitoring, troubleshooting, and documentation.

---

# 4. Network Segmentation

The environment will begin transitioning away from a completely flat architecture.

Logical security zones will be introduced.

Potential zones include:

### Server Network

Contains infrastructure systems such as:

- Active Directory
- DNS
- file services
- future enterprise servers

### Client Network

Contains enterprise workstations and user systems.

### Security / Monitoring Network

Reserved for systems responsible for:

- SIEM
- log collection
- monitoring
- security analysis

### Attack Network

Contains systems such as Kali that generate controlled adversary activity.

The exact architecture may evolve during implementation.

The important design principle is that systems should be separated according to **function, trust, and security requirements**.

---

# 5. pfSense Integration

A future pfSense firewall/router will become an important component of the LEE network architecture.

Instead of relying entirely on VMware's virtual networking to connect systems, pfSense will eventually provide controlled communication between network segments.

This will allow the lab to demonstrate concepts such as:

- routing
- firewall policy
- network segmentation
- inter-network access control
- security zones
- traffic logging
- blocked connections
- permitted connections

The firewall will therefore become both a **network device** and a **security telemetry source**.

---

# 6. Firewall Policy

Once segmentation exists, communication between networks should occur intentionally.

Firewall rules will eventually answer questions such as:

> Should a workstation be allowed to communicate with the Domain Controller?

Yes—but only using the services necessary for domain operations.

Examples may include:

- DNS
- Kerberos
- LDAP
- SMB
- RPC
- Global Catalog services

Other traffic may be restricted.

This creates an opportunity to understand not only how firewalls work, but also **what enterprise applications actually require from the network**.

---

# 7. Active Directory Dependency Mapping

Active Directory is heavily dependent on network communication.

Changing the network architecture therefore creates an opportunity to study the protocols required for Windows domain operations.

Examples include:

| Service | Common Port |
|---|---:|
| DNS | TCP/UDP 53 |
| Kerberos | TCP/UDP 88 |
| RPC Endpoint Mapper | TCP 135 |
| LDAP | TCP/UDP 389 |
| SMB | TCP 445 |
| Kerberos Password Change | TCP/UDP 464 |
| LDAPS | TCP 636 |
| Global Catalog | TCP 3268 |
| Global Catalog SSL | TCP 3269 |

Dynamic RPC communication must also be considered.

Rather than simply allowing unrestricted communication between networks, the lab will gradually identify the traffic necessary for enterprise services to operate.

This transforms networking problems into learning opportunities.

---

# 8. Traffic Visibility

One of the most important objectives of Phase 2 is increasing visibility into network activity.

Later phases of the Living Enterprise Environment will analyze activity from several perspectives.

### Endpoint Perspective

What happened on the system?

Examples:

- process creation
- authentication attempts
- account changes
- PowerShell execution

### Identity Perspective

What happened within Active Directory?

Examples:

- Kerberos authentication
- account lockouts
- group membership changes
- privileged account activity

### Network Perspective

What communication occurred between systems?

Examples:

- connection attempts
- scanning
- unusual ports
- denied traffic
- lateral movement
- attacker-to-victim communication

These perspectives will eventually be correlated during security investigations.

---

# 9. Attacker Isolation

Kali currently serves as the primary offensive security platform in the lab.

As attack simulations become more advanced, isolating attacker infrastructure becomes increasingly important.

The attacker system should eventually exist within a controlled network where communication can be intentionally permitted or denied.

This provides several benefits:

- prevents accidental interaction with unrelated systems
- creates a clear attacker trust boundary
- allows firewall logs to capture attacker activity
- makes network reconnaissance easier to identify
- allows controlled lateral movement scenarios
- creates cleaner telemetry for future investigations

The attacker network becomes part of the experiment rather than simply another machine on the LAN.

---

# 10. Establishing Normal Network Behavior

Before generating malicious traffic, normal behavior must be understood.

Phase 2 will therefore establish network baselines.

Examples may include:

- workstation DNS queries
- Kerberos authentication traffic
- LDAP communication
- SMB access
- normal ICMP traffic
- DHCP behavior
- domain controller communication
- Windows client-to-server communication

Understanding normal traffic makes abnormal activity easier to recognize later.

This follows the same methodology used earlier in the Living Enterprise Environment:

**Establish the baseline before generating the anomaly.**

---

# Proposed Architecture Direction

The exact architecture will be finalized during implementation, but the environment is moving toward a model similar to:

```text
                         Internet
                            |
                       VMware NAT
                            |
                         pfSense
                            |
        +-------------------+-------------------+
        |                   |                   |
        |                   |                   |
   SERVER NETWORK      CLIENT NETWORK      ATTACK NETWORK
        |                   |                   |
        |                   |                   |
 LEE-WIN-SRV-01       LEE-WIN-CLT-01          Kali
    AD / DNS
        |
        |
   Linux Services


                SECURITY / MONITORING NETWORK
                            |
                            |
                     SIEM / Monitoring
```

This architecture is intentionally simplified.

The goal is not to reproduce
