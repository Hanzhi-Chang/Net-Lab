# Cisco Network

**Practical documentation and reproducible labs for configuring, operating, verifying, and troubleshooting Cisco networks.**

This directory is the main Cisco-focused area of Net-Lab. It connects networking concepts with the tasks performed on real devices: navigating the operating system, building configurations, checking operational state, collecting evidence, diagnosing failures, and recovering safely from mistakes.

> [!NOTE]
> The study material behind this section has already been completed. It is being reorganized into concise documentation, command references, reusable configurations, and reproducible labs, which will be published progressively.

## Who This Section Is For

- Learners preparing for CCNA or progressing toward professional-level Cisco study.
- People who understand networking theory but have had limited access to Cisco devices.
- Junior engineers building confidence with the Cisco CLI and common operational workflows.
- Practitioners looking for focused command references, verification methods, and troubleshooting exercises.

This is not an exam-cram collection. The aim is to show what a command changes, how to prove that it worked, what can go wrong, and how to investigate the result.

## Main Content

The Cisco Network section is being organized around the following areas:

### [Cisco Internetwork Operating System (IOS)](./Cisco%20Internet%20Operation%20System/)

Device and operating-system fundamentals that support everyday administration: IOS and IOS XE architecture, filesystems, boot behaviour, software images, licensing, device access, configuration management, logging, monitoring, time synchronisation, discovery protocols, diagnostics, and management-plane protection.

### Network Fundamentals and Architecture

Network models, forwarding behaviour, addressing, device roles, campus and enterprise architecture, and the design principles required to understand later labs.

### Switching

Ethernet switching, VLANs, trunks, EtherChannel, spanning tree, Layer 2 protection, and the verification and troubleshooting of common campus switching problems.

### IP Routing

IPv4 and IPv6 forwarding, static routing, route selection, redistribution, and dynamic routing with RIP, EIGRP, OSPF, IS-IS, and BGP.

### Network Services and System Management

Services and operational functions such as DHCP, DNS-related behaviour, NAT, NTP, SNMP, syslog, NetFlow, device discovery, configuration backup, image management, and routine health checks.

### WAN, MPLS, and VPN

WAN technologies, provider and enterprise connectivity, MPLS concepts, tunnelling, and VPN technologies, supported by topology-based configuration and verification.

### [Multicast](./Multicast/)

Multicast addressing, receiver membership with IGMP, Layer 2 multicast forwarding, Reverse Path Forwarding, PIM operation, Rendezvous Point design and discovery, multicast policy, and troubleshooting.

### Quality of Service

Traffic classification, marking, queuing, policing, shaping, congestion management, and evidence-based validation.

### Security

Device hardening, management access, AAA, access control, control-plane protection, infrastructure security, VPN security, and Cisco ASA topics.

### Wireless

Cisco wireless architecture, controller and access-point concepts, WLAN configuration, client connectivity, security, monitoring, and troubleshooting.

### Data-Centre Networking

Cisco Nexus and NX-OS operations, data-centre switching, virtualisation-related network features, resilient designs, and platform-specific troubleshooting.

### Software-Defined Networking and Automation

Controller-based networking, APIs, Python, Ansible, NETCONF, RESTCONF, YANG, telemetry, repeatable configuration, and automated validation.

### Tools and Lab Platforms

Supporting material for EVE-NG, virtual machines, packet capture, network utilities, configuration comparison, and other tools used to build and investigate the labs.

## Lab Standard

Where applicable, a lab should include:

1. Objectives and prerequisites.
2. Platform, software version, and feature assumptions.
3. Topology and addressing information.
4. Initial state and required configuration.
5. Complete configuration steps with explanations.
6. Verification commands, expected behaviour, and test results.
7. Troubleshooting evidence, root cause, correction, and regression checks.
8. Limitations, cleanup steps, and lessons learned.

Commands and feature behaviour can differ across IOS, IOS XE, NX-OS, hardware platforms, and software releases. Each guide should therefore identify the environment in which it was tested and call out known differences when relevant.

## Suggested Starting Point

Begin with [Cisco Internetwork Operating System (IOS)](./Cisco%20Internet%20Operation%20System/) if you have limited hands-on experience with Cisco equipment. It establishes the operational foundation needed before moving into larger switching, routing, security, and automation labs.

## Disclaimer

This material is intended for education and lab use. Review and adapt every configuration before using it in a production environment. Cisco product names and trademarks belong to their respective owners.
