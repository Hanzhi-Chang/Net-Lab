# Common Network Tools

**A practical command reference for connectivity testing, route inspection, packet analysis, performance measurement, file-transfer labs, and authorised network discovery.**

These tools answer different troubleshooting questions. A successful `ping` does not prove that an application works, a failed `traceroute` does not always mean that forwarding has failed, and an open Nmap port does not by itself identify the business purpose of a service. Use several sources of evidence and record the platform, command, time, interface, and expected result for each test.

> [!IMPORTANT]
> Run traffic-generation and scanning tools only on systems and networks that you own or are explicitly authorised to test. Start with a narrow target and conservative rate, especially when using flood ping, iPerf3, Nmap, or packet capture on shared infrastructure.

## Quick Reference

| Question | Primary Tool | Useful Supporting Evidence |
| --- | --- | --- |
| Does a name resolve to the expected address? | `nslookup` | DNS server, record type, TTL, forward and reverse results |
| Is an IP endpoint reachable? | `ping` | Packet loss, round-trip time, TTL, source interface |
| Where does the observed path stop or change? | `tracert` or `traceroute` | Hop addresses, response time, filtering behaviour |
| Which route will the host use? | `route`, `Get-NetRoute`, or `ip route` | Prefix, next hop, interface, metric, routing table |
| Which Layer 2 neighbour maps to an IP address? | `arp`, `Get-NetNeighbor`, or `ip neigh` | MAC address, interface, neighbour state |
| Which address and DHCP settings did the host receive? | `ipconfig`, `nmcli`, or a DHCP client | Address, prefix, gateway, DNS servers, lease data |
| What packets were actually exchanged? | Wireshark | Capture point, filters, timestamps, protocol fields |
| Can files be transferred securely in the lab? | FileZilla Server with FTPS | Listener, certificate, user mapping, passive ports |
| What throughput can the path sustain? | iPerf3 | Direction, protocol, streams, loss, jitter, retransmissions |
| Which authorised hosts and services are visible? | Nmap | Target scope, discovery method, scan type, port state |

## 1. DNS Lookup with nslookup

`nslookup` queries DNS information. It can perform a forward lookup from a name to an address and a reverse lookup from an address to a PTR record. The returned name server is part of the evidence: two resolvers can return different answers because of policy, caching, split DNS, or propagation state.

### Non-interactive mode

Use non-interactive mode for a single query or a script.

```console
# Query the system's configured DNS resolver
nslookup example.com

# Query a specific DNS resolver
nslookup example.com 1.1.1.1

# Request a specific record type
nslookup -type=AAAA example.com
nslookup -type=MX example.com

# Request a reverse lookup
nslookup 192.0.2.10
```

### Interactive mode

Interactive mode is useful for several related queries or when changing query settings.

```console
nslookup
> server 1.1.1.1
> set type=NS
> example.com
> set type=MX
> example.com
> exit
```

Check the queried server, response status, authoritative or non-authoritative indication, returned records, and whether the forward and reverse records agree.

## 2. Reachability Testing with ping

`ping` sends ICMP Echo Request messages and reports Echo Replies, packet loss, and round-trip time. It is useful for reachability and basic path-quality checks, but a missing reply can mean that ICMP is filtered or rate-limited rather than that the destination is down.

### Windows

```console
# Send the default four Echo Requests
ping 192.0.2.10

# Send ten requests
ping -n 10 192.0.2.10

# Send continuously until interrupted
ping -t 192.0.2.10

# Use a 1,400-byte ICMP payload and set Don't Fragment for IPv4
ping -n 4 -l 1400 -f 192.0.2.10

# Force an address family
ping -4 example.com
ping -6 example.com
```

Important Windows options include:

| Option | Purpose |
| --- | --- |
| `-t` | Continue until interrupted. |
| `-n <count>` | Send the specified number of Echo Requests. |
| `-l <size>` | Set the ICMP data size in bytes; the default is 32. |
| `-f` | Set the IPv4 Don't Fragment flag for path-MTU investigation. |
| `-i <TTL>` | Set the IPv4 TTL. |
| `-w <milliseconds>` | Set the reply timeout for each request. |
| `-4` or `-6` | Force IPv4 or IPv6. |

