# Linux Network Configuration and Management

## Validate Network Configuration

Each network interface has a name that is used to identify and configure it.

### Network Interface Names

Earlier RHEL releases commonly used names such as `eth0`, `eth1`, and so on. These names were assigned according to the order in which the operating system detected the interfaces. Because the detection order could change, an interface did not always receive the same name after a reboot or hardware change.

RHEL 7 and most modern systemd-based distributions, including Ubuntu, Debian, and Fedora, use predictable network interface names by default. These names are derived from firmware information, the PCI topology, and the interface type, so they are generally consistent across reboots.

- The name begins with the interface type, such as `en` for Ethernet or `wl` for wireless LAN.
- `oN` identifies an onboard device with firmware-provided index `N`, for example, `eno1`.
- `sN` identifies a device in PCI hot-plug slot `N`, for example, `ens1`.
- `pMsN` identifies a device on PCI bus `M` in slot `N`, for example, `enp2s0`.
- `fN` identifies PCI function `N`. It is commonly seen on multi-port adapters, for example, `enp2s0f1`.

The values of `N` depend on firmware and hardware topology; they do not always begin at 0.

The actual naming policy can be changed by the distribution, system administrator, virtual machine platform, or cloud image. Therefore, never assume that an interface is named `eth0` or `ens33`; use `ip link show` to confirm its name first.

### Inspect Interfaces and IP Addresses

```shell
# List all network interfaces
ip link show

# Display link-layer and IP information for an interface
ip addr show dev ens33

# Display interface states and addresses in a concise format
ip -br addr show

# Display IPv4 or IPv6 addresses only
ip -4 addr show dev ens33
ip -6 addr show dev ens33

# Display interface traffic counters and error statistics
ip -s link show ens33
```

The interface name in these commands, such as `ens33`, must be replaced with the actual name on the system.

### Inspect the Routing Table

```shell
# Display the IPv4 routing table
ip route

# Display the IPv6 routing table
ip -6 route
```

### Inspect Network Sockets

The `/etc/services` file maps well-known service names to transport protocols and port numbers. Commands such as `ss` can use these mappings when displaying socket information.

```shell
# Display all TCP sockets
ss -ta
```

`ss` is the modern replacement for the legacy `netstat` command. Common options include:

- `-n`: Display numeric addresses and port numbers instead of resolving names.
- `-t`: Display TCP sockets.
- `-u`: Display UDP sockets.
- `-l`: Display listening sockets only.
- `-a`: Display both listening and non-listening sockets.
- `-p`: Display the process using each socket. This may require root privileges.

For example, `ss -tulpn` displays listening TCP and UDP sockets, their numeric addresses and ports, and the associated processes.

### Test Connectivity

```shell
# Send three ICMP Echo Requests to an IPv4 address
ping -c 3 192.168.10.1

# Send one ICMPv6 Echo Request to an IPv6 address
ping -6 -c 1 2001:db8:0:1::1
```

A successful response confirms basic IP reachability. However, it does not prove that a particular application or TCP/UDP port is accessible.

### Trace the Network Path

```shell
# Discover the path and path MTU to an IPv4 destination
tracepath www.example.com

# Use IPv6
tracepath -6 www.example.com
```

`tracepath` displays the routers along the path to a destination and can also identify changes in the path maximum transmission unit (PMTU). Unlike `traceroute`, it normally does not require root privileges.

### Distribution Differences

The commands in this section come from common Linux networking utilities and use the same syntax on current RHEL, Fedora, Ubuntu, and Debian systems. The main differences are the package names and the tools used to store persistent network configuration.

| Distribution | Packages that provide these commands                         | Common persistent configuration method                       |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| RHEL         | `iproute` provides `ip` and `ss`; `iputils` provides `ping` and `tracepath` | NetworkManager, usually managed with `nmcli` or connection profiles |
| Fedora       | `iproute` provides `ip` and `ss`; `iputils` provides `ping` and `tracepath` | NetworkManager, usually managed with `nmcli`                 |
| Ubuntu       | `iproute2` provides `ip` and `ss`; `iputils-ping` and `iputils-tracepath` provide the corresponding diagnostic commands | Netplan, which uses NetworkManager or `systemd-networkd` as its renderer |
| Debian       | `iproute2` provides `ip` and `ss`; `iputils-ping` and `iputils-tracepath` provide the corresponding diagnostic commands | May use NetworkManager, `systemd-networkd`, or `/etc/network/interfaces`, depending on the installation |

If a command is missing, install the relevant packages:

```shell
# Ubuntu or Debian
sudo apt install iproute2 iputils-ping iputils-tracepath

# Fedora or RHEL
sudo dnf install iproute iputils
```

The older `netstat` command belongs to the `net-tools` package and is often not installed by default. Use `ss` for new scripts and routine troubleshooting. Also prefer `ping -6` and `tracepath -6`: some releases provide the older `ping6` and `tracepath6` command names, but their availability is less consistent.

Commands such as `ip addr` and `ip route` directly inspect or temporarily modify the kernel's current network state. Configuration changes made with them normally do not survive a reboot. Use the distribution's persistent configuration method when a change must be retained.

## Configure Network

### Describe the NetworkManager Service

`NetworkManager` monitors network devices and manages their network configuration. The `nmcli` command-line utility communicates with this service to display, create, modify, activate, and deactivate network connections.

NetworkManager distinguishes between **devices** and **connection profiles**:

