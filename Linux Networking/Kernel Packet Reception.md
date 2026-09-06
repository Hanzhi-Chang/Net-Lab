# How Does Data Travel from a Network Interface Card to the Protocol Stack?

## Overview

From a Linux perspective, the five-layer network model can be roughly mapped to the following components:

- **Application layer:** User-space processes that run applications and protocols such as HTTP and FTP. They communicate with the kernel through the socket API.
- **Transport and network layers:** The Linux kernel networking stack processes protocols such as TCP, UDP, IPv4, and IPv6.
- **Data link layer:** The network device driver allows the kernel to communicate with the network interface card (NIC). The kernel also handles functions such as Ethernet frame processing.
- **Physical layer:** The NIC, cables, optical transceivers, and physical signals transmit the data.

In the **Linux kernel source tree**, the main networking components are located in different directories:

- Network device drivers are mainly stored in `drivers/net/`. Ethernet drivers are located in `drivers/net/ethernet/`; for example, Intel Ethernet drivers are in `drivers/net/ethernet/intel/`.
- Most protocol-stack code is stored in `net/`, with protocol-specific code in directories such as `net/ipv4/` and `net/ipv6/`.

When a NIC receives a frame, it stores the data in a receive buffer and notifies the CPU. Traditionally, this notification is delivered through a hardware interrupt. The CPU briefly pauses its current work and runs the driver's interrupt handler.

Network packet processing can take a relatively long time. Performing all of it inside the hardware interrupt handler would prevent the CPU from responding promptly to other events. Linux therefore divides receive processing into two stages:

- The hardware interrupt handler performs only urgent work, such as acknowledging the interrupt and scheduling further packet processing.
- Most remaining work is deferred. Modern Linux network drivers commonly use NAPI to poll packets from the NIC, after which the packets are processed by the networking stack in softirq context.

Under heavy load, pending network softirqs may also be processed by a per-CPU `ksoftirqd` kernel thread. Therefore, `ksoftirqd` can participate in packet processing, but it does not process every received packet.

A **hardware interrupt** is a signal sent by a hardware device, such as a NIC, to request the CPU's immediate attention. The CPU temporarily pauses its current task and runs the corresponding interrupt handler.

In this networking context, a **software interrupt (softirq)** is a kernel mechanism used to schedule deferred work. It is raised by kernel code rather than directly by a hardware device, allowing time-consuming packet processing to occur after the hardware interrupt handler has completed its urgent work.

## Packet Reception Path in the Linux Kernel

1. The NIC receives an Ethernet frame from the network.
2. Using Direct Memory Access (DMA), the NIC writes the frame data into a receive buffer in system memory. The buffer is associated with an entry in the NIC's receive ring.
3. The NIC sends a hardware interrupt to notify the CPU that packets have arrived. Some NICs use interrupt moderation and may generate one interrupt for multiple packets.
4. The CPU runs the driver's hardware interrupt handler. The handler acknowledges the interrupt and schedules NAPI processing, which raises the `NET_RX_SOFTIRQ` softirq.
5. The softirq runs the kernel's network receive processing and calls the NAPI `poll` function registered by the driver. Softirqs can run directly in softirq context; under heavy load, they may be handled by the per-CPU `ksoftirqd` thread.
6. The driver's `poll` function retrieves received packets from the receive ring and represents each packet with a socket buffer (`sk_buff`, usually called an `skb`). The `skb` contains packet metadata and refers to the packet data in memory.
7. The driver passes the `skb` to the kernel networking stack. The Ethernet, network, and transport layers process it in sequence—for example, Ethernet → IPv4 → TCP.
8. If the packet is accepted and belongs to a local application, its data is placed in the corresponding socket's receive queue. The application can then retrieve the data through a socket call such as `recv()` or `read()`.

## Kernel Preparation for Receiving Packets

The packet reception path depends on several components that the kernel prepares during system initialization. For example, the kernel creates the `ksoftirqd` threads, initializes the networking subsystem, loads the NIC driver, allocates receive queues, and registers the driver's interrupt and NAPI `poll` functions.

