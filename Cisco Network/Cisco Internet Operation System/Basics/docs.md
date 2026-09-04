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

- The purpose of the IOS filesystem and filesystem prefixes.
- Storage locations and their roles:
  - Flash memory for system images and persistent files.
  - NVRAM for the startup configuration on platforms that use it.
  - ROM or bootstrap storage for bootstrap and recovery software.
  - SDRAM for the running system and running configuration.
- Opaque, network, disk, and NVRAM filesystem types.
- Read-only, read-write, and write-only file permissions.
- Access to local storage and external servers through IOS file operations.

## 3. Configuration Register

- The configuration register as a platform setting separate from the startup and running configurations.
- Its 16-bit hexadecimal representation.
- Boot-field selection, console speed, break behaviour, and recovery-related settings.
- Common legacy IOS examples such as the normal `0x2102` value and `0x2142` for temporarily ignoring the startup configuration during password recovery.

Configuration-register support and defaults are platform-dependent. Modern IOS XE devices may use different recovery and boot mechanisms.

## 4. IOS Boot Process

- Power-on self-test.
- Loading and running bootstrap software.
- Locating and loading a system image from flash memory.
- Entering ROMMON when a usable image cannot be loaded.
- Loading the startup configuration into the running configuration.
- The purpose of ROMMON for image recovery and password-recovery procedures.
- Image selection and the role of boot configuration.

## 5. Upgrading an IOS Image

- Transferring an image with TFTP, FTP, or SCP.
- Checking available flash space before copying an image.
- Backing up the existing image before a change.
- Understanding the Cisco device as the client when the `copy` operation uses an external file server.
- Verifying image integrity with an MD5 checksum where supported.
- Selecting the new image with a boot-system configuration.
- Confirming the image and boot settings before reloading the device.

An image change should be planned with a compatibility check, a verified backup, a maintenance window, console access, and a recovery path.

## 6. IOS Licensing

- The historical relationship between IOS images, feature sets, and separately licensed technology packages.
- Base functionality and additional feature activation.
- Device identification with UDI, PID, and serial-number information.
- Traditional PAK registration and `.lic` licence-file installation.
- Evaluation or trial licences described in the source material.

Cisco licensing has changed significantly across product families and software generations. The licensing model in use must be confirmed for the specific platform and release rather than inferred from a legacy IOS workflow.

## 7. Router as a Host or Service-Test Client

- Using an IOS device to test reachability to a TCP service.
- Using `telnet <ip-address> <port>` as a simple check that a destination TCP port can be reached and accepts a connection.
- Interpreting the result as a service-path test rather than proof that the complete application is healthy.

## Expected Outcomes

After completing this module, the reader should be able to:

- Distinguish the main architectural ideas behind classic IOS and IOS XE.
- Identify common storage locations and explain which files they contain.
- Explain how the configuration register can affect boot and recovery.
- Describe the IOS boot sequence and the purpose of ROMMON.
- Prepare and verify the essential steps of a software-image change.
- Identify the licensing information that must be checked for a device and release.
- Perform a basic TCP service-reachability test from the IOS CLI.
