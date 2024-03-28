# Routing Table - Getting Started

When handling any packet, the kernel must decide where to send that packet. In most cases, the destination machine will not be within the same network.

Routing is a means of sending an IP packet from one point to another.

**The route table serves this purpose by mapping known subnets to a gateway IP address and interface**.

## View Routing Table

To view the routing table in Linux there are a few ways to achieve this.

### Using route command

```bash
$ route -n

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.1        0.0.0.0         UG    303    0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     303    0        0 eth0
```

!!! info
    The `-n` option means that you want numerical IP addresses displayed, instead of the corresponding host names.

### Using netstat command

```bash
$ netstat -rn

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.1        0.0.0.0         UG    303    0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     303    0        0 eth0
```

!!! info
    The `-r` option specifies that you want the routing table. The `-n` option is similar to that of the route command.

!!! note
    netstat is deprecated and replaced with other tools, like `ip` and `ss`.

### Using ip command


```bash
ip route list
```

Even though the `ip` command is the recommended method of listing the routing table in Linux, the output doesn't look as nice as the other options.

## Flags

Following is the list of flags and their significance in the routing table :


| Flag  | Description |
|:-----:|:-----------:|
|  `U`  | Up — Route is valid |
|  `G`  | Gateway — Route is to a gateway router rather than to a directly connected network or host |
|  `H`  | Host name — Route is to a host rather than to a network, where the destination address is a complete address |
|  `R`  | Reject — Set by ARP when an entry expires (for example, the IP address could not be resolved into a MAC address) |
|  `D`  | Dynamic — Route added by a route redirect or RIP (if routed is enabled) |
|  `M`  | Modified — Route modified by a route redirect |
|  `C`  | A new route is cloned from this entry when it is used |
|  `L`  | Link—Link-level information, such as the Ethernet MAC address, is present |
|  `S`  | Static—Route added with the route command |


## Example

Let's have a look at the routing table mentioned above:

```bash
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.1        0.0.0.0         UG    303    0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     303    0        0 eth0
```

In the previous example, a request to `1.2.3.4` would be sent to `10.0.0.1`, on the `eth0` interface, because `1.2.3.4` is in the subnet described by the first rule (`0.0.0.0/0`) and not in the subnet described by the second rule (`10.0.0.0/24`). Subnets are specified by the destination and `genmask` values.

!!! info
    Linux prefers to route packets by specificity (how “small” a matching subnet is) and then by weight (“metric” in `route` output). Given our example, a packet addressed to `10.0.0.1` will always be sent to gateway `0.0.0.0` because that route matches a smaller set of addresses. If we had two routes with the same specificity, then the route with a lower metric wiould be preferred.

**Some CNI plugins make heavy use of the route table**.
