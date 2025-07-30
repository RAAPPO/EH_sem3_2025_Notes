# Core Network Terms: Complete Guide to NAT, ARP, MAC, IPv4, and IPv6

## Table of Contents
1. [NAT (Network Address Translation)](#nat-network-address-translation)
2. [ARP (Address Resolution Protocol)](#arp-address-resolution-protocol)
3. [MAC (Media Access Control) Address](#mac-media-access-control-address)
4. [IPv4 (Internet Protocol Version 4)](#ipv4-internet-protocol-version-4)
5. [IPv6 (Internet Protocol Version 6)](#ipv6-internet-protocol-version-6)

---

## NAT (Network Address Translation)

### What is NAT?
**One-line explanation:** NAT translates private internal addresses to public internet addresses so multiple devices can share one internet connection.

### Real-World Analogy
NAT works like a **company reception desk**:
- Company has one main phone number (public IP: 203.0.113.5)
- Employees have extensions 101, 102, 103 (private IPs: 192.168.1.x)
- Receptionist connects external calls to right extension
- For outgoing calls, receptionist gives company's main number
- Outside world only knows the main number

### How NAT Works - Technical Process

```
Your Home Network Setup:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Laptop    │    │   Phone     │    │   Smart TV  │
│192.168.1.10 │    │192.168.1.11 │    │192.168.1.12 │
└─────────────┘    └─────────────┘    └─────────────┘
       │                  │                  │
       └──────────────────┼──────────────────┘
                          │
                  ┌─────────────┐
                  │ NAT Router  │ ← Does the translation magic
                  │203.0.113.5  │ ← Your public IP from ISP
                  └─────────────┘
                          │
                    Internet Cloud
```

### NAT Translation Table - How Router Remembers

```
Active NAT Sessions (Router's Memory):
┌─────────────────┬─────────────────┬──────────┬─────────────────────┐
│ Internal Device │ External View   │ Protocol │ Destination         │
├─────────────────┼─────────────────┼──────────┼─────────────────────┤
│192.168.1.10:3000│203.0.113.5:5000│ TCP      │ Facebook.com:443    │
│192.168.1.11:4000│203.0.113.5:5001│ TCP      │ Instagram.com:443   │
│192.168.1.12:5000│203.0.113.5:5002│ UDP      │ Netflix.com:80      │
│192.168.1.10:3001│203.0.113.5:5003│ TCP      │ Gmail.com:443       │
└─────────────────┴─────────────────┴──────────┴─────────────────────┘

How it works:
- Router assigns unique external port (5000, 5001, 5002) to each session
- Remembers which internal device each external port belongs to
- When response comes back, router forwards to correct internal device
```

### Types of NAT with Real Examples

**1. Static NAT (One-to-One)**
```
Use Case: Company web server that needs fixed external access

Configuration:
External IP 203.0.113.10 ← Always maps to → Internal IP 192.168.1.100

Why needed:
- Web server must be reachable from internet
- DNS points to 203.0.113.10
- Customers expect consistent access
- No port confusion

Limitation: Need separate public IP for each server (expensive)
```

**2. Dynamic NAT (Pool-Based)**
```
Use Case: Company with 50 employees, 10 public IPs

IP Pool: 203.0.113.10 through 203.0.113.19 (10 addresses)
Internal Network: 192.168.1.1 through 192.168.1.50 (50 devices)

How it works:
- First 10 employees to access internet get public IPs
- Employee #11 must wait until someone else finishes
- When session ends, public IP returns to pool

Problem: Not enough public IPs for everyone simultaneously
```

**3. PAT/NAT Overload (Most Common)**
```
Use Case: Home network, small business (what your router does)

One Public IP: 203.0.113.5
Many Internal Devices: Up to 65,000+ possible connections

Magic: Router uses port numbers to track everything
- Device A's Facebook → External port 5000
- Device B's YouTube → External port 5001  
- Device A's Gmail → External port 5002
- Device C's Netflix → External port 5003

Advantage: Unlimited internal devices with one public IP
```

### NAT Process Step-by-Step Example

**Scenario: You browse YouTube from your laptop**

```
Step 1: Your laptop creates request
┌─────────────────────────────────────────────────────┐
│ FROM: 192.168.1.10:3000 (your laptop)              │
│ TO: 208.65.153.238:443 (YouTube server)            │
│ DATA: "GET /watch?v=dQw4w9WgXcQ HTTP/1.1"          │
└─────────────────────────────────────────────────────┘

Step 2: Router intercepts and modifies
Router thinks: "192.168.1.10 won't work on internet, I'll change it"

┌─────────────────────────────────────────────────────┐
│ FROM: 203.0.113.5:5000 (router's public IP:port)   │ ← Changed
│ TO: 208.65.153.238:443 (YouTube server)            │ ← Unchanged  
│ DATA: "GET /watch?v=dQw4w9WgXcQ HTTP/1.1"          │ ← Unchanged
└─────────────────────────────────────────────────────┘

Router remembers: "Port 5000 belongs to 192.168.1.10:3000"

Step 3: YouTube responds
┌─────────────────────────────────────────────────────┐
│ FROM: 208.65.153.238:443 (YouTube server)          │
│ TO: 203.0.113.5:5000 (router's public IP:port)     │
│ DATA: Video stream data                             │
└─────────────────────────────────────────────────────┘

Step 4: Router translates back and forwards
Router checks: "Port 5000 belongs to 192.168.1.10:3000"

┌─────────────────────────────────────────────────────┐
│ FROM: 208.65.153.238:443 (YouTube server)          │ ← Unchanged
│ TO: 192.168.1.10:3000 (your laptop)                │ ← Changed back
│ DATA: Video stream data                             │ ← Unchanged
└─────────────────────────────────────────────────────┘

Your laptop receives YouTube video!
```

### NAT Limitations and Solutions

**Problem 1: Incoming Connections**
```
Issue: External devices can't initiate connections to internal devices
Example: You can't run a web server accessible from internet

Solution - Port Forwarding:
Router configuration: "Forward external port 8080 to 192.168.1.100:80"
Result: Internet traffic to 203.0.113.5:8080 → Your server at 192.168.1.100:80
```

**Problem 2: P2P Applications (Gaming, File Sharing)**
```
Issue: Two devices behind NAT can't connect directly to each other

Workaround - UPnP (Universal Plug and Play):
1. Application asks router: "Please open port 12345 for me"
2. Router automatically creates port forwarding rule
3. Application tells friend: "Connect to my-public-ip:12345"
4. Friend can now connect directly

Advanced Solution - STUN/TURN servers:
- STUN helps discover your public IP and port
- TURN relays data when direct connection impossible
```

---

## ARP (Address Resolution Protocol)

### What is ARP?
**One-line explanation:** ARP finds a device's MAC address when you only know its IP address.

### Real-World Analogy
ARP is like **asking for someone's apartment number when you only know their name**:
- You know "John Smith" lives in the building (IP address)
- But you need apartment #23B to deliver a package (MAC address)
- You ask over building intercom: "John Smith, what's your apartment number?"
- John responds: "I'm in apartment 23B"
- Now you can deliver the package directly

### Why ARP is Needed

```
The Problem:
Your computer knows: "I want to send data to 192.168.1.20"
But Ethernet requires: Physical MAC address for actual delivery

Think of it like postal mail:
- IP Address = "John Smith, Springfield" (logical address)
- MAC Address = "123 Main Street, Apt 4B" (physical address)
- You need both to deliver the package
```

### ARP Process - Detailed Example

**Network Setup:**
```
Office Network 192.168.1.0/24:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Your Laptop │    │  Printer    │    │File Server  │
│192.168.1.10 │    │192.168.1.50 │    │192.168.1.100│
│MAC: AA:BB:..│    │MAC: DD:EE:..│    │MAC: 77:88:..│
└─────────────┘    └─────────────┘    └─────────────┘
```

**Scenario: Print a document to printer at 192.168.1.50**

```
Step 1: Check ARP table first
Your laptop checks: "Do I already know 192.168.1.50's MAC address?"

Current ARP Table:
┌──────────────┬───────────────────┬─────────────┐
│ IP Address   │ MAC Address       │ Age         │
├──────────────┼───────────────────┼─────────────┤
│192.168.1.1   │00:1A:2B:3C:4D:5E  │2 min ago    │
│192.168.1.100 │77:88:99:AA:BB:CC  │5 min ago    │
└──────────────┴───────────────────┴─────────────┘

Result: No entry for 192.168.1.50 found!

Step 2: Broadcast ARP Request
Your laptop broadcasts to EVERYONE on the network:

ARP Request Packet:
┌─────────────────────────────────────────────────────┐
│ Ethernet Header:                                    │
│   Destination: FF:FF:FF:FF:FF:FF (BROADCAST)        │
│   Source: AA:BB:CC:DD:EE:FF (your laptop)          │
│   Type: 0x0806 (ARP packet)                        │
├─────────────────────────────────────────────────────┤
│ ARP Payload:                                        │
│   Operation: 1 (ARP Request)                       │
│   Sender MAC: AA:BB:CC:DD:EE:FF (your laptop)      │
│   Sender IP: 192.168.1.10 (your laptop)           │
│   Target MAC: 00:00:00:00:00:00 (unknown!)         │
│   Target IP: 192.168.1.50 (printer we want)       │
└─────────────────────────────────────────────────────┘

Translation: "Hey everyone! Who has IP 192.168.1.50? 
Please tell AA:BB:CC:DD:EE:FF"

Step 3: All devices receive broadcast
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Your Laptop │    │  Printer    │    │File Server  │
│   Ignores   │    │"That's me!" │    │   Ignores   │
│  (sent it)  │    │             │    │ (not me)    │
└─────────────┘    └─────────────┘    └─────────────┘

Step 4: Printer responds with ARP Reply
Printer sends UNICAST response directly to your laptop:

ARP Reply Packet:
┌─────────────────────────────────────────────────────┐
│ Ethernet Header:                                    │
│   Destination: AA:BB:CC:DD:EE:FF (your laptop)     │
│   Source: DD:EE:FF:22:33:44 (printer)              │
│   Type: 0x0806 (ARP packet)                        │
├─────────────────────────────────────────────────────┤
│ ARP Payload:                                        │
│   Operation: 2 (ARP Reply)                         │
│   Sender MAC: DD:EE:FF:22:33:44 (printer)          │
│   Sender IP: 192.168.1.50 (printer)               │
│   Target MAC: AA:BB:CC:DD:EE:FF (your laptop)      │
│   Target IP: 192.168.1.10 (your laptop)           │
└─────────────────────────────────────────────────────┘

Translation: "Hi AA:BB:CC:DD:EE:FF! I am 192.168.1.50 
and my MAC address is DD:EE:FF:22:33:44"

Step 5: Update ARP table and print
Updated ARP Table:
┌──────────────┬───────────────────┬─────────────┐
│ IP Address   │ MAC Address       │ Age         │
├──────────────┼───────────────────┼─────────────┤
│192.168.1.1   │00:1A:2B:3C:4D:5E  │2 min ago    │
│192.168.1.50  │DD:EE:FF:22:33:44  │0 seconds    │ ← NEW!
│192.168.1.100 │77:88:99:AA:BB:CC  │5 min ago    │
└──────────────┴───────────────────┴─────────────┘

Now your laptop can send print data directly to DD:EE:FF:22:33:44
```

### ARP Cache Management

```
ARP Table Lifecycle:

Entry Creation: When ARP reply received
Entry Usage: Every time data sent to that IP
Entry Aging: Timer counts up from 0
Entry Expiration: Deleted after timeout (typically 20 minutes)

States and Timers:
┌─────────────┬─────────────────┬─────────────────────────────┐
│ State       │ Timer           │ What happens                │
├─────────────┼─────────────────┼─────────────────────────────┤
│ INCOMPLETE  │ 3 seconds       │ ARP request sent, waiting   │
│ REACHABLE   │ 30 seconds      │ Valid entry, recently used  │
│ STALE       │ 20 minutes      │ Old entry, needs refresh    │
└─────────────┴─────────────────┴─────────────────────────────┘

Cache Commands:
# View ARP table
arp -a                    # Windows
ip neighbor show          # Linux

# Add static entry (never expires)
arp -s 192.168.1.50 DD:EE:FF:22:33:44

# Delete entry (force re-discovery)
arp -d 192.168.1.50
```

### ARP Security Issues

**ARP Poisoning Attack:**
```
Normal Situation:
You: "Who has 192.168.1.1 (router)?"
Router: "I'm 192.168.1.1, my MAC is 00:1A:2B:3C:4D:5E"

Attack Situation:
You: "Who has 192.168.1.1?"
Hacker: "I'm 192.168.1.1, my MAC is 99:88:77:66:55:44" ← LYING!

Your ARP Table After Attack:
192.168.1.1 → 99:88:77:66:55:44 (attacker's MAC)

Result: All your internet traffic goes to attacker instead of router!
Attacker can: Read your data, modify it, block it, redirect to fake websites
```

**Protection Methods:**
```
1. Static ARP Entries:
   - Manually configure important devices (router, servers)
   - Entries never change, immune to attacks
   - Disadvantage: Manual maintenance required

2. ARP Monitoring:
   - Software watches for ARP table changes
   - Alerts when MAC addresses change unexpectedly
   - Examples: arpwatch, ntopng

3. Switch Port Security:
   - Switch learns first MAC address on each port
   - Blocks packets from different MAC addresses
   - Prevents spoofing attacks
```

---

## MAC (Media Access Control) Address

### What is MAC Address?
**One-line explanation:** A MAC address is a permanent, unique physical identifier for network devices.

### Real-World Analogy
MAC address is like a **Social Security Number or Vehicle VIN**:
- Assigned when device is manufactured (birth certificate)
- Never changes during device lifetime (permanent ID)
- Globally unique - no two devices have same MAC
- Used for local identification (like ID card in building)
- Format: 6 groups of 2 hex digits (AA:BB:CC:DD:EE:FF)

### MAC Address Structure

```
MAC Address Breakdown: AA:BB:CC:DD:EE:FF

┌─────────────────────────────┬─────────────────────────────┐
│         OUI (24 bits)       │        NIC (24 bits)        │
│      AA:BB:CC (3 bytes)     │      DD:EE:FF (3 bytes)     │
│     Company Identifier      │     Device Serial Number    │
└─────────────────────────────┴─────────────────────────────┘

Real Example: 00:1B:63:12:34:56
- 00:1B:63 = Apple Inc. (company code from IEEE)
- 12:34:56 = This specific device's unique number
```

### First Byte Special Bits

```
First Byte Analysis: AA (binary: 10101010)

Bit positions: 7 6 5 4 3 2 1 0
Binary value:  1 0 1 0 1 0 1 0
               │             │
               │             └─ I/G (Individual/Group) bit
               └─────────────── U/L (Universal/Local) bit

I/G Bit (bit 0):
- 0 = Individual (Unicast) - normal device address
- 1 = Group (Multicast) - group communication

U/L Bit (bit 1):  
- 0 = Universal (manufacturer assigned)
- 1 = Local (administrator assigned)

Examples:
00:1B:63:12:34:56 → 00 = 00000000 → Both bits 0 = Normal device MAC
01:00:5E:12:34:56 → 01 = 00000001 → I/G=1 = Multicast address
02:AB:CD:12:34:56 → 02 = 00000010 → U/L=1 = Locally configured
```

### Common Manufacturer Codes (OUI)

```
Popular OUI Codes:
┌─────────────────┬─────────────────────┬─────────────────────────┐
│ OUI Code        │ Manufacturer        │ Common Devices          │
├─────────────────┼─────────────────────┼─────────────────────────┤
│ 00:1B:63        │ Apple Inc.          │ iPhone, iPad, MacBook   │
│ 00:50:56        │ VMware Inc.         │ Virtual machines        │
│ 08:00:27        │ VirtualBox          │ VM network adapters     │
│ 00:0C:29        │ VMware Inc.         │ ESX server VMs          │
│ 00:15:5D        │ Microsoft Corp.     │ Hyper-V VMs            │
│ 00:1A:A0        │ Dell Inc.           │ Server network cards    │
│ 00:90:27        │ Intel Corporation   │ Ethernet controllers    │
│ 00:D0:2D        │ Novell Inc.         │ Network equipment       │
└─────────────────┴─────────────────────┴─────────────────────────┘

Special Patterns:
- 02:xx:xx:xx:xx:xx = Docker containers (locally administered)
- x2, x6, xA, xE (2nd bit set) = Custom/virtual MACs
```

### MAC Address Types and Usage

**1. Unicast (Individual Device)**
```
Format: Normal MAC like AA:BB:CC:DD:EE:FF
Usage: Regular network communication between two devices

Ethernet Frame Example:
┌───────────────────────┬───────────────────────┬─────────────┐
│ Destination MAC       │ Source MAC            │ Data        │
│ 11:22:33:44:55:66     │ AA:BB:CC:DD:EE:FF     │ Web page    │
│ (Specific printer)    │ (Your laptop)         │ to print    │
└───────────────────────┴───────────────────────┴─────────────┘

Network switch behavior:
- Looks up destination MAC in switch table
- Forwards frame only to port where that device is connected
- Other devices never see this traffic
```

**2. Multicast (Group Communication)**
```
Format: Starts with 01:xx:xx:xx:xx:xx (I/G bit = 1)
Usage: Send same data to multiple devices simultaneously

Common Multicast MACs:
┌─────────────────┬─────────────────────────────────────────────┐
│ MAC Address     │ Purpose                                     │
├─────────────────┼─────────────────────────────────────────────┤
│ 01:00:5E:xx:xx:xx│ IPv4 multicast (video streaming)          │
│ 33:33:xx:xx:xx:xx│ IPv6 multicast                            │
│ 01:80:C2:00:00:00│ Spanning Tree Protocol (STP)              │
│ 01:00:5E:00:00:FB│ Multicast DNS (find local services)       │
└─────────────────┴─────────────────────────────────────────────┘

Example - Video Conference:
One person sends video to: 01:00:5E:12:34:56
All conference participants receive the same stream
```

**3. Broadcast (Everyone)**
```
Format: FF:FF:FF:FF:FF:FF (all bits set to 1)
Usage: Send to every device on local network

Common Broadcast Uses:
- ARP requests: "Who has IP 192.168.1.20?"
- DHCP discovery: "Any DHCP servers available?"
- Wake-on-LAN: "Wake up computer with MAC xx:xx:xx:xx:xx:xx"

Network behavior:
- Switch floods broadcast to ALL ports
- Every device receives and processes the frame
- Can cause network congestion if overused
```

### How Switches Learn MAC Addresses

```
Switch MAC Address Table (CAM Table):

Initially Empty:
┌─────────────────┬─────────────┬─────────────┬─────────────┐
│ MAC Address     │ Port        │ VLAN        │ Age         │
├─────────────────┼─────────────┼─────────────┼─────────────┤
│ (empty)         │ (empty)     │ (empty)     │ (empty)     │
└─────────────────┴─────────────┴─────────────┴─────────────┘

Frame Arrives: AA:BB:CC:DD:EE:FF → 11:22:33:44:55:66 on Port 1
Switch thinks: "I learned AA:BB:CC:DD:EE:FF is on Port 1"

After Learning:
┌─────────────────┬─────────────┬─────────────┬─────────────┐
│ MAC Address     │ Port        │ VLAN        │ Age         │
├─────────────────┼─────────────┼─────────────┼─────────────┤
│AA:BB:CC:DD:EE:FF│ Port 1      │ VLAN 1      │ 0 seconds   │
└─────────────────┴─────────────┴─────────────┴─────────────┘

Next Frame: 11:22:33:44:55:66 → AA:BB:CC:DD:EE:FF on Port 5
Switch actions:
1. Learn: 11:22:33:44:55:66 is on Port 5
2. Forward: Send frame to Port 1 (knows AA:BB:CC:DD:EE:FF is there)

Final Table:
┌─────────────────┬─────────────┬─────────────┬─────────────┐
│ MAC Address     │ Port        │ VLAN        │ Age         │
├─────────────────┼─────────────┼─────────────┼─────────────┤
│AA:BB:CC:DD:EE:FF│ Port 1      │ VLAN 1      │ 15 seconds  │
│11:22:33:44:55:66│ Port 5      │ VLAN 1      │ 0 seconds   │
└─────────────────┴─────────────┴─────────────┴─────────────┘
```

### Virtual and Special MAC Addresses

```
Docker Container MACs:
Format: 02:42:xx:xx:xx:xx
Example: 02:42:AC:11:00:02

Explanation:
- 02 = U/L bit set (locally administered)
- 42 = Docker's chosen identifier (hex for 66)
- AC:11:00:02 = Container-specific part

VMware Virtual MACs:
Automatic: 00:0C:29:xx:xx:xx or 00:50:56:xx:xx:xx
Manual: 00:50:56:00:00:01 to 00:50:56:3F:FF:FF (range reserved)

Load Balancer Virtual MACs:
HSRP: 00:00:0C:07:AC:xx (Hot Standby Router Protocol)
VRRP: 00:00:5E:00:01:xx (Virtual Router Redundancy Protocol)
```

---

## IPv4 (Internet Protocol Version 4)

### What is IPv4?
**One-line explanation:** IPv4 assigns 32-bit numerical addresses to identify and route devices across networks.

### Real-World Analogy
IPv4 addresses work like **postal addresses**:
- Every building needs unique address for mail delivery
- Format: 192.168.1.10 (like 123 Main St, Springfield, IL)
- Postal workers (routers) read addresses and forward mail
- Limited addresses available (like limited street numbers in city)

### IPv4 Address Structure

```
IPv4 Address: 192.168.1.10

Decimal Format: 192 . 168 . 1 . 10
Binary Format:  11000000.10101000.00000001.00001010
Bit Count:      8 bits . 8 bits . 8 bits . 8 bits = 32 bits total

Address Space: 2^32 = 4,294,967,296 possible addresses (4.3 billion)

Think of it like: Country . State . City . House
Example meaning: Network 192.168.1, Host 10
```

### Public vs Private Addresses

```
Private Address Ranges (RFC 1918) - Internal networks only:
┌─────────────────────────┬─────────────────┬─────────────────┐
│ Address Range           │ Subnet Mask     │ Number of IPs   │
├─────────────────────────┼─────────────────┼─────────────────┤
│ 10.0.0.0 - 10.255.255.255│ 255.0.0.0 (/8) │ 16,777,216     │
│ 172.16.0.0 - 172.31.255.255│ 255.240.0.0 (/12)│ 1,048,576   │
│ 192.168.0.0 - 192.168.255.255│ 255.255.0.0 (/16)│ 65,536     │
└─────────────────────────┴─────────────────┴─────────────────┘

Usage Examples:
- 10.x.x.x: Large companies (universities, corporations)
- 172.16.x.x: Medium businesses
- 192.168.x.x: Home networks, small offices (most common)

Public Addresses: Everything else
- Routable on internet
- Assigned by Internet Service Providers (ISPs)
- Examples: 8.8.8.8 (Google DNS), 1.1.1.1 (Cloudflare DNS)
```

### Subnet Masks and CIDR Notation

```
Understanding Subnet Masks:

Network: 192.168.1.0/24
Full notation: 192.168.1.0 with subnet mask 255.255.255.0

Binary breakdown:
IP Address:    192.168.1.10  = 11000000.10101000.00000001.00001010
Subnet Mask:   255.255.255.0 = 11111111.11111111.11111111.00000000
                               └────────Network─────────┘└─Host─┘

/24 means: First 24 bits identify network, last 8 bits identify host
Result: 
- Network portion: 192.168.1 (identifies which network)
- Host portion: 10 (identifies specific device on that network)
- Available hosts: 2^8 - 2 = 254 (minus network and broadcast addresses)

Common CIDR Examples:
/8  = 255.0.0.0     = 16,777,214 hosts (Class A)
/16 = 255.255.0.0   = 65,534 hosts (Class B)  
/24 = 255.255.255.0 = 254 hosts (Class C)
/30 = 255.255.255.252 = 2 hosts (point-to-point links)
```

### Subnetting Example

```
Original Network: 192.168.1.0/24 (254 usable hosts)

Goal: Split into 4 smaller networks
Solution: Change from /24 to /26 (borrow 2 host bits for network)

Subnet Calculation:
Original: 11000000.10101000.00000001.00000000 /24
New:      11000000.10101000.00000001.00000000 /26
          └────────────Network────────────┘└─Host─┘

4 Resulting Subnets:
┌─────────────────┬─────────────────┬─────────────────────────────┐
│ Subnet          │ Usable Range    │ Purpose                     │
├─────────────────┼─────────────────┼─────────────────────────────┤
│ 192.168.1.0/26  │ .1 - .62       │ Sales Department (62 hosts) │
│ 192.168.1.64/26 │ .65 - .126     │ Engineering (62 hosts)      │
│ 192.168.1.128/26│ .129 - .190    │ Marketing (62 hosts)        │
│ 192.168.1.192/26│ .193 - .254    │ Servers (62 hosts)          │
└─────────────────┴─────────────────┴─────────────────────────────┘

Benefits:
- Logical separation by department
- Security policies per subnet
- Broadcast domains reduced (better performance)
- Easier troubleshooting
```

### IP Routing Process

```
Scenario: Computer (192.168.1.10) wants to reach Google (8.8.8.8)

Step 1: Destination Analysis
Computer checks: "Is 8.8.8.8 on my local network (192.168.1.0/24)?"
Calculation: 8.8.8.8 & 255.255.255.0 = 8.8.8.0 ≠ 192.168.1.0
Result: "No, this is a remote destination"

Step 2: Route to Default Gateway
Computer's routing table:
┌─────────────────┬─────────────────┬─────────────────┬───────────┐
│ Destination     │ Netmask         │ Gateway         │ Interface │
├─────────────────┼─────────────────┼─────────────────┼───────────┤
│ 0.0.0.0         │ 0.0.0.0         │ 192.168.1.1     │ eth0      │ ← Default
│ 192.168.1.0     │ 255.255.255.0   │ 0.0.0.0         │ eth0      │ ← Local
│ 127.0.0.0       │ 255.0.0.0       │ 127.0.0.1       │ lo        │ ← Loopback
└─────────────────┴─────────────────┴─────────────────┴───────────┘

Action: Send packet to default gateway (192.168.1.1)

Step 3: Router Forwarding
Home router checks its routing table:
- Destination 8.8.8.8 not in local networks
- Forward to ISP gateway (e.g., 10.1.1.1)

Step 4: ISP Routing
ISP router has more specific routes:
- 8.8.8.0/24 → Next hop toward Google's network
- Forward packet closer to destination

Step 5: Internet Backbone
Multiple routers forward packet:
Router A → Router B → Router C → Google's network

Step 6: Delivery
Google's router recognizes 8.8.8.8 as local
Delivers packet to correct server
```

### IPv4 Header Structure

```
IPv4 Header (20-60 bytes):

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌───┬───────┬───────────────┬─────────────────────────────────────┐
│Ver│  IHL  │     ToS       │            Total Length             │
├───┴───────┼───────────────┴───────────┬─┬─┬─┬─────────────────────┤
│           Identification              │D│M│R│   Fragment Offset   │
├───────────┬───────────────┬───────────┴─┴─┴─┴─────────────────────┤
│    TTL    │   Protocol    │            Header Checksum          │
├───────────┴───────────────┴─────────────────────────────────────┤
│                       Source Address                             │
├───────────────────────────────────────────────────────────────┤
│                    Destination Address                          │
├───────────────────────────────────────────────────────────────┤
│                         Options                               │
└───────────────────────────────────────────────────────────────┘

Key Fields Explained:
- Version (4 bits): Always 4 for IPv4
- IHL (4 bits): Header length in 32-bit words (minimum 5 = 20 bytes)
- ToS (8 bits): Type of Service (QoS markings)
- Total Length (16 bits): Entire packet size including data
- TTL (8 bits): Time to Live - decremented at each router hop
- Protocol (8 bits): Next layer (6=TCP, 17=UDP, 1=ICMP)
- Source/Dest Address (32 bits each): Sender and receiver IPs
```

### Common IPv4 Problems

```
1. Address Exhaustion:
Problem: Only 4.3 billion IPv4 addresses exist
Reality: 7+ billion people, 20+ billion devices
Solutions: 
- NAT (multiple devices share one public IP)
- IPv6 (340 undecillion addresses)
- CIDR (more efficient allocation)

2. Fragmentation:
Problem: Packet too large for network link
Solution: Router breaks packet into smaller pieces
Issue: Fragments can be lost, causing retransmission

3. Broadcast Storms:
Problem: Too many broadcast packets flood network
Symptoms: Network becomes slow or unusable
Solutions: VLANs, proper network design, broadcast filtering
```

---

## IPv6 (Internet Protocol Version 6)

### What is IPv6?
**One-line explanation:** IPv6 uses 128-bit addresses providing virtually unlimited address space for internet devices.

### Real-World Analogy
If IPv4 addresses are like **phone numbers in a small town**, IPv6 addresses are like **GPS coordinates precise enough to identify every grain of sand on every beach on Earth**.

- IPv4: 4.3 billion addresses (running out)
- IPv6: 340,282,366,920,938,463,463,374,607,431,768,211,456 addresses (never running out)

### IPv6 Address Format

```
Full IPv6 Address: 2001:0db8:85a3:0000:0000:8a2e:0370:7334

Structure: 8 groups of 4 hexadecimal digits
Each group: 16 bits (4 hex digits)
Total: 8 × 16 = 128 bits

Compression Rules:
1. Remove leading zeros: 0db8 → db8, 0000 → 0
2. Replace consecutive zeros with :: (only once per address)

Compression Examples:
┌──────────────────────────────────┬─────────────────────────┐
│ Original                         │ Compressed              │
├──────────────────────────────────┼─────────────────────────┤
│ 2001:0db8:85a3:0000:0000:8a2e:0370:7334 │ 2001:db8:85a3::8a2e:370:7334 │
│ 2001:0000:0000:0000:0000:0000:0000:0001 │ 2001::1                      │
│ 0000:0000:0000:0000:0000:0000:0000:0001 │ ::1 (loopback)               │
│ 0000:0000:0000:0000:0000:0000:0000:0000 │ :: (all zeros)               │
└──────────────────────────────────┴─────────────────────────┘
```

### IPv6 Address Types

```
1. Global Unicast Addresses (2000::/3):
Format: 2001:db8:1234:5678::1
Purpose: Internet-routable addresses (like public IPv4)
Scope: Global internet communication

Structure:
┌─────────────────┬─────────────────┬─────────────────────────┐
│ Global Prefix   │ Subnet ID       │ Interface ID            │
│ (48 bits)       │ (16 bits)       │ (64 bits)               │
│ ISP assigned    │ Your subnets    │ Device identifier       │
└─────────────────┴─────────────────┴─────────────────────────┘

2. Link-Local Addresses (fe80::/10):
Format: fe80::interface-id
Purpose: Local network communication only (like 169.254.x.x in IPv4)
Scope: Single network segment
Auto-generated: Every IPv6 device automatically gets one

3. Unique Local Addresses (fc00::/7):
Format: fd12:3456:789a::1
Purpose: Private networks (like 192.168.x.x in IPv4)
Scope: Organization-wide private communication

4. Multicast Addresses (ff00::/8):
Format: ff02::1 (all nodes), ff02::2 (all routers)
Purpose: One-to-many communication
Scope: Various (link-local, site-local, global)

5. Special Addresses:
::1 = Loopback (like 127.0.0.1 in IPv4)
:: = Unspecified address (like 0.0.0.0 in IPv4)
::ffff:192.168.1.1 = IPv4-mapped IPv6 address
```

### IPv6 Auto-Configuration (SLAAC)

```
Stateless Address Autoconfiguration Process:

Step 1: Generate Link-Local Address
Device creates: fe80::interface-identifier
Interface ID methods:
- EUI-64: Based on MAC address
- Random: Cryptographically generated
- Manual: Administrator configured

EUI-64 Example:
MAC Address: 00:1A:2B:3C:4D:5E
1. Split in half: 00:1A:2B | 3C:4D:5E
2. Insert FF:FE: 00:1A:2B:FF:FE:3C:4D:5E
3. Flip U/L bit: 02:1A:2B:FF:FE:3C:4D:5E
4. Result: fe80::21a:2bff:fe3c:4d5e

Step 2: Duplicate Address Detection (DAD)
Device sends Neighbor Solicitation to its own address
If no response: Address is unique and can be used
If response received: Address conflict, try different address

Step 3: Router Discovery
Device sends Router Solicitation: "Any routers here?"
Router responds with Router Advertisement containing:
- Network prefix (e.g., 2001:db8:1::/64)
- Default gateway information
- DNS servers
- Other network parameters

Step 4: Global Address Configuration
Combine router prefix + interface identifier:
Router prefix: 2001:db8:1::/64
Interface ID: ::21a:2bff:fe3c:4d5e
Result: 2001:db8:1::21a:2bff:fe3c:4d5e

Final Configuration:
┌─────────────────────────────┬─────────────────────────────┐
│ Address Type                │ Generated Address           │
├─────────────────────────────┼─────────────────────────────┤
│ Link-Local                  │ fe80::21a:2bff:fe3c:4d5e    │
│ Global Unicast              │ 2001:db8:1::21a:2bff:fe3c:4d5e │
└─────────────────────────────┴─────────────────────────────┘
```

### IPv6 vs IPv4 Comparison

```
┌─────────────────────┬─────────────────────┬─────────────────────┐
│ Feature             │ IPv4                │ IPv6                │
├─────────────────────┼─────────────────────┼─────────────────────┤
│ Address Length      │ 32 bits             │ 128 bits            │
│ Address Format      │ 192.168.1.1         │ 2001:db8::1         │
│ Address Space       │ 4.3 billion         │ 340 undecillion     │
│ Header Size         │ 20-60 bytes         │ 40 bytes (fixed)    │
│ Configuration       │ Manual/DHCP         │ Auto (SLAAC)        │
│ NAT Required        │ Yes (address shortage)│ No (plenty of addresses)│
│ Security            │ IPSec optional      │ IPSec built-in      │
│ QoS                 │ ToS field           │ Flow label          │
│ Broadcast           │ Yes                 │ No (multicast only) │
│ Fragmentation       │ Router + Host       │ Host only           │
│ Checksum            │ Header checksum     │ No checksum         │
└─────────────────────┴─────────────────────┴─────────────────────┘
```

### IPv6 Neighbor Discovery (replaces ARP)

```
Neighbor Discovery Protocol Messages:

1. Router Solicitation (RS):
Source: Link-local address of requesting node
Destination: ff02::2 (all routers multicast)
Purpose: "Are there any routers on this link?"

2. Router Advertisement (RA):
Source: Link-local address of router
Destination: ff02::1 (all nodes) or unicast to requesting node
Purpose: "I'm a router, here's network information"
Contains: Network prefixes, default router info, DNS servers

3. Neighbor Solicitation (NS):
Source: Link-local or global address of requesting node
Destination: Solicited-node multicast of target
Purpose: "What's your link-layer address?" (like ARP request)

4. Neighbor Advertisement (NA):
Source: Link-local or global address of responding node
Destination: Link-local address of requesting node
Purpose: "Here's my link-layer address" (like ARP reply)

Example NS/NA Exchange:
Node A (2001:db8::1) wants to reach Node B (2001:db8::2)

NS Message:
Source: 2001:db8::1
Destination: ff02::1:ff00:2 (solicited-node multicast for ::2)
Target: 2001:db8::2
Question: "What's the MAC address for 2001:db8::2?"

NA Message:
Source: 2001:db8::2
Destination: 2001:db8::1
Target: 2001:db8::2
Answer: "My MAC address is 11:22:33:44:55:66"
```

### Real-World IPv6 Example

```
Home Network with IPv6:

ISP Assignment: 2001:db8:1234::/48 (your allocated prefix)
Home Router: Uses 2001:db8:1234:1::/64 for LAN

Device Auto-Configuration:
┌─────────────┬─────────────────────────────────┬─────────────────────┐
│ Device      │ IPv6 Address                    │ How Obtained        │
├─────────────┼─────────────────────────────────┼─────────────────────┤
│ Router LAN  │ 2001:db8:1234:1::1              │ Manual config       │
│ Laptop      │ 2001:db8:1234:1::abcd:ef12:3456 │ SLAAC (random)      │
│ Phone       │ 2001:db8:1234:1::1a2b:3cff:fe4d:5e6f │ SLAAC (EUI-64) │
│ Smart TV    │ 2001:db8:1234:1::7890:abcd:ef12 │ SLAAC (random)      │
└─────────────┴─────────────────────────────────┴─────────────────────┘

Benefits over IPv4:
1. No NAT needed - each device has global address
2. Better performance - direct end-to-end connectivity  
3. Easier troubleshooting - no address translation complexity
4. Better security - IPSec built-in, harder to scan networks
5. Future-proof - address space will never be exhausted

Communication Example:
Your laptop (2001:db8:1234:1::abcd:ef12:3456) 
directly reaches Google (2001:4860:4860::8888)
No address translation required!
```

---

## Summary - How All Protocols Work Together

### Complete Communication Example

**Scenario: Browse YouTube from laptop on home network**

```
Network Setup:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Your Laptop │    │ Home Router │    │ YouTube     │
│192.168.1.10 │    │192.168.1.1  │    │208.65.153.238│
│(private)    │    │203.0.113.5  │    │(public)     │
│             │    │(public)     │    │             │
└─────────────┘    └─────────────┘    └─────────────┘

Step-by-Step Process:

1. DNS Resolution (not shown here, but happens first)
   - laptop resolves "youtube.com" to IP 208.65.153.238

2. Routing Decision
   - Laptop: "208.65.153.238 is not on my network (192.168.1.0/24)"
   - Laptop: "Send to default gateway 192.168.1.1"

3. ARP Resolution (find router's MAC)
   - Laptop checks ARP table for 192.168.1.1
   - If not found: ARP request "Who has 192.168.1.1?"
   - Router responds: "I'm 192.168.1.1 at MAC 00:1A:2B:3C:4D:5E"

4. Layer 2 Frame Creation
   ┌───────────────────────┬───────────────────────┬─────────────────┐
   │ Destination MAC       │ Source MAC            │ IP Packet       │
   │ 00:1A:2B:3C:4D:5E     │ AA:BB:CC:DD:EE:FF     │ (see below)     │
   │ (Router)              │ (Laptop)              │                 │
   └───────────────────────┴───────────────────────┴─────────────────┘

5. Layer 3 IP Packet Content
   ┌─────────────────┬─────────────────┬─────────────────────────┐
   │ Source IP       │ Destination IP  │ TCP Segment             │
   │ 192.168.1.10    │ 208.65.153.238  │ Port 80 (HTTP)          │
   │ (Laptop)        │ (YouTube)       │                         │
   └─────────────────┴─────────────────┴─────────────────────────┘

6. NAT Translation at Router
   - Router receives frame, extracts IP packet
   - Changes source IP: 192.168.1.10 → 203.0.113.5
   - Changes source port: 3000 → 5000 (unique external port)
   - Records in NAT table: 203.0.113.5:5000 ↔ 192.168.1.10:3000
   - Forwards packet to internet

7. Internet Routing
   - Multiple routers forward packet toward YouTube
   - Each router reads destination IP (208.65.153.238)
   - Packet eventually reaches YouTube's network

8. YouTube Response
   - YouTube sends response to 203.0.113.5:5000
   - Response travels back through internet to your router

9. NAT Reverse Translation
   - Router receives response for 203.0.113.5:5000
   - Looks up NAT table: "This belongs to 192.168.1.10:3000"
   - Changes destination: 203.0.113.5:5000 → 192.168.1.10:3000

10. Final Delivery
    - Router creates frame to laptop
    ┌───────────────────────┬───────────────────────┬─────────────────┐
    │ Destination MAC       │ Source MAC            │ IP Packet       │
    │ AA:BB:CC:DD:EE:FF     │ 00:1A:2B:3C:4D:5E     │ YouTube data    │
    │ (Laptop)              │ (Router)              │                 │
    └───────────────────────┴───────────────────────┴─────────────────┘
    - Laptop receives YouTube video stream!
```

### Protocol Layer Summary

```
┌─────────────┬─────────────┬─────────────────┬─────────────────────┐
│ Protocol    │ OSI Layer   │ Purpose         │ Key Security Risks  │
├─────────────┼─────────────┼─────────────────┼─────────────────────┤
│ MAC Address │ Layer 2     │ Local delivery  │ Spoofing, tracking  │
│ ARP         │ Layer 2     │ IP→MAC mapping  │ Poisoning, MITM     │
│ IPv4        │ Layer 3     │ Internet routing│ Scanning, spoofing  │
│ IPv6        │ Layer 3     │ Future routing  │ Complexity, privacy │
│ NAT         │ Layer 3     │ Address sharing │ Complexity, false   │
│             │             │                 │ sense of security   │
└─────────────┴─────────────┴─────────────────┴─────────────────────┘
```

Understanding these fundamentals is crucial for cybersecurity because:
- **Attack vectors** often exploit these protocols
- **Network monitoring** requires understanding normal behavior
- **Incident response** needs knowledge of how traffic flows
- **Security tools** (firewalls, IDS) operate at these layers