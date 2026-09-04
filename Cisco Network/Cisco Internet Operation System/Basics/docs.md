# Cisco IOS Basics

This module introduces the operating-system, storage, boot, software-image, and licensing foundations of Cisco IOS and IOS XE devices. These topics support routine operational work such as identifying a platform, checking system health, managing files, selecting a boot image, and preparing for software recovery.

The material is intended for readers who understand basic networking concepts but have little or no hands-on experience with Cisco devices. Commands and behaviour vary by hardware family and software release, so examples should always be verified against the documentation for the device being used.

## 1. Cisco IOS and IOS XE

Cisco IOS, short for **Cisco Internetwork Operating System**, is the traditional network operating system used by many generations of Cisco routers and switches. Classic IOS is commonly described as a monolithic system: many operating-system functions share the same kernel and memory space. This design is efficient on resource-constrained hardware, but a serious software fault can affect the entire control plane. A traditional IOS software upgrade also normally replaces the system image and requires a reload.

Cisco IOS XE preserves the familiar IOS command-line interface and feature set while using a more modular architecture. It runs on a Linux kernel, and the traditional IOS control-plane functions run primarily inside a process called **IOSd**. Other platform services can run as separate Linux processes, which provides stronger process isolation, improved resource management, and a foundation for capabilities such as APIs, model-driven programmability, application hosting, and software maintenance updates. The exact level of process restartability and high availability still depends on the platform, feature, and release; a process failure can therefore still affect device operation.

IOS XE is used across several Cisco enterprise product families. Common examples include Catalyst 9000 switches, ISR 4000 routers, ASR 1000 routers, Catalyst 8000 Edge Platforms, and virtual or cloud-based Catalyst 8000V instances. Product names alone must not be used to infer the operating system: for example, the ASR 9000 family uses IOS XR, while Nexus switches normally use NX-OS.

### CLI Compatibility and Operational Checks

Much of the classic IOS CLI remains available in IOS XE, which makes existing operational knowledge transferable. The two systems are not identical, however. Command availability, output, defaults, and feature behaviour can vary between operating systems, platforms, licences, and releases. This guide therefore labels platform-specific commands where necessary instead of treating IOS and IOS XE as interchangeable.

A network engineer must monitor the device as well as configure it. The following commands provide a useful first view of identity, software, hardware, CPU, and memory state:

```text
! Identify the platform, software release, uptime, and reload reason.
show version

! Display installed chassis, modules, power supplies, and serial numbers.
show inventory

! Review CPU and memory use by IOS or IOSd processes.
show processes cpu sorted
show processes memory sorted

! Inspect platform resources and hardware state on supported IOS XE devices.
show platform
show platform resources
show environment all

! Discover additional platform-specific monitoring commands.
show platform ?
show platform software ?
```

The `show processes` commands focus mainly on IOS or IOSd activity. On IOS XE, packet-forwarding hardware and Linux-based platform services may require additional platform-specific commands. For example, some ASR and ISR platforms provide commands for QuantumFlow Processor (QFP) utilisation, while Catalyst platforms expose their own forwarding and resource views.

In the examples in this repository, lines beginning with `!` are annotations or visual separators and are not commands to enter. Cisco configurations commonly use `!` as a section separator, but it should not be treated as a universal comment syntax equivalent to `#` in a programming language.