- A **device** is a physical or virtual network interface, such as `eno2`, a bridge, or a VLAN interface.
- A **connection profile**, often shortened to a *connection*, is a collection of settings that can be applied to a compatible device. Each profile has a name (`connection.id`) and a UUID.

Multiple profiles can apply to the same device. For example, one using DHCP and another using static addressing, but normally only one can be active on that device at a time. When the `keyfile` plugin is used, locally created profiles are normally stored in `/etc/NetworkManager/system-connections/`. A profile name does not have to match the device name or the `.nmconnection` filename.

#### View Network Information

```shell
# Display the status of all network devices
nmcli device status

# List all connection profiles
nmcli connection show

# List active profiles only
nmcli connection show --active

# Display one profile and its active data, if active
nmcli connection show eno2con
```

Here, `eno2` is a device name and `eno2con` is a connection profile name.

### Add a Network Connection

#### DHCP Configuration

```shell
nmcli connection add con-name eno2con type ethernet ifname eno2 \
  ipv4.method auto ipv6.method auto
```

For IPv4, `auto` means DHCP. For IPv6, `auto` uses router advertisements and may also use DHCPv6, depending on the network.

#### Static Configuration

```shell
nmcli connection add con-name eno4con type ethernet ifname eno4 \
  ipv4.method manual ipv4.addresses 192.0.2.7/24 ipv4.gateway 192.0.2.1 \
  ipv6.method manual ipv6.addresses 2001:db8:0:1::c000:207/64 \
  ipv6.gateway 2001:db8:0:1::1
```

The properties are named `ipv4.addresses` and `ipv6.addresses`, even when only one address is configured. The value `manual` tells NetworkManager to use static addressing.

### Manage Network Connections

```shell
# Activate a connection profile
nmcli connection up eno2con

# Disconnect a device and prevent immediate automatic reconnection
nmcli device disconnect eno2
```

Be careful when changing the interface used by a remote SSH session because activating or disconnecting it can interrupt access to the system.

#### Update Connection Settings

A persistent profile can request dynamic configuration. For example, a saved profile with `ipv4.method auto` receives its current address, gateway, and DNS information from DHCP. Therefore, it is more accurate to distinguish between:

- **Profile settings**, such as `ipv4.method`, `ipv4.addresses`, and `connection.autoconnect`.
- **Active data**, such as the addresses, routes, and DHCP information currently applied to the device.

In `nmcli connection show PROFILE` output, lowercase groups such as `ipv4.*` contain profile settings. Uppercase groups such as `IP4`, `IP6`, `DHCP4`, and `DHCP6` contain active data.

```shell
# Configure static IPv4 and IPv6 addresses
nmcli connection modify eno2con \
  ipv4.method manual ipv4.addresses 192.0.2.2/24 ipv4.gateway 192.0.2.254 \
  ipv6.method manual ipv6.addresses 2001:db8:0:1::a00:1/64 \
  ipv6.gateway 2001:db8:0:1::1 connection.autoconnect yes

# Restore automatic address configuration and clear the old static values
nmcli connection modify eno2con \
  ipv4.method auto ipv4.addresses "" ipv4.gateway "" \
  ipv6.method auto ipv6.addresses "" ipv6.gateway ""

# Append a DNS server without replacing existing entries
nmcli connection modify eno2con +ipv4.dns 192.0.2.53
```

For a multi-value property such as `ipv4.dns`, the `+` prefix appends a value, `-` removes a value, and using no prefix replaces the current value.

#### Apply Updated Settings

`nmcli connection modify` saves changes to the profile immediately, so it does not require `nmcli connection reload`. However, most changes must still be applied to the active device:

```shell
# Reactivate the profile and apply all settings
nmcli connection up eno2con

# Attempt to apply supported changes without reconnecting
nmcli device reapply eno2
```

If a `.nmconnection` file is edited manually, tell NetworkManager to read it again:

```shell
# Reload all connection files
nmcli connection reload

# Load or reload one specific file
nmcli connection load /etc/NetworkManager/system-connections/eno2con.nmconnection
```

`nmcli connection load` requires the actual file path, not a device or profile name. A manually created keyfile should normally be owned by `root` and have permissions of `600`.

#### Delete a Network Connection

```shell
nmcli connection delete eno2con
```

This deletes the profile, not the physical interface.

#### Check NetworkManager Permissions

```shell
nmcli general permissions
```

This shows which NetworkManager operations the current user is authorized to perform. Modifying system-wide profiles may require `root` privileges or PolicyKit authorization.

### Distribution Differences

The `nmcli` syntax is largely the same across distributions. The important difference is whether NetworkManager controls the target interface.

| Distribution | Typical behavior                                             |
| ------------ | ------------------------------------------------------------ |
| RHEL         | NetworkManager is the standard network management service. The commands and keyfile path above apply directly to current releases. |
| Fedora       | Uses NetworkManager by default, so the same `nmcli` workflow normally applies. |
| Ubuntu       | Persistent network configuration is normally defined through Netplan, which can use NetworkManager or `systemd-networkd` as its renderer. For Netplan-managed interfaces, edit `/etc/netplan/*.yaml` and run `sudo netplan apply`. |
| Debian       | Desktop installations commonly use NetworkManager, while other installations may use `ifupdown`, `/etc/network/interfaces`, or another network service. Interfaces listed in `/etc/network/interfaces` are normally not managed by NetworkManager. |

Before using `nmcli`, confirm that NetworkManager is running and manages the device:

```shell
systemctl status NetworkManager
nmcli device status
```

If the device appears as `unmanaged`, identify which service owns it. Do not configure the same interface through multiple network management systems.