This section explains these preparations one at a time.

### Creating the `ksoftirqd` Kernel Threads

Linux creates one `ksoftirqd` kernel thread for each **logical CPU**, with names such as `ksoftirqd/0` and `ksoftirqd/1`. These threads process pending softirqs when the work cannot be completed immediately in normal softirq context, particularly when the system is under heavy load.

Not every softirq runs in `ksoftirqd`. A softirq may run directly after a hardware interrupt or at another suitable point in kernel execution. `ksoftirqd` acts as a fallback that prevents excessive softirq processing from occupying the CPU continuously.

The following is a simplified version of the relevant code. The exact implementation may differ between Linux kernel versions.

```C
// Simplified from kernel/softirq.c

static struct smp_hotplug_thread softirq_threads = {
    .store             = &ksoftirqd,
    .thread_should_run = ksoftirqd_should_run,
    .thread_fn         = run_ksoftirqd,
    .thread_comm       = "ksoftirqd/%u",
};

static __init int spawn_ksoftirqd(void)
{
    BUG_ON(smpboot_register_percpu_thread(&softirq_threads));
    return 0;
}

early_initcall(spawn_ksoftirqd);
```

You do not need to understand every detail of the C syntax at this stage. The important parts are:

- `struct smp_hotplug_thread` defines a group of related settings for a per-CPU kernel thread.
- `softirq_threads` is the variable that stores these settings. This block describes how the threads should behave; it does not create them by itself.
- A field beginning with a dot, such as `.thread_fn`, assigns a value to a specific field in the structure.
- `&ksoftirqd` means “the address of `ksoftirqd`.” The kernel uses this address to store and manage the per-CPU thread objects.
- `ksoftirqd_should_run` and `run_ksoftirqd` are function names. They are stored as function pointers so that the kernel knows which functions to call.
- `%u` in `"ksoftirqd/%u"` is replaced by the logical CPU number. For example, the thread for CPU 0 is named `ksoftirqd/0`.

The structure specifies two particularly important functions:

- `ksoftirqd_should_run`: checks whether the logical CPU has pending softirqs that need to be processed.
- `run_ksoftirqd`: processes those pending softirqs. They may include network receive softirqs as well as other types of softirq.

The initialization flow can be read as follows:

1. `early_initcall(spawn_ksoftirqd)` tells the kernel to call `spawn_ksoftirqd()` during an early stage of boot.
2. `spawn_ksoftirqd()` calls `smpboot_register_percpu_thread()` and passes it the address of `softirq_threads`.
3. The SMP boot subsystem uses these settings to create and manage one `ksoftirqd` thread for each logical CPU.
4. When a thread is scheduled, the kernel uses `ksoftirqd_should_run()` to determine whether work is pending. If so, `run_ksoftirqd()` processes it.

`BUG_ON(condition)` means that the kernel treats the situation as a serious internal error if `condition` is true. In this example, it checks whether registration of the per-CPU threads failed. You only need to recognize its general purpose; its internal implementation is not important for understanding packet reception.

### Network Subsystem Initialization

During system startup, the kernel must prepare the data structures and softirq handlers used by the networking subsystem. The main initialization function is `net_dev_init()`, which is defined in `net/core/dev.c`.

The initialization process can be summarized as follows:

1. `subsys_initcall(net_dev_init)` registers `net_dev_init()` to run during the subsystem initialization stage of kernel boot.
2. When the kernel reaches that stage, it calls `net_dev_init()`.
3. `net_dev_init()` initializes one `softnet_data` structure for each possible logical CPU.
4. It registers `net_tx_action()` as the handler for `NET_TX_SOFTIRQ` and `net_rx_action()` as the handler for `NET_RX_SOFTIRQ`.
5. `open_softirq()` stores these mappings in the kernel's `softirq_vec` table.

The following is a simplified version of the relevant code:

```c
// Simplified from net/core/dev.c

static int __init net_dev_init(void)
{
    int cpu;

    for_each_possible_cpu(cpu) {
        struct softnet_data *sd = &per_cpu(softnet_data, cpu);

        memset(sd, 0, sizeof(*sd));
        skb_queue_head_init(&sd->input_pkt_queue);
        skb_queue_head_init(&sd->process_queue);
        INIT_LIST_HEAD(&sd->poll_list);
    }

    open_softirq(NET_TX_SOFTIRQ, net_tx_action);
    open_softirq(NET_RX_SOFTIRQ, net_rx_action);

    return 0;
}

subsys_initcall(net_dev_init);
```

#### Initializing `softnet_data`

`softnet_data` stores per-CPU networking state. Using a separate structure for each logical CPU reduces the need for multiple CPUs to modify the same queues and lists.

The loop can be read as follows:

- `for_each_possible_cpu(cpu)` repeats the code once for every logical CPU that the system may use.
- `struct softnet_data *sd` declares `sd` as a pointer to a `softnet_data` structure.
- `per_cpu(softnet_data, cpu)` selects the `softnet_data` structure belonging to a specific CPU.
- `&` obtains the address of that structure and stores it in the pointer `sd`.
- `sd->poll_list` accesses the `poll_list` field through the pointer `sd`. The `->` operator is used when accessing a structure through a pointer.

The initialization functions prepare several fields:

- `memset(sd, 0, sizeof(*sd))` clears the structure by setting its memory to zero.
- `skb_queue_head_init()` initializes queues used to hold packets awaiting processing.
- `INIT_LIST_HEAD(&sd->poll_list)` initializes `poll_list` as an empty linked list.

`poll_list` does **not** wait for NIC drivers to register their `poll` functions. A driver registers a NAPI instance and its `poll` function separately. When that NAPI instance is scheduled to receive packets, it is added to the current CPU's `poll_list`. Later, `net_rx_action()` processes the scheduled NAPI instances in this list and calls their `poll` functions.

The relationship can be summarized as:

1. During driver initialization, the driver creates a NAPI instance and associates it with a `poll` function.
2. When packets arrive, the driver's interrupt handler schedules that NAPI instance.
3. The scheduled NAPI instance is placed on the CPU's `softnet_data.poll_list`.
4. `NET_RX_SOFTIRQ` runs `net_rx_action()`.
5. `net_rx_action()` takes NAPI instances from `poll_list` and calls their `poll` functions.

#### Registering the Network Softirq Handlers

The following statements register the transmit and receive softirq handlers:

```c
open_softirq(NET_TX_SOFTIRQ, net_tx_action);
open_softirq(NET_RX_SOFTIRQ, net_rx_action);
```

The implementation of `open_softirq()` is very short:

```c
// Simplified from kernel/softirq.c

void open_softirq(int nr, void (*action)(struct softirq_action *))
{
    softirq_vec[nr].action = action;
}
```

The function has two parameters:

- `nr` is the softirq number, such as `NET_RX_SOFTIRQ`.
- `action` is a pointer to the handler function that should process this type of softirq. The syntax `void (*action)(struct softirq_action *)` means that `action` points to a function that returns no value (`void`) and receives a pointer to a `softirq_action` structure.

The statement inside the function can be read from right to left:

```c
softirq_vec[nr].action = action;
```

It stores the handler function received through the `action` parameter in the `action` field of entry `nr` in the `softirq_vec` array. In simpler terms, it creates the following mapping:

```c
softirq number → handler function
```

For example, calling `open_softirq(NET_RX_SOFTIRQ, net_rx_action)` stores `net_rx_action` in the table entry for `NET_RX_SOFTIRQ`.

Therefore, the two registration calls establish these mappings:

- When `NET_TX_SOFTIRQ` is pending, run `net_tx_action()`.
- When `NET_RX_SOFTIRQ` is pending, run `net_rx_action()`.

Internally, `open_softirq()` records each mapping in `softirq_vec`. You can think of `softirq_vec` as a table in which the softirq number is used to find the corresponding handler function.

