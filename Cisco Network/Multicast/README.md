# IP Multicast

This directory covers IP multicast from host membership and Layer 2 forwarding to multicast routing, PIM operation, Rendezvous Point design, and policy controls. The material explains how multicast delivers one stream to multiple receivers, how receivers join and leave groups, and how routers and switches build and maintain the forwarding state required to carry the traffic efficiently.

> [!NOTE]
> The study material behind this section has already been completed. It will be reorganized into focused documentation, command references, reusable configurations, and reproducible labs, which will be published progressively.

## Main Content

### Multicast Fundamentals

- Common multicast use cases such as IPTV, market-data distribution, and video or audio conferencing.
- The relationship between IGMP at the receiver edge and PIM in the routed network.
- IPv4 multicast addressing in `224.0.0.0/4` and the purpose of different multicast address ranges.
- Mapping an IPv4 multicast destination to a Layer 2 multicast MAC address.

### Internet Group Management Protocol

- IGMP operation between receivers and the last-hop router.
- Membership queries, membership reports, leave messages, and group-specific queries.
- IGMPv1, IGMPv2, and IGMPv3 behaviour and differences.
- IGMPv3 source filtering and its role in source-specific reception.
- IGMP filtering to control which groups receivers are allowed to join.
- IGMP Proxy for forwarding membership information across specific network designs.

### Layer 2 Multicast

- Why an ordinary Layer 2 switch floods multicast traffic when it does not know which ports contain receivers.
- Cisco Group Management Protocol as a legacy Cisco-proprietary mechanism.
- IGMP Snooping for learning receiver-facing ports from IGMP messages.
- PIM Snooping for controlling multicast forwarding toward multicast routers across a Layer 2 domain.

### Layer 3 Multicast Forwarding

- Multicast forwarding state represented by `(*, G)` and `(S, G)` entries.
- Dense-mode flood-and-prune behaviour and sparse-mode receiver-driven forwarding.
- Reverse Path Forwarding checks, RPF interfaces, and loop prevention.
- Multicast forwarding across GRE tunnels when an intermediate network does not support multicast.
- Multicast scoping to restrict how far multicast traffic can travel.

### Protocol Independent Multicast

- PIM operation and its dependence on the unicast routing table rather than a specific unicast routing protocol.
- PIM neighbour discovery and adjacency maintenance with Hello messages.
- PIM Designated Router election on multi-access networks.
- PIM Dense Mode workflow, source-rooted trees, pruning, grafting, and multicast routing state.
- PIM Sparse Mode workflow, receiver joins, source registration, Rendezvous Point Trees, and Shortest Path Trees.
- PIM Sparse-Dense Mode behaviour.
- PIM Prune Override on shared networks.
- Bidirectional PIM for many-to-many multicast communication.

### Rendezvous Point Design and Discovery

- The role of the Rendezvous Point in PIM Sparse Mode.
- Static RP configuration.
- Cisco Auto-RP with Candidate RPs and Mapping Agents.
- Bootstrap Router operation, Candidate BSRs, Candidate RPs, and RP selection.
- Anycast RP for redundancy and Multicast Source Discovery Protocol for source-state exchange.

### Policy, Filtering, and Operational Features

- Multicast packet and protocol address references.
- Multicast Stub Routing for remote sites connected over constrained links.
- `accept-rp` controls for limiting the RP and group mappings accepted by a router.
- `accept-register` controls for limiting which sources and groups may register with an RP.
- Multicast boundary filtering.
- Conversion between selected broadcast and multicast traffic patterns.

## Disclaimer

Multicast behaviour and command support can differ between classic IOS, IOS XE, device families, feature sets, and software releases. Each detailed document or lab should identify the platform, image, topology, and software version used for validation.