Official background: [Cisco networking software overview](https://www.cisco.com/c/en/us/products/ios-nx-os-software/index.html).

## 2. Cisco IOS Filesystem

Cisco IOS uses an Integrated File System (IFS) to give the CLI a consistent way to access local storage, configuration data, internal functions, and remote file servers. A file is normally identified with the following format:

```text
filesystem-prefix:[directory/]filename
```

For example, `flash:configs/before-change.cfg` identifies a file in the `configs` directory of the device's primary flash filesystem. The available prefixes vary by platform, so the device itself should be checked before a file operation is planned.

### Storage and Filesystem Prefixes

The main storage areas have different purposes:

- `flash:` or `bootflash:` normally holds system images, software packages, crash data, and files that must survive a reload. Some platforms expose additional devices such as `flash-1:`, `disk0:`, or `usbflash0:`.
- `nvram:` traditionally contains the startup configuration. `nvram:startup-config` is therefore another way to identify the saved configuration on platforms that use NVRAM for this purpose.
- `system:` is a virtual or opaque filesystem that exposes internal objects. For example, `system:running-config` represents the active configuration rather than a conventional file stored on flash.
- `tftp:`, `ftp:`, `scp:`, `http:`, and `https:` represent network filesystems. They allow a `copy` operation to use an external server as its source or destination when the platform supports the protocol.
- `null:` discards data written to it. It can be useful for testing whether a remote file is readable without saving the file locally.

The `show file systems` command displays the filesystems available on the current device:

```text
show file systems
```

Its output includes the total size, free space, filesystem type, permissions, and prefix. Common types include `flash` or `disk` for persistent storage, `nvram` for non-volatile configuration storage, `network` for remote servers, and `opaque` for internally generated objects. The permission flags are `ro` for read-only, `wo` for write-only, and `rw` for read-write. An asterisk identifies the current default filesystem, while a hash sign in the prefix field identifies a bootable filesystem on platforms that use that notation.

### Navigating and Reading Files

The basic filesystem commands resemble familiar operating-system commands:

```text
! Display the current default filesystem or directory.
pwd

! List the current directory or a named filesystem.
dir
dir flash:

! Change the current location.
cd flash:
cd flash:configs

! Display a text file.
more flash:configs/before-change.cfg
```

Using the full prefix makes a command easier to understand and reduces the chance of working in the wrong storage area. `show flash:` appears in older material and on some platforms, but `dir flash:` fits the IFS syntax and is generally clearer.

### Managing Local Files

Files stored on flash can be organised into directories and copied with the same `copy` command used for configurations and network transfers:

```text
mkdir flash:configs
copy running-config flash:configs/before-change.cfg
dir flash:configs
more flash:configs/before-change.cfg
```

To copy a file between local locations, provide both the source and destination:

```text
copy flash:old-config.cfg flash:configs/old-config.cfg
```

Deletion commands should be used only after the exact prefix and filename have been checked:

```text
delete flash:old-config.cfg
rmdir flash:empty-directory
```

Some filesystems support recovery commands such as `undelete`, but this must not be treated as a reliable rollback method. Keep a verified external backup before deleting a system image or an important configuration.

### Accessing Remote File Servers

During a network copy, the Cisco device is normally the client and the remote system is the server. A URL can identify the protocol, server, path, and filename:

```text
copy tftp://192.0.2.10/images/router-image.bin flash:
copy scp://netops@192.0.2.10/backups/router.cfg flash:restored.cfg
```

TFTP is simple but provides neither authentication nor encryption. FTP supports authentication but does not protect credentials or data in transit. SCP and HTTPS are safer choices when they are supported and correctly configured. Avoid embedding a password in a URL because it can be exposed in command history, logs, or documentation.

### Working with TAR Archives

IOS can create and inspect TAR archives on supported platforms. This is useful when a web interface or another software component consists of several related files:

```text
archive tar /create flash:configs.tar flash:configs
archive tar /table flash:configs.tar
archive tar /xtract flash:configs.tar flash:restored-configs
```

Available options differ across releases. Confirm the syntax with `archive tar ?`, check free space before extraction, and inspect the destination afterwards.

## 3. Configuration Register

The configuration register is a 16-bit platform setting that influences how a device starts. It is written as four hexadecimal digits, such as `0x2102`. Unlike an interface or routing command, it is not an ordinary line in the startup configuration, although the `config-register` command used to change it appears in the running configuration on many platforms.

The register can control several low-level behaviours, including:

- how the device selects a system image;
- whether it uses or temporarily ignores the startup configuration;
- whether a Break sequence can interrupt startup; and
- the console speed on platforms that implement those bits.

The low-order four bits form the **boot field**. A value of `0x0` normally stops the device in ROMMON, `0x1` requests the first bootable image in flash on supported legacy platforms, and values from `0x2` through `0xF` tell the device to use configured boot instructions and platform fallback logic. Exact behaviour must be checked for the hardware in use.

### Viewing and Changing the Register

The current value is usually shown near the end of `show version` output:

```text
show version | include register
show bootvar
```

Many classic IOS routers use `0x2102` for normal operation. A change is made in global configuration mode and normally takes effect at the next reload:

```text
configure terminal
config-register 0x2102
end
write memory
show version | include register
```

The verification output may show both the value currently in use and the value that will take effect after the next reload. Do not reload a production device merely to test a register change; confirm the image, boot variables, console access, and recovery plan first.

### Password Recovery and 0x2142

On many legacy IOS routers, `0x2142` sets the bit that tells the device to ignore the startup configuration during the next boot. This provides access to the CLI without applying the saved passwords. The configuration is still present in NVRAM and must be merged into memory before credentials are changed:

```text
! Entered in ROMMON on platforms that support this procedure.
confreg 0x2142
reset

! Entered after IOS has started without the saved configuration.
enable
copy startup-config running-config
configure terminal
config-register 0x2102
! Repair the required credentials here.
end
write memory
```

The order matters. Saving the blank running configuration before copying the startup configuration can overwrite the configuration that is being recovered. Password-recovery procedures differ on modern IOS XE platforms, and the `no service password-recovery` feature can deliberately restrict recovery. Always follow the recovery guide for the exact model and release.

## 4. IOS Boot Process

The boot process moves the device from hardware initialisation to an operational IOS system. Although the messages and implementation vary, the main sequence is consistent across many classic IOS platforms.

1. **Power-on self-test:** The device checks processors, memory, interfaces, and other essential hardware.
2. **Bootstrap and ROMMON:** Code stored in ROM or boot storage initialises enough hardware to locate a system image. ROMMON provides a limited recovery environment if normal startup cannot continue.
3. **System-image selection:** The bootstrap evaluates the boot field, configured `boot system` statements, boot variables, and platform fallback rules.
4. **Image loading:** The selected IOS image is decompressed or loaded into memory and begins to initialise system services and hardware drivers.
5. **Configuration loading:** IOS reads the startup configuration and applies it as the running configuration. If no usable startup configuration is found, the device may offer the initial configuration dialog.
6. **Normal operation:** Interfaces, protocols, and management services enter their configured states, and the CLI becomes available.

ROMMON is not a full IOS environment. It contains a small command set for tasks such as selecting an image manually, recovering a damaged image, changing certain boot settings, or performing a platform-supported password-recovery procedure.

### Inspecting the Current Boot State

Before changing any boot setting, collect the current state:

```text
! Show the image that is currently running.
show version | include System image file

! Show boot variables and the configuration-register value.
show bootvar

! Show configured boot statements.
show running-config | include ^boot system

! Confirm that candidate images exist in local storage.
dir flash:
```

Some platforms use `show boot` or another platform-specific command instead of `show bootvar`.

### Selecting a Local Image

Classic IOS uses `boot system` statements to define the preferred image. Multiple statements can provide an ordered fallback:

```text
configure terminal
no boot system
boot system flash:new-image.bin
boot system flash:known-good-image.bin
end
write memory
show bootvar
```

The first statement selects the new image, while the second retains a known-good local image as a fallback. This behaviour is platform-dependent, so the result must be confirmed in the boot variables and platform documentation. Do not assume that IOS will always choose the image with the alphabetically first or smallest filename.

### Network Boot and ROMMON Recovery

Classic IOS can be configured to request an image from a network server, commonly with TFTP. This was useful when local flash was too small or during recovery, but it makes startup dependent on network reachability and an unencrypted protocol. It is rarely a preferred normal boot design.

Some ROMMON versions provide variables similar to the following:

```text
IP_ADDRESS=192.0.2.20
IP_SUBNET_MASK=255.255.255.0
DEFAULT_GATEWAY=192.0.2.1
TFTP_SERVER=192.0.2.10
TFTP_FILE=router-image.bin
tftpdnld -r
```

Variable names, address syntax, storage behaviour, and support for `tftpdnld` differ substantially across platforms. The `-r` option runs the image from memory on certain routers instead of writing it to flash. Treat this as a recovery pattern, not a universal command sequence, and verify the exact ROMMON guide before using it.

## 5. Upgrading an IOS Image

An IOS image upgrade replaces the software that the device loads at startup. Copying the file is only one part of the change. A safe upgrade also checks compatibility, protects the current state, verifies the transferred image, defines a rollback path, and confirms service operation after the reload.

The workflow below describes the traditional IOS image process. Modern IOS XE platforms may use a platform-specific software installation workflow rather than only changing a boot statement. Always use the upgrade guide and release notes for the exact model, current release, and target release.

### Preparing the Change

Record the device state before downloading an image:

```text
show version
show inventory
show bootvar
show file systems
dir flash:
show running-config | include ^boot system
```

Confirm the following before proceeding:

- The target image supports the hardware, modules, licences, memory, and required features.
- Cisco's release notes do not require an intermediate upgrade, ROMMON upgrade, or another prerequisite.
- Local storage has room for the new image without deleting the known-good image needed for rollback.
- The running and startup configurations have been saved and copied to external storage.
- A maintenance window, console or out-of-band access, and a tested recovery path are available.

Configuration backups can be sent to a remote server with an interactive copy operation:

```text
copy running-config scp:
copy startup-config scp:
```

### Transferring the Image

The device acts as a client when it retrieves an image. Common choices include:

- **SCP:** encrypted and authenticated; normally preferable when supported;
- **HTTPS:** encrypted and useful on platforms that support HTTP-based file copies;
- **FTP:** authenticated but unencrypted; suitable only on a trusted management network; and
- **TFTP:** simple and widely supported, but unauthenticated and unencrypted.

Examples:

```text
copy scp://netops@192.0.2.10/images/router-image.bin flash:
copy tftp://192.0.2.10/images/router-image.bin flash:
```

After the copy completes, use `dir flash:` to confirm the filename and size. Do not delete the existing boot image simply to make the directory look tidy.

### Verifying Image Integrity

Compare a locally calculated digest with the value published for the image by Cisco:

```text
verify /md5 flash:router-image.bin
```

Some platforms also support stronger algorithms such as SHA-512. MD5 can detect an incomplete or corrupted transfer, but it is not sufficient by itself to establish that a file came from a trusted source. Obtain the image and expected digest through an authorised Cisco download channel.

### Activating and Verifying a Classic IOS Image

For a classic image-based boot workflow, configure the new image first and retain the old image as a fallback where the platform supports ordered boot statements:

```text
configure terminal
no boot system
boot system flash:router-image.bin
boot system flash:known-good-image.bin
end
write memory
show bootvar
```

Check every filename against `dir flash:` before reloading. After the maintenance window begins, reload the device through the normal CLI rather than by interrupting power:

```text
reload
```

After startup, verify more than the version number:

```text
show version
show bootvar
show logging
show ip interface brief
```

Also check the interfaces, neighbours, routes, tunnels, security functions, and management services required by that device. If validation fails, use the documented rollback method while console access and the maintenance window are still available.

## 6. IOS Licensing

Cisco licensing determines which software features or capacity levels a device is entitled to use. The process has changed several times, so commands from one generation must not be applied to another without checking the platform and release.

### How IOS Licensing Evolved

Early IOS platforms often used different image feature sets. A device might require a different image to add security, voice, or advanced IP services. Image names and purchase options therefore indicated both the software release and the included features.

Later platforms commonly used a universal image containing several technology packages. A base package provided standard functions, while a licence activated packages such as Security, Data, or Unified Communications. These platforms used mechanisms such as Product Authorization Keys (PAKs), device-bound `.lic` files, evaluation licences, and Right-to-Use declarations.

Current IOS XE platforms generally use Cisco Smart Licensing or **Smart Licensing Using Policy**. Entitlements are managed through a Cisco Smart Account and Cisco Smart Software Manager (CSSM), while the device records licence usage and reports it according to the policy and deployment method. Direct cloud connectivity is not the only design; supported platforms can use tools such as Cisco Smart Licensing Utility, an on-premises manager, or offline workflows.

### Device and Licence Identity

The Unique Device Identifier (UDI) normally combines the Product ID (PID) and serial number. Legacy PAK registration used the UDI to bind a purchased entitlement to a particular device. It remains useful when inventory or licensing support cases must identify a product instance.

```text
show license udi
show inventory
```

Useful licensing checks on current IOS XE releases include:

```text
show license summary
show license usage
show license status
show license all
```

The available commands and output fields vary. Read the reported status, entitlement name, count, authorisation requirements, reporting deadlines, and account information instead of checking only whether licensing is globally enabled.

### Legacy Licence Files and Technology Packages

Older IOS and IOS XE releases may use commands such as:

```text
show license
show license feature
license install flash:device-license.lic
license boot module <module-name> technology-package <package-name>
```

These commands belong to legacy software-activation workflows and may require a reload or acceptance of licence terms. Temporary licences were often issued for a fixed period, such as 60 days, but duration and expiry behaviour were product-specific. Do not assume that an expired feature will remain enabled simply because one older platform operated that way.

### Licensing Checks Before a Change

Before an upgrade, replacement, or feature activation:

- record the existing licence status and UDI;
- confirm that the target release uses the same licensing model;
- check whether the entitlement is tied to a device, Smart Account, subscription, throughput level, or feature tier;
- understand any reporting or authorisation deadline; and
- include licensing verification in the post-change test plan.

Licensing is both a technical and an entitlement issue. A feature appearing in the CLI does not by itself prove that the organisation is licensed to use it.

## 7. Router Host Client and Server Roles

An IOS device normally forwards traffic as a router or switch, but it can also originate client connections and provide selected network services. These capabilities are useful for testing and small lab environments. Every enabled service also increases the device's attack surface, so production devices should expose only the management and infrastructure services that are required.

### Operating as an IP Host

When IP routing is disabled, the device behaves as an IP host and needs a default gateway to reach remote networks:

```text
configure terminal
no ip routing
ip default-gateway 192.0.2.1
end
```

This pattern is common on a Layer 2 switch or in a limited boot environment. It is unusual to disable routing on a production router. When IP routing is enabled, use a routed default route instead:

```text
ip route 0.0.0.0 0.0.0.0 192.0.2.1
```

Verify the selected path with `show ip route` and test reachability from the intended source interface or VRF.

### Testing a Remote TCP Service

The IOS Telnet client can open a TCP connection to a specific destination port:

```text
telnet 192.0.2.10 22
telnet 192.0.2.10 443
```

A successful connection shows that the device can reach the destination and complete a TCP handshake on that port. A refusal normally means the host is reachable but no service is accepting the connection, while a timeout can indicate packet loss, filtering, missing routes, an unavailable host, or an unresponsive service.

This test does not validate the application protocol. Connecting to TCP 443, for example, does not prove that TLS negotiation, certificates, authentication, or the web application work. To leave a Telnet session, use the platform's escape sequence, commonly `Ctrl-Shift-6` followed by `x`, and close the suspended session if necessary.

### Using DNS as a Client

IOS can query an external DNS resolver and can also use a local static host table:

```text
configure terminal
ip name-server 192.0.2.53
ip domain lookup
ip domain name lab.example
ip host server1.lab.example 192.0.2.50
end

show hosts
ping server1.lab.example
```

`ip name-server` identifies the resolver. `ip domain lookup` enables name resolution, while `ip domain name` defines the device's default domain name and is also used by features such as SSH key generation. A static `ip host` entry can resolve a name without consulting DNS.

### Providing HTTP and HTTPS Services

Some management features depend on the IOS HTTP server. Plain HTTP sends data without encryption, so HTTPS should be used when the platform and management application support it:

```text
configure terminal
no ip http server
ip http authentication local
ip http secure-server
end
```

An HTTPS service also requires suitable credentials, keys or certificates, and access restrictions. Enabling the command alone does not create a secure management design. Restrict the service to a management network or VRF, use AAA where appropriate, and verify which web-based features actually require it.

### Providing DNS and File Transfer Services

An IOS router can act as a caching DNS server and answer for entries in its local host table on supported releases:

```text
configure terminal
ip dns server
ip name-server 192.0.2.53
ip host server1.lab.example 192.0.2.50
end
```

The router can also make a specific local file available through TFTP:

```text
tftp-server flash:router-image.bin alias router-image.bin
```

TFTP has no authentication or encryption. Use this feature only on an isolated or tightly controlled network and remove it after the transfer. IOS can also provide an SCP server with `ip scp server enable`, but SCP depends on a correct SSH configuration, user authentication, and access control.

### Legacy TCP Small Servers

The following command enables small test services such as Echo, Discard, and Chargen on supported platforms:

```text
service tcp-small-servers
```

These legacy services are disabled by default on current configurations and should normally remain disabled. They provide little operational value and can expose the control plane to reconnaissance or denial-of-service traffic. Confirm the secure state with:

```text
no service tcp-small-servers
no service udp-small-servers
```

On supported platforms, commands such as `show control-plane host open-ports` help identify services listening on the device. Compare the result with the intended management design and disable anything that is not required.

## Expected Outcomes

After completing this module, the reader should be able to:

- Distinguish the main architectural ideas behind classic IOS and IOS XE.
- Identify common storage locations and explain which files they contain.
- Explain how the configuration register can affect boot and recovery.
- Describe the IOS boot sequence and the purpose of ROMMON.
- Prepare and verify the essential steps of a software-image change.
- Identify the licensing information that must be checked for a device and release.
- Use an IOS device as a service-test client and explain the risks of enabling server functions on network infrastructure.

## Official References

- [Using the Cisco IOS Integrated File System](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ifs/configuration/15-s/ifs-15-s-book/ifs-using.html)
- [Troubleshooting password recovery on IOS and IOS XE routers](https://www.cisco.com/c/en/us/support/docs/ios-nx-os-software/ios-xe-16/217045-troubleshoot-password-recovery-in-cisco.html)
- [Loading and managing Cisco IOS system images](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sys-image-mgmt/configuration/15-mt/sysimgmgmt-15-mt-book/sysimgmgmt-rebooting.html)
- [Smart Licensing Using Policy for Cisco enterprise routing platforms](https://www.cisco.com/c/en/us/td/docs/routers/sl_using_policy/b-sl-using-policy/info_about.html)
- [Cisco IOS DNS configuration](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipaddr_dns/configuration/15-sy/dns-15-sy-book/dns-config-dns.html)
- [Cisco IOS XE basic system management](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/syst-mgmt/b-system-management/m_bsm-basic-sys-manage-xe.html)
