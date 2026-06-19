## Operating System Selection

The Windows server for Phase 3 was deployed using:

- Windows Server 2025 Standard Evaluation
- Desktop Experience (GUI)

Desktop Experience was selected to facilitate:
- Active Directory administration
- Event Viewer analysis
- DNS management
- Documentation and screenshots
- Educational purposes within the Living Enterprise Environment

## Network Baseline Validation

Following remediation of a VMware virtual networking issue, network connectivity was successfully restored and validated between the Linux and Windows systems.

### LEE-WIN-SRV-01

* Hostname: LEE-WIN-SRV-01
* IPv4 Address: 192.168.133.130
* Subnet Mask: 255.255.255.0
* Default Gateway: 192.168.133.2
* DHCP Server: 192.168.133.254
* DNS Server: 192.168.133.2

### LEE-Ubuntu-01

* Hostname: LEE-Ubuntu-01
* Interface: ens33
* IPv4 Address: 192.168.133.129
* Default Gateway: 192.168.133.2

### Validation Results

* DHCP leases successfully obtained on both systems.
* Default gateways assigned correctly.
* Network interfaces reported operational status.
* Both systems placed on the same 192.168.133.0/24 subnet.
* Environment validated and ready for Active Directory deployment.

