# Cisco Network

**Practical documentation and reproducible labs for configuring, operating, verifying, and troubleshooting Cisco networks.**

This directory is the Cisco-focused part of Net-Lab. It currently contains CLI and IOS foundations, a complete baseline campus-network lab, focused IP-routing guides, and an evidence-based IGMP lab within the IP-multicast section. Additional material will be added progressively as it is reorganised and validated.

The goal is not to collect commands in isolation. Each technical guide explains what a feature does, how it affects forwarding, how to verify the operational state, and which limitations or failure cases matter in a lab or production-style workflow.

## Current Content

### Start Here

- [Warmup and Conventions](./WARMUP.md) introduces the Cisco CLI, operating modes, command syntax, context-sensitive help, command completion, and output filtering. Read this first if you are new to Cisco devices or to the conventions used in this repository.
- [Cisco IOS System Management](./Cisco%20Internet%20Operation%20System/) is the index and roadmap for IOS and IOS XE operational topics.
- [Cisco IOS Basics](./Cisco%20Internet%20Operation%20System/Basics/docs.md) is the currently published system-management module. It covers IOS and IOS XE architecture, filesystems, the configuration register, boot behaviour, image upgrades, licensing, and basic router host services.

### Integrated Lab

- [Small Campus Network: Baseline Design and Configuration](./Campus%20Network/) builds a layered network from access switches to a Cisco ASA and two simulated ISP paths. It includes the topology, addressing plan, reusable configurations, verification evidence, known limitations, and a roadmap for later improvements.

  The current baseline demonstrates VLAN segmentation, dual-homed access switches, Rapid PVST+, Port Security, HSRP gateway load sharing, DHCP, inter-VLAN routing, OSPF, ASA security zones, source-specific dynamic NAT, and a verified VLAN 10 path to a simulated Internet destination.

### IP Routing

- [IPv4 Directed Broadcast on Cisco IOS](./IP%20Routing/Directed%20Broadcast.md) explains last-hop broadcast conversion, the secure default behaviour, optional ACL control, packet-capture verification, cleanup, and regression testing.
- [OSPF Default Route Blackholing and Failover](./IP%20Routing/OSPF%20Default%20Route%20Failover.md) reproduces a stale Type-5 default advertisement, traces the resulting black hole, and validates conditional default origination during an ISP-1 failure and recovery cycle.
- [NAT Destination-Port Redirection from Inside to Outside](./IP%20Routing/NAT%20Port%20Redirection.md) reproduces an ISP-blocked application port and shows how inside-source PAT and outside-source static PAT translate different endpoints of the same flow.
- [Legacy NAT Between Overlapping Networks](./IP%20Routing/NAT%20Between%20Overlapping%20Networks.md) reproduces bidirectional static NAT across overlapping address spaces and diagnoses routing order, a platform-created NAT alias, failed PBR, and Proxy ARP behaviour.
- [DHCP Snooping and Dynamic ARP Inspection](./IP%20Routing/DAI.md) is an EVE-NG lab that validates Option 82 handling, DHCP Snooping bindings, trusted and untrusted ports, and DAI drop counters.
- [Routing Policy Matching on Cisco IOS](./IP%20Routing/Routing%20Policies.md) compares ACLs, wildcard masks, prefix lists, route maps, BGP AS-path matching, offset lists, and Policy-Based Routing according to the object being matched and the feature consuming the result.

### IP Multicast

- [IP Multicast](./Multicast/) provides the multicast roadmap and published material for receiver membership, Layer 2 forwarding, multicast routing, PIM, Rendezvous Point design, and policy controls.
- [IGMPv1, IGMPv2, and IGMPv3 on Cisco IOSv](./Multicast/IGMP.md) is an EVE-NG lab with captured evidence for querier election, IGMP version behaviour, static membership, group filtering, SSM source selection, and basic IGMP Snooping.

## Repository Map