The `-l` and `-f` combination tests an IP path's ability to carry a selected packet size. It does not by itself prove that every Layer 2 segment supports jumbo frames.

### Linux

```console
# Stop after four replies
ping -c 4 192.0.2.10

# Send every 0.5 seconds and stop after a five-second deadline
ping -c 10 -i 0.5 -w 5 192.0.2.10

# Use a 1,400-byte ICMP payload
ping -c 4 -s 1400 192.0.2.10

# Select an interface or source address
ping -I eth0 -c 4 192.0.2.10

# Force an address family
ping -4 example.com
ping -6 example.com
```

Common Linux options include:

| Option | Purpose |
| --- | --- |
| `-c <count>` | Stop after the specified number of replies. |
| `-i <seconds>` | Set the interval between requests. |
| `-w <seconds>` | Set an overall deadline. |
| `-W <seconds>` | Set the wait time for an individual reply. |
| `-s <bytes>` | Set the ICMP data size. |
| `-t <TTL>` | Set the IPv4 TTL. |
| `-q` | Print only the summary. |
| `-l <preload>` | Send a number of requests without first waiting for replies; privileges or limits may apply. |
| `-f` | Send requests at a very high rate; use only in a controlled lab and with appropriate privileges. |
| `-4` or `-6` | Force IPv4 or IPv6. |

### Testing a TCP port

A TCP reachability tool can test whether a connection to a particular port succeeds when ICMP is unavailable or when the application path matters.

```console
tcping <destination> <port>
```

Several unrelated tools use the name `tcping`, and their syntax differs. Record the implementation and version, and check its built-in help before copying options between systems.

## 3. Path Tracing with tracert and traceroute

Path-tracing tools send probes with increasing TTL or Hop Limit values. Each router decrements the value; when it reaches zero, the router normally returns an ICMP Time Exceeded message. The sequence of replies reveals the responding interfaces along the observed path.

An asterisk does not automatically identify a failed forwarding hop. A router can forward traffic while suppressing, filtering, or rate-limiting the diagnostic response. Return paths can also differ from forward paths.

### Windows tracert

Windows `tracert` uses ICMP Echo probes.

```console
tracert example.com
tracert -d -h 20 -w 1000 192.0.2.10
```

| Option | Purpose |
| --- | --- |
| `-d` | Do not resolve hop addresses to names. |
| `-h <hops>` | Set the maximum number of hops. |
| `-w <milliseconds>` | Set the wait time for each reply. |
| `-4` or `-6` | Force IPv4 or IPv6. |

### Linux traceroute

The common Linux implementation uses UDP probes by default and can use ICMP or TCP probes when selected.

```console
traceroute example.com
traceroute -n -m 20 -w 2 192.0.2.10
traceroute -I 192.0.2.10
traceroute -T -p 443 192.0.2.10
```

| Option | Purpose |
| --- | --- |
| `-n` | Do not resolve hop addresses to names. |
| `-f <TTL>` | Set the first TTL. |
| `-m <TTL>` | Set the maximum TTL. |
| `-i <interface>` | Send through a specific interface. |
| `-I` | Use ICMP Echo probes. |
| `-T` | Use TCP SYN probes. |
| `-p <port>` | Set the UDP or TCP destination port, depending on the method. |
| `-s <address>` | Select the source address. |
| `-w <seconds>` | Set the reply wait time. |

## 4. Routing Table Inspection

The routing table determines the next hop and outgoing interface selected for a destination. Read the active table before changing it, and record the original state so that temporary lab changes can be removed safely.

### Windows route

```console
# Display IPv4 and IPv6 route tables and interface indexes
route print

# Display only IPv4 routes
route print -4

# Add a temporary IPv4 route
route add 198.51.100.0 mask 255.255.255.0 192.0.2.1 metric 10 if 12

# Add a persistent IPv4 route
route -p add 198.51.100.0 mask 255.255.255.0 192.0.2.1 metric 10 if 12

# Delete the route
route delete 198.51.100.0 mask 255.255.255.0
```

