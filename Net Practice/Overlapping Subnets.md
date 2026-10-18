## 1. Example Scenario

Suppose we have the following IPv4 network:

```text
Network:       103.198.14.0/24
Subnet Mask:   255.255.255.0
Total IPs:     256
Usable Hosts:  254
Network:       103.198.14.0
Broadcast:     103.198.14.255
```

We decide to divide this network into **two smaller subnets** by borrowing one bit from the host portion.

The prefix length changes from `/24` to `/25`.

```text
Original Network
103.198.14.0/24
        |
        | Subnetting (borrow 1 bit)
        |
        +---------------------------+
        |                           |
        v                           v
103.198.14.0/25             103.198.14.128/25
128 addresses                128 addresses
126 usable hosts             126 usable hosts
```

### Subnet Details

| Property | Subnet A | Subnet B |
|---|---|---|
| Network Address | 103.198.14.0 | 103.198.14.128 |
| Prefix | /25 | /25 |
| Subnet Mask | 255.255.255.128 | 255.255.255.128 |
| First Usable IP | 103.198.14.1 | 103.198.14.129 |
| Last Usable IP | 103.198.14.126 | 103.198.14.254 |
| Broadcast | 103.198.14.127 | 103.198.14.255 |
| Usable Hosts | 126 | 126 |

Both subnets together cover exactly the same address range as the original `/24` network.

**Important:** Subnetting does not delete or invalidate the original network. It changes how we organize and use its address space.

---

## 2. What Happens to the Original /24 Network?

After dividing `103.198.14.0/24` into two `/25` subnets, the original `/24` prefix still exists as a valid representation of the entire address block.

However, if we want to operate the two `/25` subnets as separate networks, we should configure the devices with their corresponding `/25` masks.

For example:

```text
Subnet A: 103.198.14.0/25

PC1: 103.198.14.10/25
PC2: 103.198.14.100/25

Subnet B: 103.198.14.128/25

PC3: 103.198.14.150/25
PC4: 103.198.14.200/25
```

PC1 and PC2 belong to the same subnet, while PC3 and PC4 belong to another subnet.

If these are separate Layer 2 networks, communication between them requires routing.

### Can We Still Assign an IP Address With a /24 Mask?

**Yes, technically.**

For example:

```text
PC1: 103.198.14.10/24
```

This is a valid host IP configuration by itself.

However, it may conflict with our intended subnet design.

The important distinction is:

- An IP address identifies an interface.
- A subnet mask determines which addresses the device considers directly reachable on its local network.
- A network prefix in a routing table describes a destination address range.

These concepts are related, but they are not interchangeable.

---

## 3. Problems Caused by Incorrect Subnet Masks

Consider the following topology:

```text
       Subnet A                  Subnet B
    103.198.14.0/25          103.198.14.128/25

 PC1 -------- Router -------- PC2
 .10            |             .150
                |
          Routing between
          the two subnets
```

Suppose we accidentally configure PC1 with a `/24` mask instead of `/25`.

```text
PC1: 103.198.14.10/24
PC2: 103.198.14.150/25
```

### What Does PC1 Think?

PC1 believes that every address from `103.198.14.0` to `103.198.14.255` belongs to its directly connected network.

Therefore, when PC1 wants to send a packet to PC2 (`103.198.14.150`), it assumes that PC2 is on the same local network.

Instead of sending the packet to its default gateway, PC1 attempts to discover PC2's MAC address using ARP.

```text
PC1:
"Who has 103.198.14.150?"
```

But PC2 is located on another Layer 2 network, separated by a router.

Ordinary routers do not forward ARP broadcasts between networks.

As a result, PC1 cannot resolve PC2's MAC address, and communication fails.

### What Does PC2 Think?

PC2 uses the correct `/25` mask.

It recognizes that `103.198.14.10` is outside its local subnet and therefore sends traffic toward its configured gateway.

This can produce asymmetric forwarding behavior, where one device tries to communicate directly while the other uses a router.

**Key lesson:** An incorrect subnet mask can cause a device to incorrectly classify a remote destination as local.

> Note: Proxy ARP or specialized network configurations may allow communication in some situations, but these are exceptions rather than the normal behavior.

---

## 4. Can We Use /24 in Routing Tables?

**Yes! This is one of the most important distinctions in subnetting.**

Even though we divided the original network into two `/25` subnets, routers may still use the original `/24` prefix as a destination route.

This is possible through **route summarization**, also called **route aggregation**.

### Example: Route Summarization

Imagine Router A connects both subnets, and Router B needs to reach them.

```text
                          +-- 103.198.14.0/25
                          |
Router B ---- Router A ---+
                          |
                          +-- 103.198.14.128/25
```

Router B could have two separate routes:

```text
Destination          Next Hop
103.198.14.0/25      Router A
103.198.14.128/25    Router A
```

However, because these two networks are contiguous and share the same next hop, we can summarize them:

```text
Destination          Next Hop
103.198.14.0/24      Router A
```

