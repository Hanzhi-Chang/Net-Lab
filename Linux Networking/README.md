# Linux Networking

**Practical Linux networking notes for configuration, verification, packet processing, and troubleshooting.**

This section connects day-to-day Linux network administration with the kernel mechanisms that move packets between a network interface and an application. It begins with operational commands and persistent configuration, then develops toward deeper packet-path analysis.

## Current Content

### [Linux Network Configuration and Management](./Network%20Configuration%20with%20NetworkManager.md)

Inspect interfaces, addresses, routes, sockets, and reachability with `ip`, `ss`, `ping`, and `tracepath`. The guide also covers NetworkManager devices and connection profiles, DHCP and static addressing, applying changes safely, and important differences among RHEL, Fedora, Ubuntu, and Debian.

### [How Does Data Travel from a Network Interface Card to the Protocol Stack?](./Kernel%20Packet%20Reception.md)

Follow the Linux receive path from a NIC and DMA receive ring through hardware interrupts, NAPI, `NET_RX_SOFTIRQ`, `sk_buff`, protocol registration, the IP and transport layers, and finally a user-space socket. An appendix introduces the C pointers, function pointers, and callback registration patterns used in the examples.

## Suggested Reading Order

1. Start with [Linux Network Configuration and Management](./Network%20Configuration%20with%20NetworkManager.md) to build confidence inspecting live network state and managing persistent settings.
2. Continue with [the kernel packet-reception guide](./Kernel%20Packet%20Reception.md) to understand what happens below those user-space tools when a packet arrives.

The command syntax, persistent configuration method, and kernel implementation can vary by Linux distribution, software release, driver, and hardware platform. Confirm the local interface names and network-management service before changing a system, and test disruptive changes through console access or another recovery path whenever possible.

## Planned Topics

Future material may cover network namespaces, bridges, VLAN interfaces, virtual Ethernet pairs, firewalling, packet capture, performance analysis, and fault-based troubleshooting labs.
