These special addresses are recorded by IANA in **IANA IPv4 Special-Purpose Address Registry**.

These addresses are reserved for specific purposes
Some addresses cannot be used for internet communication

To do later
`0.0.0.0/0`
is there any address like `0.0.0.1/1`

# Conventions
- `127.0.0.0/8` actually means any IP address in form of `127.x.x.x`

# Loopback
127.0.0.0/8 -> loopback (subnet mask is not relevant)

**Example:**
`127.0.0.1/8` -> Loopback
`127.10.20.30/24` -> Loopback
`127.100.50.25/16` -> Loopback
`127.255.255.254/32` -> Loopback
`128.0.0.0/8` -> Not Loopback

# Current Network
0.0.0.0/8 -> this network

# Private-Use
`10.0.0.0/8`
`172.16.0.0/12`
`192.168.0.0/16`

Can be used for subnetting just like public IP addresses.

[RFC 1918](https://www.rfc-editor.org/info/rfc1918/)
