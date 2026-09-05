# Routing Policy Matching on Cisco IOS

Routing policy begins with a simple question: **what object must the device match?** A packet, an IPv4 address, a route prefix, a prefix length, or a BGP path requires a different selector. The selector identifies an object; the feature that calls the selector decides what happens next.

This distinction prevents a common mistake: an ACL does not always filter traffic. When attached to an interface with `ip access-group`, it permits or drops packets. When referenced by a route map, offset list, NAT rule, or another feature, it normally classifies objects for that feature instead.

This guide focuses on classic IPv4 policy mechanisms in Cisco IOS and IOS XE. Syntax, supported match and set clauses, numeric ranges, and forwarding implementation can vary by platform and release. Validate every example against the target image before deployment.

## Choose the Matcher by Object

| Object to match | Preferred mechanism | Typical consumer or action |
| --- | --- | --- |
| Packet source, destination, protocol, or Layer 4 port | Extended IPv4 ACL | Interface filtering, classification, PBR, NAT, QoS |
| Packet source address only | Standard IPv4 ACL | Simple filtering or classification |
| IPv4 route prefix and prefix-length range | IPv4 prefix list | BGP policy, redistribution, route maps |
| Route attributes such as tag, metric, next hop, or BGP community | Route-map `match` clauses | Redistribution and BGP policy |
| BGP AS path | AS-path access list | BGP filtering or route-map matching |
| RIP or EIGRP route metric adjustment | Offset list | Add an offset to selected routing updates |
| Forwarded packet requiring a non-default path | Policy-Based Routing (PBR) | Set a next hop or output interface |

> [!TIP]
> Use an ACL for packet fields and a prefix list for route prefixes whenever the calling feature supports both. Prefix lists express prefix-length requirements directly and are easier to review than route matching encoded in an extended ACL.

## Wildcard Masks

Some IOS features, including IPv4 ACLs and selected routing-protocol commands, use wildcard masks rather than subnet masks.

For every address bit:

- Wildcard bit `0`: the candidate bit must match the configured address bit.
- Wildcard bit `1`: ignore the candidate bit.

A precise way to express the comparison is:

```text
candidate-address AND (NOT wildcard)
    =
configured-address AND (NOT wildcard)
```

For an ordinary contiguous network, subtract each subnet-mask octet from 255 to obtain the wildcard mask:

```text
Subnet mask:   255.255.255.0
Wildcard mask:   0.  0.  0.255
```

Unlike a subnet mask, a wildcard mask can be noncontiguous. That flexibility is useful, but noncontiguous masks are harder to audit and should be documented carefully.

| Address and wildcard | Meaning |
| --- | --- |
| `192.0.2.10 0.0.0.0` | Match one address exactly. Equivalent to `host 192.0.2.10`. |
| `192.0.2.0 0.0.0.255` | Match all addresses from `192.0.2.0` through `192.0.2.255`. |
| `10.0.0.0 0.0.255.0` | Match `10.0.x.0`: ignore the third octet but require the fourth octet to be zero. |
| `0.0.0.0 255.255.255.255` | Ignore every bit. Equivalent to `any`. |

> [!NOTE]
> A wildcard mask selects address bits. It does not inherently describe a route's prefix length. The meaning depends on the command that consumes it.

## IPv4 Access Control Lists

An IPv4 ACL is an ordered list of permit and deny entries. It can operate as either:

- **A traffic filter:** the ACL is attached to a forwarding or management path, and its action decides whether matching traffic is allowed.
- **A classifier:** another feature references the ACL to identify packets, addresses, or routes. The calling feature defines the final action.

### Standard and Extended ACLs

| ACL type | Numbered ranges | Named configuration mode | Fields available for matching |
| --- | --- | --- | --- |
| Standard | `1-99`, `1300-1999` | `ip access-list standard NAME` | IPv4 source address |
| Extended | `100-199`, `2000-2699` | `ip access-list extended NAME` | Protocol, source, destination, and supported Layer 4 details |

Named ACLs communicate intent more clearly and make sequence-based editing safer.

