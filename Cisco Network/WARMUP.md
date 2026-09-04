# Warmup and Conventions

This guide introduces the Cisco command-line interface and defines the command conventions used throughout the Cisco Network section of Net-Lab. Read it before using the technology documents, labs, or configuration examples.

Some of these concepts may not be immediately clear or easy to remember all at once. Feel free to return to this guide whenever you need a refresher.

## 1. What Is a CLI?

A command-line interface (CLI) is a text-based interface used to configure, operate, verify, and troubleshoot a device. Instead of selecting options from a graphical interface, an engineer enters commands at a prompt and reads the resulting output.

Cisco IOS and IOS XE use a hierarchical CLI. The commands available at any moment depend on the current operating mode and privilege level. The prompt indicates that context:

```text
Router>            User EXEC mode
Router#            Privileged EXEC mode
Router(config)#    Global configuration mode
Router(config-if)# Interface configuration mode
```

The prompt itself is not part of the command and should not be typed or pasted when reproducing an example.

## 2. Cisco Command-Syntax Conventions

Cisco documentation uses typographic symbols to distinguish literal keywords from values supplied by the user.

| Convention | Meaning | Example |
| --- | --- | --- |
| **Bold** | A command or keyword that must be entered exactly as shown. | **show ip route** |
| *Italic* | A parameter that must be replaced with a value appropriate to the device or lab. | *interface-id* |
| `{ }` | A required element or a required choice. Select one of the enclosed alternatives. | `{ active or standby }` |
| `[ ]` | An optional element. The command can be entered with or without it. | `[ detail ]` |
| `|` | Separates mutually exclusive alternatives in command syntax. Select one alternative. | `active | standby` |

For example:

```text
show ip route [prefix]
transport input {ssh | telnet | all | none}
```

In a syntax definition, the braces, brackets, and vertical bar normally explain how to construct the command; they are not entered literally. Replace a parameter such as *prefix* or *interface-id* with a real value.

> [!IMPORTANT]
> The vertical bar has two different roles. In a command-syntax definition, it separates alternative choices and is not typed. In an operational command such as `show running-config | include hostname`, it is an actual pipe character that must be typed.

## 3. Context-Sensitive Help with `?`

The question mark provides help for the current CLI context without executing the command.

### List Commands Available in the Current Mode

```text
Router# ?
```

### List Keywords or Parameters That Can Come Next

Enter a space before `?` to see what can follow the current command:

```text
Router# show ?
Router# show ip ?
Router(config)# interface ?
```

### Complete or Check a Partial Keyword

Enter `?` immediately after the characters, without a space, to display matching keywords:

```text
Router# show run?
running-config
```

If the CLI displays `% Incomplete command`, the command is valid so far but requires another keyword or parameter. If it displays `% Invalid input detected`, check the current mode, spelling, command availability, and the position marked by `^`.

## 4. Command Completion with the Tab Key

Press **Tab** after entering enough characters to uniquely identify a command or keyword:

```text
Router# show run<Tab>
Router# show running-config
```

`<Tab>` represents pressing the Tab key; do not type the characters shown inside angle brackets. If the text does not complete, the abbreviation may still be ambiguous. Enter more characters and press Tab again, or use `?` to display the available matches.

Cisco IOS usually accepts an unambiguous abbreviated command, but this project normally uses complete command names in documentation and saved configurations so that the intended operation remains clear.

## 5. Cisco CLI Hierarchy

### User EXEC Mode

The prompt ends with `>`. This mode provides limited monitoring and connectivity commands.

```text
Router>
```

Use `enable` to enter privileged EXEC mode.

### Privileged EXEC Mode

The prompt ends with `#`. This mode provides broader verification, maintenance, debugging, file-management, and device-control commands.

```text
Router#
```

Use `configure terminal` to enter global configuration mode. Use `disable` to return to user EXEC mode.

### Global Configuration Mode

The prompt contains `(config)#`. Commands entered here change the running configuration and can also open more specific configuration modes.

```text
Router(config)#
```

Common subconfiguration modes include:

| Mode | Example prompt | Entered with |
| --- | --- | --- |
| Interface configuration | `Router(config-if)#` | `interface interface-id` |
| Routing-process configuration | `Router(config-router)#` | A routing-process command such as `router ospf process-id` |
| Line configuration | `Router(config-line)#` | `line console 0` or `line vty first-line last-line` |

Use `exit` to return one level, or use `end` or **Ctrl+Z** to return directly to privileged EXEC mode.

### Running EXEC Commands with `do`

While working in global configuration mode or a subconfiguration mode, prefix an EXEC command with `do` to run it without leaving the current configuration mode:

```text
Router(config)# do show ip interface brief
Router(config-if)# do show running-config | include hostname
```

`do` is useful for checking operational state while configuring a device. It does not change the current mode, and the `do` command itself is not stored in the running configuration.

## 6. Mode Conventions Used in This Project

To keep command examples concise and consistent, this project uses short mode labels. A line beginning with `!` is a documentation label that identifies the required configuration mode; it is not a command to enter on the device.

### EXEC Mode: No Label

Commands without a mode label are executed in **EXEC mode**. Most `show` examples assume privileged EXEC mode:

```text
show running-config
```

### Global Configuration Mode: `! global`

Commands entered directly from `Router(config)#` are preceded by the `! global` label:

```text
! global
ip multicast-routing
```

### Specific Configuration Modes: Use the Mode Name

Commands entered in a specific subconfiguration mode are preceded by a label such as `! interface`, `! router ospf`, or `! line vty`:

```text
! interface
ip address 192.0.2.1 255.255.255.0
```

When a Cisco prompt is included instead of a project mode label, the prompt shows the required mode and is not part of the command to be entered.

## 7. Filtering `show` Command Output

Many `show` commands generate more output than is needed. Cisco IOS and IOS XE can pass the output through a pipe and display only the relevant portion.

### `| begin` or `| b`

Displays output beginning with the first line that matches the expression and continues to the end:

```text
show running-config | begin router ospf
show running-config | b router ospf
```

Use it when the required information appears near the end of long output and the following lines are also relevant.

### `| include` or `| i`

Displays only lines that match the expression:

```text
show ip interface brief | include up
show ip interface brief | i up
```

Use it to locate repeated status values, addresses, interface names, or configuration statements.

### `| exclude` or `| e`

Displays every line except those that match the expression:

```text
show ip interface brief | exclude down
show ip interface brief | e down
```

Use it to remove repetitive or irrelevant lines while retaining the rest of the command output.

### `| section` or `| s`

Displays the complete section associated with a matching heading or configuration line:

```text
show running-config | section router ospf
show running-config | s router ospf
```

Use it when the matching line and its related indented configuration must be read together.

The expression used by `begin`, `include`, `exclude`, and `section` can be a simple word or a regular expression. Matching is normally case-sensitive. Pipe options, abbreviations, and regular-expression behaviour can vary between platforms and releases; append `| ?` to a specific command, such as `show running-config | ?`, to confirm what the current device supports.

## Next Step

After completing this warmup, continue with [Cisco Internetwork Operating System (IOS)](./Cisco%20Internet%20Operation%20System/) to learn the system-management foundations used throughout later labs.
