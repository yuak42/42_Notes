Remember, private addresses cannot be routed **on the public internet**. Then how can I have a private address and still communicate on the internet? The answer is **Network Address Translation (NAT)**.

NAT allows a router to translate private IP addresses into a public IP address when communicating with the internet.

**Example:**

- My PC: `192.168.1.10` (Private IP)
    
- My router: `203.0.113.5` (Example Public IP)
    
- Destination: `8.8.8.8`
    

When my PC sends a packet to `8.8.8.8`, the router replaces its private source IP with the router's public IP. When the response arrives, the router uses its NAT translation table to forward it back to my PC.

**Key takeaway:** NAT allows multiple devices with private IPs to share a single public IP address, typically using different port numbers (PAT).
## NAT vs. PAT

**NAT (Network Address Translation)** translates IP addresses between private and public networks.

**PAT (Port Address Translation)** is a type of NAT that translates both IP addresses and port numbers, allowing multiple devices to share a single public IP.

**Example:**

Two devices connect to the same web server:

```
Private Network             Public Network

192.168.1.10:5000 ──┐
                    ├── NAT/PAT ──► 203.0.113.5:40001
192.168.1.11:5000 ──┘          └──► 203.0.113.5:40002
```

The router maintains a translation table to map incoming responses back to the correct device.

**Key takeaway:**

- **NAT:** Translates IP addresses.
    
- **PAT:** Translates IP addresses and ports, allowing many devices to share one public IP.
    
- **Relationship:** PAT is a specific form of NAT, also called _NAT overload_ or _NAPT_.
    

> Most home routers use PAT when connecting multiple private devices to the internet.