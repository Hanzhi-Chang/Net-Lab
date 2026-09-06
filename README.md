# Net-Lab


**A practical, evolving collection of network engineering notes, labs, configurations, and troubleshooting exercises.**

Net-Lab is a learning-focused repository for exploring how networks are designed, configured, verified, and troubleshot. It connects networking theory with reproducible lab work across Cisco technologies, Cumulus Linux, and Linux networking.

> [!NOTE]
> The study material behind this repository has already been completed. It is now being reorganized and transformed into clear, reproducible labs for publication on GitHub, with content released progressively as this process is completed.

## Why I Created This Project

I created Net-Lab with two main goals:

- **Learn in public:** Organize what I have learned, turn theory into practical experiments, and improve my understanding through clear technical documentation.
- **Share and collaborate:** Provide useful material for people who are learning networking or preparing to enter the field, while exchanging ideas and feedback with network engineers and learners around the world.

Writing a configuration is only one part of network engineering. For that reason, this project also focuses on explaining design choices, verifying expected behaviour, investigating failures, and recording lessons learned.

## Project Scope

### Cisco Networking

This is currently the main area of the repository. Planned topics include:

- Network fundamentals and architecture
- Layer 2 switching, VLANs, trunking, EtherChannel, and spanning tree
- IPv4 and IPv6 routing
- RIP, EIGRP, OSPF, IS-IS, and BGP
- MPLS, VPNs, and WAN technologies
- Multicast and Quality of Service
- Network security and Cisco ASA
- Wireless networking
- Network services and system management
- Cisco Nexus and data-centre networking
- Software-defined networking and network automation
- Python, Ansible, NETCONF, RESTCONF, YANG, and related tools

### Cumulus Linux

This section will document my journey into open networking with Cumulus Linux. It will begin with fundamental system and interface operations, then develop toward switching, routing, configuration management, and troubleshooting.

### [Linux Networking](./Linux%20Networking/)

This section will explore the Linux networking knowledge and tools that are useful to network engineers, including:

- Interface, address, route, and neighbour management
- Network namespaces, bridges, VLANs, and virtual Ethernet interfaces
- Connectivity and path-testing tools
- Packet capture and traffic analysis
- DNS, sockets, ports, and common network services
- Firewalling and packet filtering
- Performance monitoring and troubleshooting

Available material currently covers:

- [Linux Network Configuration and Management](./Linux%20Networking/Network%20Configuration%20with%20NetworkManager.md) — interface naming, addresses, routes, sockets, connectivity tests, and persistent configuration with NetworkManager.
- [How Data Travels from a NIC to the Linux Protocol Stack](./Linux%20Networking/Kernel%20Packet%20Reception.md) — DMA, receive rings, interrupts, NAPI, softirqs, socket buffers, protocol registration, and delivery to applications.

## What Each Lab Will Include

Where applicable, each lab will contain:

1. **Objectives** — what the lab is intended to demonstrate.
2. **Prerequisites** — the required concepts, software, and platform.
3. **Topology** — a diagram and an addressing plan.
4. **Device configurations** — complete configurations or clearly documented configuration changes.
5. **Concept explanation** — the important ideas behind the implementation.
6. **Verification** — commands, expected output, and connectivity tests.
7. **Troubleshooting** — common issues or an intentionally introduced fault, supported by evidence and analysis.
8. **Lessons learned** — conclusions, limitations, and possible improvements.

The aim is to make the labs understandable, repeatable, and useful beyond a single successful configuration.

## Repository Structure

The repository currently follows this high-level structure:

```text
Net-Lab/
├── Cisco Network/
├── Linux Networking/
│   ├── README.md
│   ├── Kernel Packet Reception.md
│   └── Network Configuration with NetworkManager.md
├── assets/
│   ├── My-tools.md
│   └── Network-Tools.md
├── LICENSE
└── README.md
```

Individual labs may include their own README, topology files, configurations, scripts, test results, and troubleshooting notes.

## Tools and Workflow

See [My Tools](./assets/My-tools.md) for the applications I use to design topologies, build virtual labs, access device consoles, analyse packets, and write the documentation in this repository. [Common Network Tools](./assets/Network-Tools.md) provides a practical command reference for connectivity testing, route inspection, packet analysis, performance measurement, file-transfer labs, and authorised network discovery.

## About the Author

I am Hanzhi Chang, a CCIE Enterprise Infrastructure-certified network engineer currently pursuing an MSc in Computer Science at University College Dublin. My background also includes a Master's degree in Cyberspace Security and hands-on experience in an Internet service provider environment, where I supported enterprise network deployment, operations, migration, and troubleshooting.

Alongside routing and switching, I work with Python, Ansible, Linux, and network automation, and I hold RHCE, CKA, and NVIDIA-Certified Professional AI Networking (NCP-AIN) certifications. My AI networking background extends from traditional data-centre networking into infrastructure for high-performance AI workloads, including NVIDIA Spectrum-X and InfiniBand fabrics, RoCE, congestion management, telemetry, Kubernetes integration, troubleshooting, and automation.

My interests lie at the intersection of enterprise networking, Linux, cloud-native systems, and AI infrastructure. Through Net-Lab, I turn technical knowledge and practical experience into structured, reproducible labs that others can study, test, and improve.

## Disclaimer

This repository is created for education and lab experimentation. Configurations should be reviewed and adapted before being used in a production environment. Product names and trademarks belong to their respective owners.

## License

This project is licensed under the [MIT License](LICENSE).
