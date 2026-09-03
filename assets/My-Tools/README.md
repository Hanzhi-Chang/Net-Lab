# My Tools

**The tools I use to design, build, operate, analyse, and document the network labs in this repository.**

The value of a tool is not just the task it performs, but the evidence it helps produce. My workflow combines topology design, network emulation, device access, packet analysis, and documentation so that each lab can be repeated and its results can be verified.

## Toolchain at a Glance

| Stage | Tool | How I Use It |
| --- | --- | --- |
| Design | Microsoft Visio | Create structured, presentation-ready network topology diagrams. |
| Sketch | Excalidraw | Explore ideas and produce quick explanatory diagrams during planning and troubleshooting. |
| Build | EVE-NG | Emulate multi-vendor network topologies and test configurations in an isolated lab environment. |
| Operate | SecureCRT | Access device consoles and remote CLI sessions, organise connections, and record command output. |
| Analyse | Wireshark | Capture and inspect packets to validate protocol behaviour and investigate faults. |
| Document | Typora | Write and preview the Markdown documentation published in this repository. |

## EVE-NG — Network Emulation

EVE-NG is the main platform I use to build virtual network labs. It allows me to connect routers, switches, firewalls, servers, and Linux hosts in one topology without requiring a separate physical device for every role.

I use EVE-NG to:

- Reproduce switching, routing, security, multicast, and automation scenarios.
- Build multi-device topologies that can be reset and tested repeatedly.
- Introduce controlled faults and compare the network state before and after a change.
- Connect virtual devices to external tools for terminal access, packet capture, and automation.
- Record the platform, node types, software versions, links, and resource assumptions required by a lab.

Vendor images and licences are not distributed in this repository. Anyone reproducing a lab must obtain and use software images in accordance with the relevant vendor licence.

## SecureCRT — Terminal Access

SecureCRT is my primary terminal client for working with network-device command-line interfaces. Its session management and tabbed interface are especially useful when a topology contains many devices.

I use SecureCRT to:

- Connect through SSH, serial console, or lab-only Telnet sessions where required by the emulated platform.
- Group and label sessions so that device access follows the topology naming scheme.
- Work with multiple device consoles side by side.
- Log command output for configuration review, verification, and troubleshooting evidence.
- Reuse consistent terminal, authentication, and logging settings across labs.

SSH is preferred whenever the platform supports it. Telnet is limited to isolated lab or console-access scenarios and is not recommended for production management traffic.

## Microsoft Visio — Formal Network Diagrams

Microsoft Visio is the main tool I use for polished topology diagrams. It is suitable for diagrams that need consistent alignment, labels, connectors, device roles, network boundaries, and addressing information.

A final lab diagram should make the following details easy to understand:

- Device names and functional roles.
- Interface-to-interface connections.
- VLANs, subnets, autonomous systems, or routing areas where relevant.
- Management, control-plane, and data-plane boundaries when they matter to the scenario.
- A clear visual distinction between the initial topology and any failure or test condition.

## Excalidraw — Rapid Sketching

Excalidraw is useful when speed and clarity matter more than formal presentation. I use it during early design, when explaining packet flow, and when breaking a troubleshooting problem into smaller parts.

Typical uses include:

- Drafting a topology before building the final Visio diagram.
- Illustrating protocol exchanges or traffic paths.
- Marking suspected failure points during troubleshooting.
- Creating lightweight diagrams for concept notes and discussions.

## Typora — Markdown Authoring

Typora is my primary editor for writing and previewing Markdown. It provides a clean way to work on documentation while checking headings, tables, lists, code blocks, images, and links as they will appear in the repository.

I use Typora to prepare:

- Lab objectives and prerequisites.
- Configuration procedures and command examples.
- Verification results and troubleshooting records.
- Tables, diagrams, screenshots, and cross-references.
- Lessons learned, limitations, and follow-up improvements.

The Markdown files remain plain text and Git-friendly, so their history and changes can be reviewed independently of the editor used to create them.

## Wireshark — Packet Capture and Protocol Analysis

Wireshark helps me verify what is happening on the wire rather than relying only on device status or configuration output. Packet captures are used as supporting evidence when protocol behaviour, timing, fields, or message exchanges need closer inspection.

I use Wireshark to:

- Apply capture and display filters to isolate relevant traffic.
- Inspect headers, flags, options, protocol fields, and packet sequences.
- Follow conversations and compare successful traffic with failed traffic.
- Validate control-plane exchanges and data-plane forwarding behaviour.
- Support troubleshooting conclusions with packet-level evidence.

Before a capture or screenshot is published, sensitive information such as credentials, public addresses, tokens, personal data, and unrelated traffic must be removed or anonymised.

## How the Tools Work Together

A typical lab moves through the following workflow:

1. **Sketch the idea in Excalidraw** to identify devices, links, traffic flows, and the behaviour to test.
2. **Create the final topology in Visio** when the design and addressing plan are stable.
3. **Build the topology in EVE-NG** and record the platform and software assumptions.
4. **Configure and verify devices through SecureCRT**, saving useful command output as evidence.
5. **Capture and analyse traffic with Wireshark** when packet-level validation is required.
6. **Document the lab in Typora**, including the topology, configuration, verification, troubleshooting, and lessons learned.

Not every lab needs every tool. The toolchain is selected according to the evidence required to make the result understandable and reproducible.

## Repository Practice

Tool-generated files are included only when they help someone understand or reproduce a lab. Published material should use clear filenames, identify the tool or format where necessary, and avoid containing credentials, licensed software images, or other sensitive data.

Product names and trademarks belong to their respective owners. The tools listed here are part of my personal lab workflow; their inclusion is not an endorsement or sponsorship statement.