```cisco
ip access-list extended USER-TO-DNS
 10 remark Permit DNS from the user subnet
 20 permit udp 192.0.2.0 0.0.0.255 host 198.51.100.53 eq domain
 30 permit tcp 192.0.2.0 0.0.0.255 host 198.51.100.53 eq domain
 90 deny ip any any log
```

The equivalent numbered syntax begins with the global `access-list` command:

```text
access-list acl-number {permit | deny} source [source-wildcard]

access-list acl-number {permit | deny} protocol
  source source-wildcard [source-port-operator]
  destination destination-wildcard [destination-port-operator]
  [options]
```

The keywords `host` and `any` replace common address and wildcard pairs:

```cisco
permit ip host 192.0.2.10 any
permit icmp any host 198.51.100.20 echo
```

### ACL Evaluation Rules

1. IOS evaluates entries in ascending sequence order.
2. The first matching entry determines the ACL result; later entries are not examined.
3. Every ACL has an implicit deny after its final explicit entry.
4. An interface can normally have one IPv4 ACL per protocol, per direction.
5. An inbound ACL is evaluated as the packet enters an interface. An outbound ACL is evaluated after the forwarding decision, before the packet leaves.

Do not assume that a data-plane interface ACL also governs traffic generated by the router itself or all control-plane traffic. Locally originated and device-destined traffic can require management-plane or control-plane controls, depending on the objective and platform.

The `log` option records matches for the entry on which it is configured, whether the entry permits or denies. Logging every packet is not implied and excessive ACL logging can consume device resources.

The extended ACL keyword `established` checks for TCP packets with the ACK or RST flag set. It is a flag test, not a stateful firewall, and it does not validate the complete TCP session.

### Applying and Editing ACLs

Apply an IPv4 ACL to an interface:

```cisco
interface GigabitEthernet0/0
 ip access-group USER-TO-DNS in
```

Restrict inbound VTY access:

```cisco
line vty 0 4
 access-class MGMT-SOURCES in
```

Edit a specific sequence from ACL subconfiguration mode:

```cisco
ip access-list extended USER-TO-DNS
 no 30
 25 permit tcp 192.0.2.0 0.0.0.255 host 198.51.100.53 eq domain
```

> [!WARNING]
> A command such as `no access-list 100` removes the entire numbered ACL. Enter ACL subconfiguration mode and remove a specific sequence when only one entry should be deleted.

Verify both the ACL definition and where it is attached:

```text
show access-lists
show ip access-lists
show ip interface GigabitEthernet0/0
show running-config | section ip access-list
```

Match counters provide evidence that traffic reached an entry, but counters alone do not prove end-to-end application success.

## Using ACLs to Match Routes

When a routing feature references an ACL, the ACL is no longer inspecting a packet header. It is matching values supplied by that routing feature.

### Standard ACL Route Matching

A standard ACL can match the address portion of a route, but it cannot test the route's prefix length.

```cisco
access-list 10 permit 172.16.0.0 0.0.255.255
```

When used as a route selector, this entry matches route network addresses whose first 16 bits are `172.16`. It does **not** mean “match only `172.16.0.0/16`.” More-specific routes such as `172.16.10.0/24` can also match because the ACL does not compare their masks.

```cisco
access-list 10 permit host 192.168.1.1
```

This matches the route address `192.168.1.1`, but the invoking feature still determines whether and how the route mask is considered.

### Extended ACL Route Matching

Some IOS routing-policy features historically interpret an extended ACL as:

- ACL source field: route network address.
- ACL destination field: route subnet mask.

Examples:

```cisco
! Exact 10.0.0.0/16
access-list 100 permit ip 10.0.0.0 0.0.0.0 255.255.0.0 0.0.0.0

! 10.0.x.0 routes with an exact /24 mask
access-list 101 permit ip 10.0.0.0 0.0.255.0 255.255.255.0 0.0.0.0

! Route addresses under 172.16.0.0/16 with masks from /24 through /32
access-list 102 permit ip 172.16.0.0 0.0.255.255 255.255.255.0 0.0.0.255
```

This encoding is difficult to read and easy to misapply because the same extended ACL syntax normally represents packet source and destination addresses. Prefer a prefix list when the policy consumer supports one, and verify the exact interpretation in that feature's command reference.

An ACL deny entry referenced by a route-map `match` clause means “this selector did not match.” It does not automatically execute a route-map deny action. Processing can continue to the next route-map sequence.