The most useful fields are the destination prefix, mask, gateway or next hop, interface, and metric. The lowest metric matters only after the destination is matched; route selection first prefers the longest matching prefix.

PowerShell provides a structured view:

```powershell
Get-NetRoute -AddressFamily IPv4 |
    Sort-Object DestinationPrefix, RouteMetric
```

### Linux ip route

`ip route` is the preferred modern interface to the Linux kernel routing tables. The older `route` command may still be available, but new documentation and automation should normally use `iproute2` syntax.

```console
# Show the main IPv4 table
ip route show

# Ask the kernel which route it would use
ip route get 198.51.100.10

# Add a route
sudo ip route add 198.51.100.0/24 via 192.0.2.1 dev eth0

# Replace a default route
sudo ip route replace default via 192.0.2.1 dev eth0

# Delete a route
sudo ip route del 198.51.100.0/24 via 192.0.2.1 dev eth0

# Show IPv6 routes
ip -6 route show
```

Routes added directly with `ip route` are runtime state and normally do not survive a reboot. Persistent configuration depends on the distribution and network manager, so document the method used by the lab host instead of copying a distribution-specific file path without checking it.

## 5. ARP and Neighbour Tables

ARP maps IPv4 addresses to link-layer addresses on the local network. IPv6 uses Neighbor Discovery rather than ARP. A neighbour-table entry can be reachable, stale, incomplete, failed, or permanent; the state is often as important as the recorded MAC address.

### Windows

```console
arp -a
arp -a -N 192.0.2.20
```

PowerShell exposes both IPv4 and IPv6 neighbours:

```powershell
Get-NetNeighbor
Get-NetNeighbor -AddressFamily IPv4
```

### Linux

```console
ip neigh show
ip neigh show dev eth0
ip neigh get 192.0.2.10 dev eth0
ip -6 neigh show
```

An empty entry can simply mean that the host has not recently tried to reach the neighbour. Generate one controlled packet, then inspect the table and packet capture together.

## 6. DHCP Client Information and Renewal

DHCP supplies more than an address. Verification should include the prefix or mask, default gateway, DNS servers, lease time, DHCP server, and the interface on which the lease was obtained.

### Windows

```console
# Display complete interface and lease information
ipconfig /all

# Release and renew all eligible IPv4 leases
ipconfig /release
ipconfig /renew

# Release and renew a named adapter
ipconfig /release "Ethernet"
ipconfig /renew "Ethernet"
```

Releasing a lease can immediately interrupt remote access. Perform the operation from a console or another recovery path when managing a remote system.

### Linux

First identify the active network manager and interface:

```console
ip address show
ip route show
nmcli device status
nmcli device show eth0
```

On systems that use ISC `dhclient` and have it installed:

```console
sudo dhclient -r eth0
sudo dhclient eth0
```

Do not run a second DHCP client alongside NetworkManager, systemd-networkd, or another active manager. Use that manager's supported reconnect or renewal procedure instead.

## 7. Packet Analysis with Wireshark

Wireshark copies packets delivered by the selected capture interface into a capture session and decodes the protocols it recognises. The capture point determines what can be observed: a host capture, switch mirror port, routed link, tunnel endpoint, and firewall interface can all show different versions of the same flow.

### Capture filters and display filters

Wireshark uses two different filter languages:

| Filter | Applied | Result |
| --- | --- | --- |
| Capture filter | Before packets are stored | Only matching packets enter the capture file. |
| Display filter | After packets have been captured | All captured packets remain available, but only matching packets are shown. |

Capture filters use Berkeley Packet Filter syntax. Examples:

```text
host 192.0.2.10
src host 192.0.2.10
dst net 198.51.100.0/24
tcp port 443
udp portrange 5000-5100
arp or icmp
```

The main qualifiers are:

- **Type:** `host`, `net`, `port`, or `portrange`.
- **Direction:** `src`, `dst`, `src or dst`, or `src and dst`.
- **Protocol:** `ether`, `ip`, `ip6`, `tcp`, `udp`, `arp`, or another supported protocol.

Display filters use Wireshark field names and operators. Examples:

```text
ip.addr == 192.0.2.10
tcp.port == 443
dns
icmp
tcp.flags.syn == 1 && tcp.flags.ack == 0
tcp.stream eq 3
```

### Practical workflow

1. Select the exact interface and confirm activity before starting.
2. Apply a capture filter only when file size, privacy, or performance requires it.
3. Reproduce one clearly timed test condition.
4. Stop the capture promptly and apply display filters for analysis.
5. Use **Statistics > Conversations** to identify communicating endpoints.
6. Use **View > Name Resolution** only when names improve interpretation; retain address-based evidence as well.
7. Use **File > Export Specified Packets** when a new file should contain only displayed or marked packets.
8. Record the capture point, filter, time, and expected exchange alongside the `.pcapng` file.

Remote capture can be provided by an extcap source such as SSH-based capture or by `rpcapd`, depending on the environment. Protect remote-capture credentials and traffic, and do not expose an unauthenticated capture service.

Before publishing a capture, remove or anonymise credentials, tokens, cookies, personal data, public addresses, and unrelated traffic.

## 8. File Transfer Labs with FileZilla Server

FileZilla Server can provide FTP and FTP over TLS (FTPS) for lab exercises. Plain FTP sends credentials and data without encryption, so require explicit FTPS when encryption is needed and keep plain FTP inside an isolated demonstration environment.

### Listener security

| Mode | Typical Control Port | TLS Behaviour | Guidance |
| --- | --- | --- | --- |
| Explicit FTPS | 21 | The client connects, then requests TLS with `AUTH TLS`. | Preferred when FTP compatibility is required. |
| Implicit FTPS | 990 | TLS begins as soon as the connection is established. | Use only when both endpoints require and support it. |
| Plain FTP | 21 | No encryption. | Restrict to an isolated lab. |

The administration listener and FTP listener are separate services and must not use the same address and port. Verify the TLS certificate fingerprint and configure the minimum accepted TLS version according to the lab requirements.

### Active and passive data connections

FTP uses separate control and data connections.

| Mode | Data Connection Initiator | Firewall and NAT Consideration |
| --- | --- | --- |
| Active | Server connects back to the client. | The client must accept the inbound data connection. |
| Passive | Client connects to an address and port supplied by the server. | Usually easier through client-side NAT; the server-side passive range must be allowed and forwarded when required. |

### Configuration checklist

1. Configure the administration connection and verify its certificate fingerprint.
2. Create the FTP or FTPS listener and avoid an address or port collision with administration.
3. Prefer **Require explicit FTP over TLS** and configure a suitable certificate.
4. Create users or groups, set credentials, and map native paths to visible virtual paths such as `/`.
5. Configure a bounded passive port range when predictable firewall rules are required.
6. Permit the control listener and passive range through the host firewall and NAT device as applicable.
7. Test authentication, directory visibility, upload, download, and denied access.
8. Review server logs and remove test credentials after the lab.

## 9. Throughput Testing with iPerf3

iPerf3 measures network throughput using a client-server model. TCP tests report throughput and can report retransmissions; UDP tests report throughput, jitter, and packet loss. iPerf3 is not a general latency-measurement replacement for `ping`.

### Basic test

Start the server, which listens on TCP port 5201 by default:

```console
iperf3 -s
```

Run the client against the server:

```console
iperf3 -c 192.0.2.10
```

### Useful options

```console
# Test for 30 seconds and report every second
iperf3 -c 192.0.2.10 -t 30 -i 1

# Test in the reverse direction: server sends to client
iperf3 -c 192.0.2.10 -R

# Use four parallel TCP streams
iperf3 -c 192.0.2.10 -P 4

# Send UDP at a target rate of 100 Mbit/s
iperf3 -c 192.0.2.10 -u -b 100M -t 30

# Omit the first three seconds from the final statistics
iperf3 -c 192.0.2.10 -O 3 -t 30

# Produce JSON output for automation
iperf3 -c 192.0.2.10 -J
```