Now Router B needs only one route instead of two.

Router A can still maintain the more specific `/25` routes to forward packets to the correct subnet.

### Why Does This Work?

Routers use a principle called **Longest Prefix Match (LPM)**.

When multiple routing entries match a destination IP, the router selects the matching route with the longest prefix.

For example:

```text
Destination          Next Hop
103.198.14.0/24      Router A
103.198.14.128/25    Router B
```

A packet destined for `103.198.14.150` matches both routes.

However:

- `/24` matches the first 24 bits.
- `/25` matches the first 25 bits.

The router selects the `/25` route because it is more specific.

**Important:** Route summarization is safe only when the summarizing router can correctly reach all destinations covered by the summary, or intentionally discard unreachable destinations. Otherwise, it can create routing loops or black holes.

---

## 5. What Happens If We Mix /24 and /25 Networks?

Consider these two interfaces:

```text
Interface A: 103.198.14.10/24
Interface B: 103.198.14.150/25
```

Their calculated networks are:

```text
Interface A:
103.198.14.0/24

Interface B:
103.198.14.128/25
```

These networks overlap.

The `/25` address range is entirely contained within the `/24` address range.

### Is Overlapping Always Invalid?

Not necessarily.

Overlapping prefixes are common in routing tables, especially when using route summarization and longest prefix matching.

However, overlapping IP subnets on different physical or logical interfaces can cause problems.

For example:

- Devices may incorrectly assume that remote hosts are local.
- ARP resolution may fail.
- Routing decisions may become ambiguous.
- Operating systems or routers may reject overlapping connected-interface configurations.
- Traffic may be sent through the wrong interface.

Therefore, in a conventional routed network, each Layer 3 subnet should have a clearly defined, non-overlapping address range.

**Remember:** Overlapping routes are not the same as overlapping connected subnets.

---

## 6. Can We Recombine the Subnets?

Yes.

The two `/25` networks can be represented by their original `/24` prefix.

```text
103.198.14.0/25
        +
103.198.14.128/25
        |
        v
103.198.14.0/24
```

This operation is called **supernetting** or **route aggregation**, depending on the context.

However, there are two different situations.

### Case A: Combining Routes

We replace two routing entries with one summarized route.

The physical network structure does not necessarily change.

```text
Before:
103.198.14.0/25
103.198.14.128/25

After:
103.198.14.0/24
```

Both subnets can remain separate behind the next-hop router.

### Case B: Combining Actual Networks

We want to turn two separate `/25` LANs into one `/24` LAN.

This requires more than changing the routing table.

We generally need to:

1. Place the devices in the same Layer 2 broadcast domain.
2. Reconfigure their subnet masks to `/24`.
3. Adjust gateway addresses and routing configurations.
4. Verify that no IP address conflicts exist.

Only then will the devices operate as members of one larger subnet.

## 7. Common Misconceptions

| Misconception | Reality |
|---|---|
| Subnetting destroys the original network. | The original address block still exists. |
| After subnetting, `/24` cannot be used anywhere. | `/24` can still be used for summarized routes or a redesigned network. |
| A device can never use a `/24` mask after the network has been divided. | It can, but this may conflict with the intended network topology. |
| Two different subnet masks always prevent communication. | Not always; connectivity depends on topology, routes, and address resolution. |
| Overlapping routes are always invalid. | Routers commonly use overlapping routes with longest prefix matching. |
| A subnet mask is only used to calculate available hosts. | It also determines local reachability and influences routing decisions. |
| Subnetting automatically creates separate physical networks. | Logical addressing and physical or Layer 2 separation must be configured appropriately. |

## 8. Final Summary

When we divide `103.198.14.0/24` into:

```text
103.198.14.0/25
103.198.14.128/25
```

We should understand the following principles:

1. **The original `/24` network is not deleted.** Its address space is simply divided into two smaller networks.

2. **Devices should use the subnet mask corresponding to their intended subnet.** Incorrect masks can cause devices to make incorrect local-versus-remote forwarding decisions.

3. **The `/24` prefix can still appear in routing tables.** It can summarize both `/25` subnets when they are reachable through the same next hop.

4. **Overlapping connected subnets can create problems.** They should generally be avoided in ordinary routed network designs.

5. **Routers use longest prefix matching.** A more specific route, such as `/25`, takes precedence over a matching `/24` route.

6. **Subnetting and routing are different operations.** Subnetting organizes address space, while routing determines how packets travel between networks.

### References

- [RFC 4632 — Classless Inter-domain Routing (CIDR)](https://www.rfc-editor.org/rfc/rfc4632)
- [RFC 1812 — Requirements for IP Version 4 Routers](https://www.rfc-editor.org/rfc/rfc1812)
- [RFC 826 — Address Resolution Protocol (ARP)](https://www.rfc-editor.org/rfc/rfc826)

**Core Principle:** Subnetting determines how an address space is divided. Interface configuration determines how devices interpret their local networks. Routing determines how packets reach other networks.