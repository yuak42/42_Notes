These special addresses are recorded by IANA in **IANA IPv4 Special-Purpose Address Registry**.

These addresses are reserved for specific purposes
Some addresses cannot be used for internet communication

To do later
`0.0.0.0/0`
is there any address like `0.0.0.1/1`

# Conventions
- `127.0.0.0/8` actually means any IP address in form of `127.x.x.x`
- `0.0.0.0/0` -> means whole address space also **default route**.
# Loopback
`127.0.0.0/8` -> loopback
- Subnet mask is not relevant. Any IP address under this IP is a loopback address.

**Example:**
`127.0.0.1/8` -> Loopback
`127.10.20.30/24` -> Loopback
`127.100.50.25/16` -> Loopback
`127.255.255.254/32` -> Loopback
`128.0.0.0/8` -> Not Loopback

# Current Network
`0.0.0.0/8` -> this network

# Private-Use
`10.0.0.0 – 10.255.255.255` --- `10.0.0.0/8` -> `10.x.x.x`
`172.16.0.0 – 172.31.255.255` -/- `172.16.0.0/12` -> Notice not `172.16.x.x` !
`192.168.0.0 – 192.168.255.255` --- `192.168.0.0/16` -> `192.168.x.x`

Can be used for subnetting just like public IP addresses. However, they cannot be used for routing *on the internet* because these addresses are not unique.
As an experiment you can connect some private networks using a router. However, ISPs does not let private addresses to be routed because they are used on most of the subnets.

[RFC 1918](https://www.rfc-editor.org/info/rfc1918/)

# Link-local
`169.254.0.0/16`
Used for **automatic IP assignment** when DHCP is unavailable (APIPA).
- Allows devices on the same local link to communicate without a DHCP server.
- **Cannot be routed** beyond the local link.
**Example:** Two PCs connected directly can communicate using `169.254.1.10/16` and `169.254.1.20/16`.

# Multicast
`224.0.0.0/4`
Used for **one-to-many communication**, sending data to a specific group of devices instead of everyone.
- Devices join a multicast group to receive its traffic.
- Can cross routers if multicast routing is configured (except link-local multicast groups).
**Example:** A video server sends one stream to `239.1.1.1`, and all devices subscribed to that group receive it.


loopback adresleri iletişim için kullanılabilir mi?