When the kernel processes pending softirqs—either directly in softirq context or through `ksoftirqd`—the softirq dispatcher checks the pending softirq numbers, finds their handlers in `softirq_vec`, and calls the appropriate functions. There is no separate kernel component named `ksoftirq`; the relevant kernel thread is called `ksoftirqd`.

### Protocol Stack Registration

The Linux kernel implements IPv4 processing in the network layer and protocols such as TCP, UDP, and ICMP above it. Some important receive functions are `ip_rcv()`, `tcp_v4_rcv()`, and `udp_rcv()`.

These functions contain the actual packet-processing logic. Registration does not implement the protocols; instead, it tells the kernel **which function should be called for each protocol value**.

Packet delivery uses two main registration tables:

| Header field                           | Registration structure | Registered entry | Handler function |
| -------------------------------------- | ---------------------- | ---------------- | ---------------- |
| Ethernet EtherType = IPv4 (`ETH_P_IP`) | `ptype_base`           | `ip_packet_type` | `ip_rcv()`       |
| IPv4 Protocol = UDP (`IPPROTO_UDP`)    | `inet_protos`          | `udp_protocol`   | `udp_rcv()`      |
| IPv4 Protocol = TCP (`IPPROTO_TCP`)    | `inet_protos`          | `tcp_protocol`   | `tcp_v4_rcv()`   |

This creates a two-stage receive path:

```c
Ethernet frame
    └─ EtherType = IPv4 → ip_rcv()
                            └─ IP Protocol = TCP → tcp_v4_rcv()
                            └─ IP Protocol = UDP → udp_rcv()
                            └─ IP Protocol = ICMP → icmp_rcv()
```

#### Defining the Protocol Descriptors

The kernel first creates structures that describe the protocols and their handler functions:

```c
// Simplified from net/ipv4/af_inet.c

static struct packet_type ip_packet_type __read_mostly = {
    .type = cpu_to_be16(ETH_P_IP),
    .func = ip_rcv,
};

static const struct net_protocol udp_protocol = {
    .handler     = udp_rcv,
    .err_handler = udp_err,
    .no_policy   = 1,
    .netns_ok    = 1,
};

static const struct net_protocol tcp_protocol = {
    .early_demux = tcp_v4_early_demux,
    .handler     = tcp_v4_rcv,
    .err_handler = tcp_v4_err,
    .no_policy   = 1,
    .netns_ok    = 1,
};
```

These blocks create and initialize three structure variables:

- `ip_packet_type` describes how the kernel should handle an Ethernet frame whose EtherType is IPv4. Its `func` member points to `ip_rcv()`.
- `udp_protocol` describes IPv4 UDP processing. Its `handler` member points to `udp_rcv()`.
- `tcp_protocol` describes IPv4 TCP processing. Its `handler` member points to `tcp_v4_rcv()`.

Some C and kernel-specific syntax in this code has the following meaning:

- `struct packet_type` and `struct net_protocol` are structure types. `ip_packet_type`, `udp_protocol`, and `tcp_protocol` are structure variables.
- Members such as `.type` and `.handler` are initialized by name.
- `static` limits these variables to this source file.
- `const` means that the structure should not be modified after initialization.
- `__read_mostly` is a kernel annotation indicating that the variable is read frequently but rarely modified. It helps the kernel arrange data for better cache behavior on multiprocessor systems.
- `cpu_to_be16(ETH_P_IP)` converts the IPv4 EtherType to 16-bit big-endian byte order, which is the byte order used in Ethernet headers.

#### Running `inet_init()` During Kernel Startup

The protocol descriptors are registered by `inet_init()`:

```c
// Simplified from net/ipv4/af_inet.c

static int __init inet_init(void)
{
    inet_add_protocol(&icmp_protocol, IPPROTO_ICMP);
    inet_add_protocol(&udp_protocol,  IPPROTO_UDP);
    inet_add_protocol(&tcp_protocol,  IPPROTO_TCP);

    dev_add_pack(&ip_packet_type);

    return 0;
}

fs_initcall(inet_init);
```