| Path | Current purpose |
| --- | --- |
| [`WARMUP.md`](./WARMUP.md) | CLI introduction and repository command conventions |
| [`Cisco Internet Operation System/`](./Cisco%20Internet%20Operation%20System/) | IOS and IOS XE system-management index |
| [`Cisco Internet Operation System/Basics/docs.md`](./Cisco%20Internet%20Operation%20System/Basics/docs.md) | Published IOS fundamentals module |
| [`Campus Network/`](./Campus%20Network/) | Baseline campus lab, evidence, and publication configurations |
| [`IP Routing/Directed Broadcast.md`](./IP%20Routing/Directed%20Broadcast.md) | Directed-broadcast concept and packet-capture lab |
| [`IP Routing/OSPF Default Route Failover.md`](./IP%20Routing/OSPF%20Default%20Route%20Failover.md) | OSPF default-origination failure, correction, failover, and recovery lab |
| [`IP Routing/NAT Port Redirection.md`](./IP%20Routing/NAT%20Port%20Redirection.md) | Destination-port translation with simultaneous source PAT |
| [`IP Routing/NAT Between Overlapping Networks.md`](./IP%20Routing/NAT%20Between%20Overlapping%20Networks.md) | Bidirectional static NAT, order of operations, NAT alias, PBR, and Proxy ARP troubleshooting |
| [`IP Routing/DAI.md`](./IP%20Routing/DAI.md) | DHCP Snooping, Option 82, binding verification, and DAI lab |
| [`IP Routing/Routing Policies.md`](./IP%20Routing/Routing%20Policies.md) | Routing-policy matching reference |
| [`Multicast/`](./Multicast/) | Multicast roadmap and published IGMP lab |
| [`Multicast/IGMP.md`](./Multicast/IGMP.md) | Evidence-based IGMPv1, IGMPv2, IGMPv3, SSM, filtering, and Snooping lab |

## Suggested Reading Order

1. Start with [Warmup and Conventions](./WARMUP.md) to understand the CLI notation used throughout the repository.
2. Continue with [Cisco IOS Basics](./Cisco%20Internet%20Operation%20System/Basics/docs.md) for device, filesystem, boot, image, and licensing fundamentals.
3. Work through the [Small Campus Network](./Campus%20Network/) lab to connect Layer 2 switching, first-hop redundancy, routing, DHCP, firewalling, and NAT in one topology.
4. Use the [IP Routing](./IP%20Routing/) guides for focused forwarding, DHCP Snooping, DAI, and policy study.
5. Read the [IP Multicast](./Multicast/) roadmap, then complete the [IGMPv1, IGMPv2, and IGMPv3](./Multicast/IGMP.md) lab.

## Documentation and Lab Standard

Where applicable, material in this section aims to include:

1. Scope, objectives, environment, and feature assumptions.
2. Topology, addressing, and traffic-flow context.
3. Configuration examples with an explanation of their purpose.
4. Verification commands and evidence of the observed state.
5. Troubleshooting considerations, limitations, cleanup, and regression checks.
6. Sanitised publication configurations without credentials, licensed software images, or device-specific secrets.

Commands and feature behaviour can differ across IOS, IOS XE, ASA, NX-OS, hardware platforms, virtual images, and software releases. Each guide should therefore identify its tested environment and avoid presenting an educational topology as a production reference design.

## Planned Expansion

The following areas are planned but should not be treated as currently published modules:

- Switching, EtherChannel, and spanning-tree labs.
- Additional routing protocols, redistribution, path selection, and failure testing.
- Network services, management-plane security, AAA, monitoring, and telemetry.
- WAN, MPLS, VPN, QoS, wireless, and data-centre networking.
- Python, Ansible, NETCONF, RESTCONF, YANG, and automated validation.
- Additional multicast-routing, Rendezvous Point, failure-injection, and troubleshooting labs.
- Incremental security, resilience, observability, and automation improvements built on the baseline Campus Network lab.

## Disclaimer

This material is intended for education and isolated lab use. Review addressing, security policy, licensing, platform support, and release-specific syntax before adapting any configuration to another environment. Cisco product names and trademarks belong to their respective owners.
