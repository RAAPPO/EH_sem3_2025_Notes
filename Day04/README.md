# Day 4: Networks – In-depth Notes

---

## 1. What is a Network?

A **network** is a collection of devices (computers, servers, phones, IoT devices) connected together to share data and resources. Networks can use **wired** (Ethernet cables) or **wireless** (Wi-Fi, Bluetooth) connections.

<div align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/6/60/NetworkTopologies.png" width="600" alt="Network Topologies Diagram"/>
  <br/><sup><i>Common network topologies: bus, star, ring, mesh, hybrid</i></sup>
</div>

**Key Points:**
- Enables sharing files, internet, printers, etc.
- Reduces costs by sharing resources.
- Can be private (home, office) or public (Internet).

### Types of Networks
| Type | Full Form | Scale | Example |
|------|-----------|-------|---------|
| **LAN** | Local Area Network | Single building/campus | Home, School |
| **WAN** | Wide Area Network | Regional/global | The Internet |
| **MAN** | Metropolitan Area Network | City/town | City-wide Wi-Fi |
| **PAN** | Personal Area Network | Few meters | Bluetooth between phone and earbuds |

### Common Network Devices
- **Switches:** Connect devices within LAN, forwards data based on MAC address.
- **Routers:** Connects different networks (e.g. home to internet), handles IP addresses.
- **Hubs:** Basic device, broadcasts data to all devices (not used much now).
- **Access Points:** Allow wireless (Wi-Fi) connections.

---

## 2. Subnetting

A **subnet** (sub-network) divides a larger network into smaller, manageable segments. Used to:
- Reduce congestion by limiting broadcast traffic.
- Improve security by isolating groups (e.g., HR vs. IT).
- Simplify network management.

<div align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/5/57/Network_subnetting.png" width="400" alt="Subnetting Example"/>
  <br/><sup><i>Subnetting an IP range</i></sup>
</div>

---

## 3. IP (Internet Protocol) Address

- **IP address**: Unique identifier for devices on a network.
- Determines where data is sent/received.
- Two main versions: IPv4 (most common), IPv6 (newer, for more devices).
- Can be **public** (internet-facing) or **private** (internal, like 192.168.x.x).

**CIDR Notation Example:**  
- `192.168.1.0/24` → 256 possible addresses (192.168.1.0 to 192.168.1.255)

---

## 4. MAC Address

- **MAC (Media Access Control) Address**: Unique, hardware-embedded address for network interfaces (e.g., Wi-Fi/Bluetooth/Ethernet adapter).
- Format: 6 pairs of hex digits, separated by `:` or `-` (e.g., `AA:BB:CC:DD:EE:FF`)
- Used at Data Link Layer (OSI Layer 2).
- The first 3 bytes identify the manufacturer (OUI).

---

## 5. IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Launched | 1981 | 1998 |
| Size | 32 bits | 128 bits |
| Address Space | 4.3 billion | 340 undecillion |
| Format | Decimal (e.g. 192.168.1.1) | Hexadecimal (e.g. 2001:0db8:85a3:0000:0000:8a2e:0370:7334) |
| Security | Optional | Built-in (IPsec) |
| Speed | Slower | More efficient routing |
| Example | 192.168.1.10 | 2001:db8::1 |

---

## 6. Static vs Dynamic IP Addresses

- **Static IP**: Does not change, set manually. Used for servers, printers, CCTV.
- **Dynamic IP**: Assigned automatically by DHCP. Changes each time device reconnects.

---

## 7. TCP and UDP

**TCP (Transmission Control Protocol):**
- Connection-based, reliable.
- Guarantees order and delivery.
- Used for: web browsing, email, file downloads.

**UDP (User Datagram Protocol):**
- Connectionless, fast, but not reliable.
- No guarantee of delivery/order.
- Used for: video streaming, gaming, VoIP.

|  | TCP | UDP |
|--|-----|-----|
| Connection | Yes | No |
| Reliability | Yes | No |
| Speed | Slower | Faster |
| Use case | Web, files | Streaming, gaming |

---

## 8. UAC (User Account Control)

- Windows security feature that prevents unauthorized changes.
- Prompts for permission/credentials when a program tries to change system settings.

---

## 9. Reconnaissance in Cybersecurity

The process of gathering information about targets, crucial for ethical hacking and penetration testing.

**Types:**
- **Active Reconnaissance:** Direct interaction (e.g., port scans). Easier to detect.
- **Passive Reconnaissance:** No direct interaction (e.g., searching public databases, using Wayback Machine). Harder to detect.

**Tools:**
- `whois`, `nslookup`, `theHarvester`, `Shodan`, `Google Dorking`
- Active recon can trigger **IDS (Intrusion Detection Systems)**

---

## 10. Nmap (Network Mapper)

- Powerful tool for network discovery and security auditing.

**Capabilities:**
- Discover live hosts on a network.
- Identify open ports and services.
- Detect OS and service versions.
- Run vulnerability scripts.

<div align="center">
  <img src="https://nmap.org/images/screenshots/nmap-gui.png" width="600" alt="Nmap Scan Example"/>
  <br/><sup><i>Nmap graphical output example</i></sup>
</div>

### Common Nmap Flags

| Flag         | Meaning                   | Purpose / When to Use                    |
| ------------ | ------------------------- | ---------------------------------------- |
| **-Pn**      | Skip host discovery       | Scan even if ping is blocked             |
| **-A**       | Aggressive scan           | Detect OS, versions, traceroute, scripts |
| **-sV**      | Service/version detection | Find service and version on ports        |
| **-O**       | OS detection              | Guess the target’s operating system      |
| **-oM**      | Machine-readable output   | Save results for automation              |
| **--script** | Run Nmap scripts (NSE)    | Check for vulnerabilities, extra info    |
| **-v / -vv** | Verbose / very verbose    | Show more details in scan output         |
| **-p**       | Specify port(s)           | Scan only chosen ports                   |
| **-sS**      | SYN (stealth) scan        | Faster, less detectable                  |
| **-sT**      | TCP connect scan          | Full handshake, reliable but noisy       |

**Example Command:**
```bash
nmap -A -p 22,80,443 192.168.1.1
```
- Scans ports 22, 80, 443 on host `192.168.1.1` with OS and service detection.

---

## Summary Table

| Term | Description |
|------|-------------|
| **Network** | Group of connected devices for data/resource sharing |
| **Subnet** | Division of a network for efficiency/security |
| **IP Address** | Unique identifier for networked device |
| **MAC Address** | Hardware address for network interface |
| **TCP/UDP** | Communication protocols (reliable/unreliable) |
| **Reconnaissance** | Information gathering phase in security |
| **Nmap** | Tool for network scanning and enumeration |

---

## Further Reading

- [Nmap Official Docs](https://nmap.org/book/man.html)
- [Subnetting Made Easy](https://www.geeksforgeeks.org/subnetting-in-computer-network/)
- [OSI Model Explained](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
- [IPv6 Explained](https://www.cloudflare.com/learning/network-layer/what-is-ipv6/)

---

> **Tip:** Practice network scanning in a safe, legal lab environment only! Always have permission before scanning networks.