`fs_initcall(inet_init)` registers `inet_init()` to run at the filesystem initialization level of kernel startup. Despite the name, this initcall level is also used by some non-filesystem subsystems. The macro does not immediately call `inet_init()` at the line where it appears.

When the kernel later runs `inet_init()`:

1. `inet_add_protocol()` registers ICMP, UDP, and TCP for their respective IPv4 Protocol numbers.
2. `dev_add_pack()` registers IPv4 for the Ethernet EtherType `ETH_P_IP`.

The `&` operator is required because these registration functions receive pointers to the protocol structures. For example, `&udp_protocol` is the address of the `udp_protocol` structure.

#### Registering TCP and UDP in `inet_protos`

The following is a simplified view of `inet_add_protocol()`:

```c
// Simplified from net/ipv4/protocol.c

int inet_add_protocol(const struct net_protocol *prot,
                      unsigned char protocol)
{
    if (!prot->netns_ok)
        return -EINVAL;

    return !cmpxchg((const struct net_protocol **)&inet_protos[protocol],
                    NULL, prot) ? 0 : -1;
}
```

The important parts of this function are:

- `prot` points to a `net_protocol` structure such as `udp_protocol`.
- `protocol` is the corresponding number, such as `IPPROTO_UDP`.
- `prot->netns_ok` accesses the `netns_ok` member through the structure pointer.
- `!prot->netns_ok` means “if `netns_ok` is false.”
- `cmpxchg(location, old, new)` means: if the value at `location` is still `old`, replace it with `new`. Here, it stores `prot` only when the table entry is still `NULL`.
- The conditional expression `condition ? 0 : -1` returns `0` when registration succeeds and `-1` when the entry is already occupied.

`cmpxchg()` performs the comparison and replacement atomically, making the update safe when multiple CPUs may access the table. Its lower-level implementation is not necessary for understanding the registration process.

Most importantly, `inet_protos` does **not** directly store the address of `udp_rcv()` or `tcp_v4_rcv()`. It stores pointers to `net_protocol` structures, and the `handler` member inside each structure contains the receive-function pointer:

```c
inet_protos[IPPROTO_UDP]
    → &udp_protocol
        → udp_protocol.handler
            → udp_rcv()
```

#### Registering IPv4 in `ptype_base`

`dev_add_pack()` registers a `packet_type` structure for an Ethernet protocol. The relevant logic can be simplified as follows:

```c
// Simplified from net/core/dev.c

void dev_add_pack(struct packet_type *pt)
{
    struct list_head *head = ptype_head(pt);

    /* Add pt to the linked list selected by ptype_head(). */
}

static inline struct list_head *ptype_head(const struct packet_type *pt)
{
    if (pt->type == htons(ETH_P_ALL))
        return &ptype_all;

    return &ptype_base[ntohs(pt->type) & PTYPE_HASH_MASK];
}
```

This code can be read as follows:

- `pt` points to a `packet_type` structure such as `ip_packet_type`.
- `ptype_head(pt)` selects the linked-list head where that structure should be stored.
- `ETH_P_ALL` uses the special `ptype_all` list because it matches every Ethernet protocol.
- Other EtherTypes are converted to host byte order, passed through a hash calculation, and placed in one of the `ptype_base` buckets.
- `static inline` means that the function is only visible in this source file and suggests that the compiler substitute its code at the call site when appropriate.

`ptype_base` is therefore not a simple array of handler-function addresses. It is a hash table whose buckets contain linked lists of `packet_type` structures. The `func` member inside `ip_packet_type` points to `ip_rcv()`:

```c
ptype_base[IPv4 hash bucket]
    → ip_packet_type
        → ip_packet_type.func
            → ip_rcv()
```

The complete registration relationship is:

```c
EtherType registration:
ETH_P_IP → ptype_base → ip_packet_type.func → ip_rcv()

IPv4 protocol registration:
IPPROTO_UDP → inet_protos → udp_protocol.handler → udp_rcv()
IPPROTO_TCP → inet_protos → tcp_protocol.handler → tcp_v4_rcv()
```

## Appendix: C Concepts Used in the Examples

Reading Linux kernel source code requires more than knowing basic C syntax. Kernel code frequently uses pointers, structures, function pointers, callbacks, and registration mechanisms. These techniques allow different kernel components and hardware drivers to work together without being directly dependent on one another.

This section introduces the concepts needed to understand the code examples in this note.

### What Is a Pointer Variable in C?

A pointer variable stores the memory address of another object.

```
int value = 10;
int *p = &value;
```

This code can be read as follows:

- `value` is an `int` variable whose value is `10`.
- `&value` obtains the memory address of `value`.
- `p` stores that address, so `p` points to `value`.
- `*p` accesses the integer stored at that address. In this example, `*p` is `10`.
- `&p` is the address of the pointer variable `p` itself.

The `*` symbol has different meanings depending on where it appears:

```
int *p;        // Declaration: p is a pointer to an int.
int n = *p;    // Expression: read the int stored at the address in p.
```

When a pointer refers to a structure, the `->` operator accesses a member of that structure:

```
struct device *dev;
dev->name;
```

`dev->name` is equivalent to `(*dev).name`: first access the structure pointed to by `dev`, and then access its `name` member.

Pointers are widely used in the Linux kernel because kernel functions often need to share or modify existing data structures without copying all their contents.

### What Is a Function Pointer?

Ordinary pointers store the addresses of data. A **function pointer** stores the address of a function.

Suppose the following function has been defined:

```
void print_number(int n)
{
    /* Print n. */
}
```

A pointer to this function can be declared as follows:

```
void (*handler)(int) = print_number;
```

This declaration means:

- `handler` is a pointer.
- It points to a function that receives an `int` argument.
- That function returns no value (`void`).

The function can then be called through the pointer:

```
handler(10);
```

This is equivalent to calling `print_number(10)` in this example.

Function-pointer declarations are easier to read from the variable name outward. For example:

```
void (*action)(struct softirq_action *)
```

Read it as: `action` is a pointer to a function that receives a pointer to a `softirq_action` structure and returns `void`.

Function pointers allow the kernel to store behavior as data. Instead of hard-coding one function call, the kernel can select and call different functions at runtime.

### What Does “Register” Mean in Linux Source Code?

In this context, **register** is a verb meaning “tell a subsystem about an object or function and store it for later use.” It does not refer to a CPU register or the old C `register` keyword.

Registration usually follows this pattern:

1. A component creates a data structure or function.
2. It passes a pointer to a kernel subsystem.
3. The subsystem stores that pointer in a table, list, or another data structure.
4. When a relevant event occurs later, the subsystem finds the stored pointer and uses the object or calls the function.

For example:

```
open_softirq(NET_RX_SOFTIRQ, net_rx_action);
```

This does not immediately call `net_rx_action()`. Instead, it registers `net_rx_action` as the handler for `NET_RX_SOFTIRQ`. The kernel stores the function pointer and calls it later when that softirq becomes pending.

A function that is registered and later called by another component is commonly called a **callback function**.

Direct calling and registration serve different purposes:

```
net_rx_action(...);                       // Call the function now.
open_softirq(NET_RX_SOFTIRQ, net_rx_action); // Store it so the kernel can call it later.
```

Registration is useful because the component that handles an event does not need to know in advance which driver or function will provide the implementation. For example, the networking core can provide a common NAPI framework, while each NIC driver registers its own `poll` function.

The same general pattern appears throughout the Linux kernel:

```
provider registers a function → kernel stores its address → event occurs → kernel calls the function
```

When reading kernel source code, a function whose name contains `register` usually establishes a relationship for future use rather than performing the final operation immediately. Names such as `add`, `init`, and `open` may do something similar, but you must inspect their implementation to confirm their behavior.