## IPv4 Prefix Lists

A prefix list is purpose-built to match both a route's network address and its prefix length.

```text
ip prefix-list list-name [seq sequence]
  {permit | deny} prefix/length [ge minimum-length] [le maximum-length]
```

IOS evaluates prefix-list entries from the lowest sequence number and stops at the first match. If no entry matches, an implicit deny applies. When IOS assigns sequence numbers automatically, it normally starts at 5 and increments by 5.

A candidate route must first fall within the configured base prefix. Its prefix length must then satisfy the length rule:

| Optional keywords | Prefix lengths matched |
| --- | --- |
| Neither `ge` nor `le` | Exactly the base `length` |
| `le maximum` only | Base `length` through `maximum` |
| `ge minimum` only | `minimum` through `/32` |
| Both `ge minimum le maximum` | The inclusive range from `minimum` through `maximum` |

Examples:

```cisco
! Match only the IPv4 default route
ip prefix-list DEFAULT-ONLY seq 5 permit 0.0.0.0/0

! Match every IPv4 route, including the default route
ip prefix-list ALL-ROUTES seq 5 permit 0.0.0.0/0 le 32

! Match every IPv4 route except the default route
ip prefix-list NON-DEFAULT seq 5 permit 0.0.0.0/0 ge 1 le 32

! Match routes with an exact /18 prefix length
ip prefix-list EXACT-18 seq 5 permit 0.0.0.0/0 ge 18 le 18

! Match /24 through /28 routes contained by 10.0.0.0/16
ip prefix-list BRANCHES seq 5 permit 10.0.0.0/16 ge 24 le 28
```

The entry `permit 10.0.0.0/16` without `ge` or `le` matches only the exact `10.0.0.0/16` route. It does not match all more-specific routes.

Verify a list and its counters before attaching it to a routing process:

```text
show ip prefix-list
show ip prefix-list BRANCHES
show ip prefix-list detail
```

## Route Maps

A route map combines ordered matching logic with an action. Each sequence contains:

- A sequence number that determines processing order.
- A `permit` or `deny` action.
- Zero or more `match` clauses.
- Zero or more `set` clauses.

```text
route-map map-name {permit | deny} sequence-number
 match ...
 set ...
```

Use explicit actions and sequence numbers even where IOS supplies defaults. Explicit policy is easier to review and modify safely.

### Route-Map Evaluation

IOS processes sequences in ascending order:

1. If a sequence's match criteria fail, IOS moves to the next sequence.
2. If the criteria match a `deny` sequence, the object is rejected by that policy application.
3. If the criteria match a `permit` sequence, supported `set` actions are applied and processing normally stops.
4. If no sequence matches, the route map has an implicit deny.
5. A sequence with no `match` clause matches every object reaching that sequence.

The effect of “rejected” depends on the consumer. A denied route can be suppressed during redistribution or BGP policy. A packet denied by a PBR route-map sequence is not dropped; it returns to ordinary destination-based forwarding.

For common IOS route-map logic:

- Multiple values on the same `match` command are alternatives, so one may match.
- Different match types in the same sequence must all match.
- Repeated commands and application-specific clauses can have platform-specific behaviour; verify complex Boolean logic on the target release.

### Selector and Route-Map Actions Are Separate

| Selector result | Route-map sequence | Result |
| --- | --- | --- |
| ACL or prefix list permits the object | `permit` | Sequence matches; apply supported `set` actions and accept for this policy. |
| ACL or prefix list permits the object | `deny` | Sequence matches; reject for this policy application. |
| ACL or prefix list denies the object | Either | The `match` clause fails; continue to a later sequence. |
| No later sequence matches | Implicit deny | Reject for this policy application. |

A deny inside the referenced ACL followed by a deny route-map sequence does not create a double negative that permits the object.

### Common Match and Set Clauses

