# Cisco IOS System Management

This directory covers the complete system-management scope for Cisco IOS and IOS XE devices. It explains how a device operates and is administered before routing, switching, and network services are added: how the system boots, where it stores files and configurations, how engineers gain and control access, how changes are recorded, how operational data is collected, and how faults are investigated.

> [!IMPORTANT]
> Many people preparing for CCNA, people without professional network experience, and people who have not worked directly with Cisco equipment may not realise how important these foundations are. Some of the topics receive little attention in an exam blueprint, but they are encountered in day-to-day network operations. It is possible to study advanced technologies such as OSPF, BGP, and multicast while still being unable to manage software images, recover device access, interpret logs, verify time, or collect troubleshooting evidence. That gap can make it difficult to demonstrate practical readiness and can become a barrier to obtaining a first network role.

## Main Content

### [Cisco IOS Basics](./Basics/)

- Classic IOS and IOS XE architecture and software packaging.
- Cisco filesystems, storage, configuration-register behaviour, and the boot process.
- ROMMON, software-image transfer and upgrade, image verification, boot selection, licensing, and basic host or service testing.

### Device Access and Security

- Console and remote CLI access.
- User EXEC and privileged EXEC modes and their protection.
- Local users, passwords, privilege levels, and access-control methods.
- Password recovery for Cisco routers and switches.
- Telnet and SSH server and client operation.

### Authentication, Authorisation, and Accounting

- The purpose and operation of AAA.
- Local authentication on Cisco IOS routers and switches.
- Authentication method lists and fallback behaviour.
- Local command authorisation and IOS privilege levels.
- TACACS+ integration with a Linux CentOS server.

### Configuration Management

- Copying or merging configuration into the running configuration.
- Saving, backing up, and restoring configurations.
- Configuration archives, snapshots, comparison, and rollback.
- Configuration-change notification and command logging.

### Logging and Monitoring

- Cisco IOS syslog messages, severity levels, timestamps, and logging destinations.
- Simple Network Management Protocol concepts and operation.
- NetFlow traffic visibility and flow records.
- Interface optical-power monitoring and transceiver diagnostics.

### Network Time

- Cisco Network Time Protocol configuration and operation.
- NTP unicast, multicast, and broadcast behaviour.
- NTP authentication, NTPv4, and troubleshooting.
- Precision Time Protocol concepts and forwarding modes.

### Device Discovery Protocols

- Cisco Discovery Protocol operation and neighbour information.
- Link Layer Discovery Protocol operation and interoperability.

### Troubleshooting and Diagnostics

- A structured troubleshooting process and evidence collection.
- Ping and traceroute behaviour on Cisco IOS.
- Conditional debug for targeted diagnostics.
- MTU and Path MTU troubleshooting.
- Cisco Embedded Packet Capture.
- ERSPAN configuration on Cisco IOS XE.

### Performance and Automation

- Cisco IOS Embedded Event Manager policies and event-driven actions.
- Combining IP SLA and EEM for monitoring and automated response.

### Security and Control-Plane Protection

- Control Plane Policing to protect device control-plane resources.
- Management Plane Protection to restrict where management traffic is accepted.

## Relationship to the Basics Module

The [Basics](./Basics/) directory covers only the first section of this system-management material. The remaining sections extend from initial device operation into secure access, change management, observability, troubleshooting, automation, and infrastructure protection.

Commands and behaviour can differ between classic IOS, IOS XE, device families, and software releases. Each detailed document or lab should identify the platform and version used for validation.