| Option | Purpose |
| --- | --- |
| `-p <port>` | Set the server listener or client destination port. |
| `-i <seconds>` | Set the reporting interval. |
| `-t <seconds>` | Set the client test duration; the default is 10 seconds. |
| `-u` | Use UDP instead of TCP. |
| `-b <rate>` | Set the target bitrate; especially important for UDP. |
| `-P <streams>` | Use parallel client streams. |
| `-R` | Reverse the test direction. |
| `-O <seconds>` | Exclude initial slow-start or warm-up time from final statistics. |
| `-J` | Return JSON output. |

Run the executable from its installation directory or add that directory to the user's `PATH`. Do not copy binaries or supporting libraries into the Windows system directory.

Record direction, protocol, target rate, duration, stream count, interface speed, host CPU load, and any QoS or policing applied to the path. A test can be limited by either endpoint rather than by the network.

## 10. Authorised Discovery with Nmap

Nmap discovers hosts and classifies how services appear from the scanner's location. Its results describe observed reachability, not an immutable property of the target: a firewall, route, NAT rule, source address, or scan technique can change the result.

### Basic syntax and examples

```console
nmap [scan-types] [options] <target-specification>

# Scan Nmap's default set of common TCP ports
nmap 192.0.2.10

# Scan selected ports
nmap -p 22,80,443 192.0.2.10

# Perform host discovery without a port scan
nmap -sn 192.0.2.0/24

# Detect service versions on selected ports
nmap -sV -p 22,80,443 192.0.2.10

# Save normal, XML, and grepable output with one basename
nmap -sV -p 22,80,443 -oA results/host-services 192.0.2.10
```

### Target specification

Nmap accepts hostnames, individual addresses, CIDR prefixes, address ranges, and input files.

```console
nmap 192.0.2.10
nmap 192.0.2.0/24
nmap 192.0.2.10-20
nmap -iL authorised-targets.txt
nmap 192.0.2.0/24 --exclude 192.0.2.1,192.0.2.254
nmap -iL authorised-targets.txt --excludefile excluded-targets.txt
```

Useful target and name-resolution options include:

| Option | Purpose |
| --- | --- |
| `--resolve-all` | Scan every address returned for a hostname rather than only the first. |
| `-n` | Disable reverse DNS resolution. |
| `-R` | Always perform reverse DNS resolution. |
| `--system-dns` | Use the operating system resolver. |
| `--dns-servers <servers>` | Query the specified DNS servers; do not combine with `--system-dns`. |
| `--unique` | Scan each resolved address only once. |

Always preview or calculate a large range before scanning it. `192.0.2.0/24` contains 256 addresses, whereas a `/8` contains more than 16 million.

### Host discovery

By default, Nmap discovers active hosts before applying later scan phases. Local Ethernet targets normally use ARP for IPv4 or Neighbor Discovery for IPv6 because these methods are efficient and reliable on the local link.

| Option | Purpose |
| --- | --- |
| `-sL` | List targets without sending probes to them; reverse DNS can still occur unless `-n` is used. |
| `-sn` | Perform host discovery without a port scan. |
| `-Pn` | Skip host discovery, treat every target as online, and apply the requested scan to every address. |
| `-PS<ports>` | Send TCP SYN discovery probes, for example `-PS22,443`. |
| `-PA<ports>` | Send TCP ACK discovery probes. |
| `-PU<ports>` | Send UDP discovery probes. |
| `-PY<ports>` | Send SCTP INIT discovery probes. |
| `-PE`, `-PP`, `-PM` | Send ICMP Echo, Timestamp, or Address Mask discovery probes. |
| `-PO<protocols>` | Send IP Protocol Ping probes. The final character is the letter `O`, not zero. |
| `--disable-arp-ping` | Disable implicit ARP or IPv6 ND discovery for local-link targets. |