| Purpose | Example clause |
| --- | --- |
| Match route addresses through an ACL | `match ip address 10` |
| Match route prefixes through a prefix list | `match ip address prefix-list BRANCHES` |
| Match a BGP AS-path access list | `match as-path 20` |
| Match a BGP community list | `match community CUSTOMER-ROUTES` |
| Match a route metric or tag | `match metric 100`, `match tag 200` |
| Set a route tag | `set tag 200` |
| Change a metric | `set metric 50` or a supported relative form |
| Set BGP local preference or weight | `set local-preference 200`, `set weight 500` |
| Prepend a BGP AS path | `set as-path prepend 65010 65010` |
| Set BGP origin or community | `set origin igp`, `set community 65010:100 additive` |
| Set a PBR next hop | `set ip next-hop 192.0.2.2` |

Not every `match` or `set` clause is valid for every route-map consumer.

### Example: Select and Tag Routes

```cisco
ip prefix-list INTERNAL seq 5 permit 10.0.0.0/8 ge 16 le 24
!
route-map TAG-INTERNAL permit 10
 match ip address prefix-list INTERNAL
 set tag 100
!
route-map TAG-INTERNAL permit 100
```

Sequence 100 is an explicit empty permit that passes all remaining routes unchanged. Remove it when the intended policy is to deny everything not selected by sequence 10.

The `continue` keyword can allow additional route-map sequences to be evaluated in supported applications. Its availability and interaction with set actions are application- and release-dependent, so it should not be treated as universal route-map behaviour.

Verify route-map structure and match counters:

```text
show route-map
show route-map TAG-INTERNAL
```

## BGP AS-Path Regular Expressions

An AS-path access list uses a regular expression to select routes by their BGP AS_PATH attribute:

```text
ip as-path access-list number {permit | deny} regular-expression
```

Like other access lists, it has an implicit deny at the end.

Common Cisco AS-path regular-expression tokens include:

| Token | Meaning |
| --- | --- |
| `^` | Beginning of the AS-path string |
| `$` | End of the AS-path string |
| `_` | AS boundary, including a space, string boundary, comma, braces, or parentheses |
| `.` | Any single character |
| `*` | Zero or more repetitions of the preceding expression |
| `+` | One or more repetitions |
| `?` | Zero or one repetition |
| `[0-9]` | One digit in the specified character range |
| `[^...]` | One character not in the set |
| `(...)` | Group an expression |
| `\|` | Alternative expressions |

Useful patterns:

```cisco
! Locally originated BGP routes: empty AS_PATH
ip as-path access-list 10 permit ^$

! AS 65001 appears anywhere in the path
ip as-path access-list 11 permit _65001_

! Path begins with AS 65001
ip as-path access-list 12 permit ^65001_

! Route originated in AS 65001
ip as-path access-list 13 permit _65001$
```

Test the expression against the current BGP table before applying policy:

```text
show ip bgp regexp _65001_
```

Then reference it from a route map:

```cisco
route-map FROM-TRANSIT permit 10
 match as-path 11
 set local-preference 80
```

## Offset Lists

An offset list adds a value to the metric of selected incoming or outgoing routing updates. It is associated primarily with distance-vector protocols such as RIP and EIGRP.

```text
offset-list {acl-number | acl-name} {in | out} offset [interface]
```

The ACL selects route network addresses, and the optional interface limits where the rule applies.

```cisco
access-list 15 permit 10.20.0.0 0.0.255.255
!
router rip
 offset-list 15 in 2 GigabitEthernet0/0
```

This example adds two hops to matching RIP routes received on `GigabitEthernet0/0`. Metric interpretation, valid offset range, maximum metric, and saturation behaviour differ between RIP and EIGRP and can vary by implementation. Check the command reference and verify the resulting routing table rather than reusing one numeric range for both protocols.

```text
show ip protocols
show ip route
show access-lists 15
```

## Policy-Based Routing

Policy-Based Routing changes the forwarding decision for selected packets without installing a replacement destination route in the routing table. It commonly matches an extended ACL and sets a next hop.

### Forwarded Traffic

```cisco
ip access-list extended PBR-HTTPS
 10 permit tcp 192.168.10.0 0.0.0.255 any eq 443
!
route-map USE-WAN2 permit 10
 match ip address PBR-HTTPS
 set ip next-hop 192.0.2.2
!
interface GigabitEthernet0/0
 ip policy route-map USE-WAN2
```

Apply `ip policy route-map` on the interface where the selected packets enter the router.

The evaluation outcomes are important:

- ACL permit plus route-map permit: apply the PBR set action.
- ACL deny: the match fails and evaluation continues.
- Matching route-map deny: stop PBR processing and use normal routing.
- No matching PBR sequence: use normal routing.

PBR is therefore not a substitute for a security ACL. A route-map deny does not drop the packet.

### Locally Generated Traffic

Interface PBR does not select packets originated by the router. Apply a separate local policy when that traffic is intentionally in scope:

```cisco
ip local policy route-map USE-WAN2
```

Local PBR can affect management and control traffic. Define a narrow selector, preserve a recovery path, and test from an out-of-band session where possible.

### PBR Verification

```text
show ip policy
show ip local policy
show route-map USE-WAN2
show ip access-lists PBR-HTTPS
show ip route 192.0.2.2
```

Use source-aware ping or traceroute and compare the actual egress path before and after the policy. `debug ip policy` can provide packet-level evidence in a small lab, but debug output can be expensive on a production device.

When setting an output interface instead of a next-hop address, verify how the platform resolves Layer 2 adjacency. A next-hop address is usually clearer on a multi-access Ethernet segment.

## Switching ACL Boundary

Port ACLs (PACLs) and VLAN ACLs (VACLs) are switching-platform features with hardware, direction, and control-protocol limitations. Their interaction order with routed ACLs is platform-specific. They belong in the Switching section and should be validated against the exact switch family rather than treated as generic IOS routing policy.

## Policy Design Workflow

1. Define the object: packet, address, route prefix, prefix length, AS path, or another route attribute.
2. Choose the narrowest matcher that represents that object clearly.
3. Write explicit sequence numbers, actions, and remarks.
4. Account for the selector's implicit deny and the calling feature's no-match behaviour.
5. Verify the selector independently with show commands and counters.
6. Record the routing table and forwarding path before attaching the policy.
7. Apply the policy at the correct attachment point and direction.
8. Repeat the same checks and generate both matching and nonmatching test cases.
9. Roll back the attachment first if the result is unsafe, then correct the selector offline.

## Common Failure Patterns

| Symptom | Likely cause | Check |
| --- | --- | --- |
| Expected route does not match a standard ACL | The ACL matches address bits but not the assumed prefix length | Compare the route network address and use a prefix list |
| Prefix list matches only one aggregate | `ge` and `le` were omitted, so the base prefix length is exact | Review `show ip prefix-list` |
| Route map unexpectedly rejects everything else | No explicit final permit sequence was added | Inspect the last sequence and consumer semantics |
| ACL deny does not trigger a route-map deny | A denied selector is a failed match, not a matched negative action | Separate selector result from route-map action |
| PBR traffic follows the routing table | ACL did not match, next hop was unusable, policy is on the wrong ingress interface, or the traffic is locally generated | Check ACL counters, `show ip policy`, reachability, and local policy |
| BGP regex matches unintended AS numbers | AS boundaries or start/end anchors are missing | Test with `show ip bgp regexp` |
| ACL change removes more than intended | Whole numbered ACL was deleted instead of one sequence | Use named/numbered ACL subconfiguration and sequence editing |

## References

- [Cisco IOS XE Security Configuration Guide: Access Control Lists](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_data_acl/configuration/xe-16-11/sec-data-acl-xe-16-11-book.pdf)
- [Cisco IOS XE IP Routing Commands: IPv4 Prefix Lists](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9500/software/release/17-6/command_reference/b_176_9500_cr/ip_routing_commands.html)
- [Cisco: Filtering BGP Routes with Standard, Extended, and Prefix Lists](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/13750-22.html)
- [Cisco IOS XE Protocol-Independent Features](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9500/software/release/17-17/configuration_guide/rtng/b_1717_rtng_9500_cg/protocol_independent_features.html)
- [Cisco IOS Protocol-Independent Command Reference: Route Maps and PBR](https://www.cisco.com/c/en/us/td/docs/ios/iproute_pi/command/reference/iri_book/iri_pi1.html)
- [Cisco IOS RIP Command Reference: Offset Lists](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_rip/command/irr-cr-book/irr-cr-rip.html)
- [Cisco: Use Regular Expressions in BGP](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/13754-26.html)
