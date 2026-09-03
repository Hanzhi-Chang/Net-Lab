# Cisco IOS Basics

This module introduces the operating-system, storage, boot, software image, and licensing foundations of Cisco IOS and IOS XE devices. It corresponds specifically to the **Cisco IOS Basics** section of the system-management material.

## 1. Cisco IOS and IOS XE

- The classic IOS software architecture and the relationship between the operating system, processes, memory, and CPU resources.
- IOS XE as a Linux-based system in which IOS services run as processes.
- Modular IOS XE software packaging and the role of platform packages described in the source material:
  - `RPBase` for route-processor operating-system functions.
  - `RPControl` for control-plane interaction between IOS and the platform.
  - `RPAccess` for access services such as SSH and SSL.
  - `RPIOS` for the Cisco IOS software component.
  - `ESPBase` for Embedded Services Processor functions and data-plane services.
  - `SIPBase` for SPA Interface Processor operating-system and control functions.
  - `SIPSPA` for SPA drivers and field-programmable-device support.
- The consolidated package that contains the complete set of software packages.

Package names and installation models vary across IOS XE platforms and releases, so detailed procedures must be checked against the documentation for the device being used.

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