`-Pn` can make a scan much slower because every address proceeds to the requested scan phase. Use it only when discovery probes are filtered or when the scope specifically requires it.

### Port states

Nmap reports six port states:

| State | Meaning from the Scanner's Viewpoint |
| --- | --- |
| `open` | An application accepts TCP connections, UDP datagrams, or SCTP associations on the port. |
| `closed` | The port is reachable, but no application is listening. |
| `filtered` | Filtering prevents Nmap from determining whether the port is open or closed. |
| `unfiltered` | The port is reachable, but the selected scan cannot determine whether it is open or closed. |
| `open|filtered` | The selected scan cannot distinguish an open port from a filtered port. |
| `closed|filtered` | The selected scan cannot distinguish a closed port from a filtered port. |

### Scan techniques

| Option | Technique | Notes |
| --- | --- | --- |
| `-sS` | TCP SYN scan | Sends SYN probes without intentionally completing a TCP connection; raw-packet privileges are normally required. |
| `-sT` | TCP connect scan | Uses the operating system's `connect` call and is the normal fallback when SYN scan privileges are unavailable. |
| `-sU` | UDP scan | Often slower because open UDP services may not respond to an empty probe. |
| `-sY` | SCTP INIT scan | Classifies SCTP services using INIT and response behaviour. |

Nmap commonly uses a SYN scan when raw-packet privileges are available and a TCP connect scan otherwise. Do not describe `-sS` as invisible: modern hosts, firewalls, IDSs, and packet captures can record it.

### Timing and broader detection

Timing templates range from `-T0` to `-T5`. `-T3` is the normal default; `-T4` is commonly used on a fast and reliable authorised network. Higher numbers are more aggressive and can reduce accuracy or overload fragile systems. There is no `-T6` template.

The `-A` option enables several features, including operating-system detection, version detection, script scanning, and traceroute. Use the individual features you need rather than enabling `-A` by habit, and review script behaviour before running it.

## Recording Useful Evidence

For a reproducible lab, record:

- The tool name and version.
- The operating system and interface used.
- The exact command or filter.
- The source and authorised target scope.
- The date and time, including time zone.
- The expected result and actual result.
- Relevant raw output, packet capture, server log, or screenshot.
- Any firewall, NAT, routing, DNS, or privilege assumptions.
- Cleanup performed after temporary routes, listeners, users, or test services were created.

## References

- Microsoft Learn: [`nslookup`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/nslookup), [`ping`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ping), [`tracert`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tracert), [`route`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/route_ws2008), [`arp`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/arp), and [`ipconfig`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig).
- Linux manual pages: [`ip route`](https://man7.org/linux/man-pages/man8/ip-route.8.html), [`ip neigh`](https://man7.org/linux/man-pages/man8/ip-neighbour.8.html), and [`traceroute`](https://man7.org/linux/man-pages/man8/traceroute.8.html).
- Wireshark User's Guide: [capture filters](https://www.wireshark.org/docs/wsug_html_chunked/ChCapCaptureFilterSection.html) and [display filters](https://www.wireshark.org/docs/wsug_html_chunked/ChWorkBuildDisplayFilterSection.html).
- FileZilla Server documentation: [FTP listeners and connection security](https://filezillapro.com/docs/server/advanced-options/setting-up-ftp-listeners-and-connection-security/) and [passive mode](https://filezillapro.com/docs/server/advanced-options/filezilla-server-passive-mode/).
- ESnet: [Invoking iPerf3](https://software.es.net/iperf/invoking.html).
- Nmap Reference Guide: [target specification](https://nmap.org/book/man-target-specification.html), [host discovery](https://nmap.org/book/man-host-discovery.html), [port states](https://nmap.org/book/man-port-scanning-basics.html), [scan techniques](https://nmap.org/book/man-port-scanning-techniques.html), and [timing templates](https://nmap.org/book/performance-timing-templates.html).
