# Evidence Asset Index

These images are referenced by the main lab document and record the verified baseline state.

| File | Evidence |
| --- | --- |
| <code>image-20260913190002473.png</code> | Complete physical topology and interface mapping |
| <code>image-20260913190531189.png</code> | SW1 VLAN membership |
| <code>image-20260913190551745.png</code> | SW1 802.1Q trunk state, allowed VLANs, and native VLAN |
| <code>image-20260913190631695.png</code> | SW1 Port Security summary |
| <code>image-20260913190737213.png</code> | SW1 Rapid PVST+ state for VLANs 10 and 20 |
| <code>image-20260913191027478.png</code> | DSW1 HSRP role summary |
| <code>image-20260913191327355.png</code> | DSW1 DHCP pool utilisation and excluded-address counts |
| <code>image-20260913191239451.png</code> | DSW1 DHCP bindings |
| <code>image-20260913191526905.png</code> | DSW1 OSPF neighbours |
| <code>image-20260913191615955.png</code> | DSW1 OSPF route paths |
| <code>image-20260913191726706.png</code> | DSW2 OSPF route paths |
| <code>image-20260913191759876.png</code> | R1 OSPF neighbours |
| <code>image-20260913191905952.png</code> | ASA NAT policy and translation-hit counters |
| <code>image-20260913191929476.png</code> | ASA active routing table |
| <code>image-20260913194533573.png</code> | PC1 DHCP and IP information |
| <code>image-20260913194552471.png</code> | PC1-to-HSRP gateway reachability |
| <code>image-20260913194610065.png</code> | PC1 inter-VLAN reachability |
| <code>image-20260913192010641.png</code> | PC1 reachability to the simulated Internet endpoint |

## Capture Rules

- Capture only the command, relevant output, device prompt, and enough context to identify the device.
- Redact usernames, passwords, password hashes, keys, tokens, public management addresses, and unrelated configuration.
- Do not include EVE-NG credentials, browser tabs, desktop notifications, or other personal information.
- Use a consistent terminal size, font, and colour scheme.
- Re-run every command after the final configuration change; do not reuse stale evidence.
- If a command fails, document the failure and correction rather than cropping it out and implying success.
- Optimise PNG files before committing them when this does not reduce text readability.
