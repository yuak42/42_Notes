## What is longest prefix match?




## When do we need static routing?

Directly Connected vs. Longest Prefix Match

## Can an address like `0.0.0.1/1` exist?

**Yes!** IPv4 supports prefix lengths from `/0` to `/32`.

`0.0.0.1/1` belongs to the network `0.0.0.0/1`, which covers half of the IPv4 address space (`0.0.0.0 – 127.255.255.255`).

### Real-World Example: VPN Routing

Some VPNs use two routes instead of replacing the default route (`0.0.0.0/0`):

- `0.0.0.0/1` → First half of IPv4 addresses.
- `128.0.0.0/1` → Second half of IPv4 addresses.

**Why does this work?**

Routers use **Longest Prefix Match**: when multiple routes match a destination, the route with the longest prefix (most specific match) is preferred.

Since `/1` is more specific than `/0`, traffic follows the VPN routes instead of the default route, unless an even more specific route exists.

### Example: Longest Prefix Match

Suppose a router has the following routing table:

|Destination|Gateway|
|---|---|
|`0.0.0.0/0`|Normal Router|
|`0.0.0.0/1`|VPN|
|`128.0.0.0/1`|VPN|
|`8.8.8.0/24`|Special Router|

**Which route will be selected?**

- `1.1.1.1` → VPN (`0.0.0.0/1`)
- `142.250.1.1` → VPN (`128.0.0.0/1`)
- `8.8.8.8` → Special Router (`8.8.8.0/24`)

**Why?** `8.8.8.8` matches `/0`, `/1`, and `/24`, but `/24` wins because it has the **longest matching prefix**.

**Key takeaway:** Routing selects the most specific matching route, not simply the default gateway.