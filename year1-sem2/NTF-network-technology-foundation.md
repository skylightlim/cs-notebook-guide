# Network Technology Foundation (WIA1005)

> How computers talk to each other, from cables and switches up to the web. (Follows the Cisco CCNA *Introduction to Networks* style, with Cisco IOS commands.)

**Topics:** [Network Basics](#1-network-basics) · [Protocols & Models](#2-protocols-models--encapsulation) · [Physical & Data Link Layer](#3-physical--data-link-layer-ethernet-switching-arp) · [Network & Transport Layer](#4-network--transport-layer-ip-routing-tcpudp) · [IPv4 & Subnetting](#5-ipv4-addressing--subnetting) · [IPv6](#6-ipv6) · [Application Layer](#7-application-layer--network-services) · [Switch Configuration](#8-switch-configuration--security) · [VLANs](#9-vlans--inter-vlan-routing) · [DHCP](#10-dhcp) · [IOS Command Cheat Sheet](#cisco-ios-cheat-sheet)

---

## 1. Network Basics

A **network** is a group of devices connected to share data and resources.

### Components
| Type | Examples |
|---|---|
| **End devices** (hosts) | PCs, phones, servers, printers: where messages start or end |
| **Intermediary devices** | switches, routers, wireless access points, firewalls: forward data |
| **Media** | copper cable, fibre-optic, wireless |

> 📮 **Analogy:** the postal system. Houses are end devices, post offices and sorting centres are intermediary devices, roads and planes are the media.

**Peer-to-peer vs client-server:** in P2P every device is both client and server (small home setups); in client-server, dedicated servers provide services.

### Types of networks
| Type | Scope | Example |
|---|---|---|
| **LAN** | small area, one owner | home, office, campus |
| **WAN** | large area, connects LANs | between cities; run by ISPs |
| **Internet** | the worldwide network of networks | |
| Intranet / Extranet | private network for an organisation / its partners | UM's internal portal |

**Internet connections:** home users use cable, DSL, fibre, cellular, satellite. Businesses use dedicated leased lines, Metro Ethernet, business DSL.

### A reliable network needs
1. **Fault tolerance:** redundant paths, so if one breaks, traffic reroutes (packet switching helps).
2. **Scalability:** grow without slowing down.
3. **Quality of Service (QoS):** prioritise voice/video over email.
4. **Security:** confidentiality, integrity, availability (**CIA**).

**Trends:** BYOD, online collaboration, video, cloud computing, smart homes, powerline networking, wireless broadband.

**Security threats:** viruses/worms/trojans, spyware, zero-day attacks, DoS attacks, data theft. **Solutions:** antivirus, firewalls, ACLs, IPS, VPNs.

---

## 2. Protocols, Models & Encapsulation

A **protocol** is a set of rules for communication: message format, size, timing, encoding, and delivery options (**unicast** = one, **multicast** = some, **broadcast** = all).

> 🗣️ **Analogy:** a phone call has rules too: say hello, take turns, speak the same language, say goodbye.

### OSI vs TCP/IP models
| # | OSI layer | TCP/IP layer | Job | PDU | Example |
|---|---|---|---|---|---|
| 7 | Application | **Application** | interface to the user's apps | Data | HTTP, DNS, SMTP |
| 6 | Presentation | ↑ | format, compress, encrypt | Data | JPEG, TLS |
| 5 | Session | ↑ | start, maintain, end dialogues | Data | |
| 4 | Transport | **Transport** | end-to-end delivery, ports | **Segment** | TCP, UDP |
| 3 | Network | **Internet** | logical addressing, routing | **Packet** | IP, ICMP |
| 2 | Data Link | **Network Access** | MAC addressing, framing | **Frame** | Ethernet, Wi-Fi |
| 1 | Physical | ↑ | bits on the wire | **Bits** | cables, signals |

💡 *Memory trick (7→1):* "**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing".

### Encapsulation
As data goes **down** the layers, each layer adds its own **header** (and the data link layer also adds a trailer). The receiver **de-encapsulates** going up.
```
Data
[TCP hdr | Data]                              ← segment
[IP hdr | TCP hdr | Data]                     ← packet
[Eth hdr | IP hdr | TCP hdr | Data | FCS]     ← frame
101010110…                                    ← bits
```
> 📦 **Analogy:** a gift (data) goes in a box labelled "for Mum" (TCP: which app), then in a shipping box with the house address (IP), then on a truck with a route label for the next depot (Ethernet MAC). Each depot swaps the truck label but keeps the house address.

### Addresses
- **IP address (layer 3):** the end-to-end logical address; stays the same from source to destination.
- **MAC address (layer 2):** the physical address of a NIC (48-bit, e.g. `00-1A-2B-3C-4D-5E`); changes at **every hop**.
- To reach a remote network, a host sends the frame to its **default gateway** (router) MAC.

### Network operating system (Cisco IOS) basics
Access via **console** cable (out-of-band), **SSH** (secure, in-band), or **Telnet** (insecure).
```
Switch>                 ← User EXEC mode (limited)
Switch> enable
Switch#                 ← Privileged EXEC mode
Switch# configure terminal
Switch(config)#         ← Global config mode
Switch(config)# hostname S1
S1(config)# enable secret class          ← password for privileged mode (encrypted)
S1(config)# line console 0
S1(config-line)# password cisco
S1(config-line)# login
S1(config)# service password-encryption
S1(config)# banner motd #Authorized access only!#
S1# copy running-config startup-config  ← SAVE (or: write memory)
```
`running-config` lives in RAM (lost on reboot); `startup-config` lives in NVRAM. Use `?` for help and **Tab** to complete commands.

---

## 3. Physical & Data Link Layer (Ethernet, Switching, ARP)

### Physical layer media
| Media | Signal | Notes |
|---|---|---|
| **Copper (UTP)** | electrical | cheap, common; up to 100 m; susceptible to interference (EMI, crosstalk). Cat5e/6 with RJ-45 connectors. Straight-through (PC↔switch) vs crossover cables |
| **Fibre optic** | light | long distances, very high bandwidth, immune to EMI. Single-mode (laser, km) vs multimode (LED, shorter) |
| **Wireless** | radio waves | mobility; interference and security concerns. Wi-Fi (802.11), Bluetooth |

**Bandwidth** = capacity; **throughput** = actual measured rate; **goodput** = useful data rate (minus overhead).

### Data link layer
Two sublayers: **LLC** (talks to the network layer) and **MAC** (controls media access, adds addresses).

**Media access control:**
- **Contention-based:** CSMA/CD (old wired Ethernet hubs: listen, transmit, detect collisions), CSMA/CA (Wi-Fi: avoid collisions).
- **Controlled:** take turns (token ring).
- **Duplex:** half-duplex (one direction at a time, like a walkie-talkie) vs full-duplex (both at once, like a phone).

**Topologies:** physical (how it's wired: star, bus, ring, mesh) vs logical (how data flows).

### Ethernet frame
```
| Preamble | Dest MAC | Src MAC | Type | Data (46–1500 B) | FCS |
```
**FCS** (frame check sequence) = CRC error check. Minimum frame 64 bytes, maximum 1518.
**MAC address:** the first 24 bits = **OUI** (vendor), the last 24 = device. Broadcast MAC = `FF-FF-FF-FF-FF-FF`.

### How a switch works
A switch **learns** source MACs into its **MAC address table** (CAM) and **forwards** frames by destination MAC.
1. Frame arrives → record "source MAC is on port X".
2. Look up the destination MAC:
   - Known → send out that port only.
   - Unknown or broadcast → **flood** out all ports except the incoming one.

> 🏢 **Analogy:** a receptionist who learns which office each person sits in as they walk past. Mail for a known person goes straight to their office; mail for an unknown person gets announced on every floor.

**Switching methods:** store-and-forward (checks the whole frame for errors) vs cut-through (starts forwarding after reading the destination; faster).

### ARP (Address Resolution Protocol)
Finds the **MAC address** for a known **IPv4 address** on the local network.
1. PC A: "Who has 192.168.1.5? Tell 192.168.1.2" (broadcast **ARP request**).
2. PC B: "192.168.1.5 is at AA:BB:CC:…" (unicast **ARP reply**).
3. A stores it in its **ARP cache** (`arp -a`).

> 📢 Like shouting in a room, "Who's Ahmad?", and Ahmad raising his hand.

Risks: ARP spoofing/poisoning (an attacker answers falsely). IPv6 uses **Neighbor Discovery** instead.

---

## 4. Network & Transport Layer (IP, Routing, TCP/UDP)

### Network layer
Jobs: addressing end devices, encapsulation, **routing**, de-encapsulation.

**IP characteristics:** **connectionless** (no setup), **best-effort** (unreliable, no guarantee), **media independent**.

**IPv4 header key fields:** version, DSCP (QoS), **TTL** (time to live, decremented at each router; the packet is dropped at 0 to prevent loops), protocol (6 = TCP, 17 = UDP), source IP, destination IP.

### Routing
A host decides: is the destination on **my network** → send directly; otherwise → send to the **default gateway**.

A **router** forwards packets using its **routing table**:
```
R1# show ip route
C    192.168.1.0/24 is directly connected, G0/0
L    192.168.1.1/32 is directly connected, G0/0
S    10.0.0.0/8 [1/0] via 209.165.200.226
O    172.16.0.0/16 [110/2] via 192.168.2.2      ← learned by OSPF
S*   0.0.0.0/0 via 209.165.200.226              ← default route ("gateway of last resort")
```
- **Directly connected** (C, L), **static** (S, configured by hand), **dynamic** (learned via protocols like OSPF, EIGRP, RIP).
- **Longest prefix match:** the most specific matching route wins.

> 🗺️ **Analogy:** a router is a highway interchange with signs. It doesn't know the whole journey, just "for Penang, take exit 3".

```
R1(config)# interface g0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config)# ip route 10.0.0.0 255.0.0.0 209.165.200.226     ← static route
R1(config)# ip route 0.0.0.0 0.0.0.0 209.165.200.226        ← default route
```

### ICMP
Error and diagnostic messages for IP: `ping` (echo request/reply, tests reachability), `traceroute`/`tracert` (lists each hop using TTL), destination unreachable, time exceeded.

### Transport layer
Jobs: track individual conversations, segment data, reassemble it, and identify applications using **port numbers**.
- Well-known ports 0–1023 (HTTP 80, HTTPS 443, SSH 22, DNS 53, SMTP 25), registered 1024–49151, dynamic/private 49152–65535.
- **Socket** = IP address + port, e.g. `192.168.1.5:443`.

| | **TCP** | **UDP** |
|---|---|---|
| Connection | connection-oriented (**3-way handshake**) | connectionless |
| Reliability | acknowledgments, retransmission, ordered delivery | best-effort; no ACKs |
| Flow control | ✅ windowing | ❌ |
| Header | 20 bytes | 8 bytes |
| Speed | slower | faster |
| Used for | web, email, file transfer | streaming, VoIP, gaming, DNS, DHCP |

> 📦 **Analogy:** TCP = registered mail with tracking and signatures; if a parcel goes missing, it's re-sent. UDP = dropping flyers in letterboxes: fast, but no idea if they all arrived.

**3-way handshake:** `SYN →`, `← SYN-ACK`, `ACK →`. **Termination:** FIN/ACK exchanges (4 steps).

**Flow control / windowing:** the receiver says how much data it can accept before acknowledging (window size).

---

## 5. IPv4 Addressing & Subnetting

An IPv4 address is **32 bits**, written as 4 decimal octets: `192.168.10.10`.
The **subnet mask** (or prefix length) splits it into **network** and **host** parts:
```
IP:    192.168.10.10   = 11000000.10101000.00001010.00001010
Mask:  255.255.255.0   = 11111111.11111111.11111111.00000000  (/24)
Network: 192.168.10.0      Host part: .10
```
> 🏘️ **Analogy:** the network part is the street name; the host part is the house number.

For any network:
- **Network address:** all host bits 0 (`192.168.10.0`)
- **Broadcast address:** all host bits 1 (`192.168.10.255`)
- **Usable hosts:** `2ʰ − 2` (h = host bits). /24 → 254 hosts.

### Types and special addresses
- **Unicast**, **broadcast**, **multicast** (224.0.0.0–239.255.255.255).
- **Private addresses** (not routed on the internet; used with NAT): `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.
- **Loopback:** `127.0.0.1`. **Link-local (APIPA):** `169.254.0.0/16` (you got no DHCP address).
- Legacy classes: A (/8, 1–126), B (/16, 128–191), C (/24, 192–223).

### Subnetting
Borrow host bits to make more, smaller networks.
- Subnets created = `2ⁿ` (n = bits borrowed)
- Hosts per subnet = `2ʰ − 2`
- **Block size** (magic number) = `256 − (mask value in the interesting octet)`

**Example:** subnet `192.168.1.0/24` into 4 subnets → borrow 2 bits → `/26` (mask `255.255.255.192`), block size 64, 62 hosts each.
| Subnet | Network | First host | Last host | Broadcast |
|---|---|---|---|---|
| 1 | .0 | .1 | .62 | .63 |
| 2 | .64 | .65 | .126 | .127 |
| 3 | .128 | .129 | .190 | .191 |
| 4 | .192 | .193 | .254 | .255 |

| Prefix | Mask | Hosts |
|---|---|---|
| /24 | 255.255.255.0 | 254 |
| /25 | .128 | 126 |
| /26 | .192 | 62 |
| /27 | .224 | 30 |
| /28 | .240 | 14 |
| /29 | .248 | 6 |
| /30 | .252 | 2 (point-to-point links) |

### VLSM (Variable Length Subnet Masking)
Give each subnet only the size it needs. **Start with the largest requirement first.**
*Example from 192.168.1.0/24:* LAN A needs 100 hosts → /25 (.0–.127); LAN B 50 hosts → /26 (.128–.191); LAN C 20 → /27 (.192–.223); router links → /30s from .224.

---

## 6. IPv6

**Why?** IPv4 has only ~4.3 billion addresses, and they've run out. IPv6 has **128 bits** (≈ 3.4 × 10³⁸ addresses).

Format: 8 groups ("hextets") of 4 hex digits: `2001:0db8:0000:0000:0000:ff00:0042:8329`

**Compression rules:**
1. Omit **leading zeros** in each hextet: `0db8` → `db8`, `0042` → `42`.
2. Replace **one** run of all-zero hextets with `::` (only once per address).
```
2001:0db8:0000:0000:0000:ff00:0042:8329  →  2001:db8::ff00:42:8329
```

### Address types (no broadcast in IPv6!)
| Type | Prefix | Purpose |
|---|---|---|
| **Global unicast (GUA)** | `2000::/3` | public, routable (like public IPv4) |
| **Link-local (LLA)** | `fe80::/10` | only on the local link; every IPv6 interface has one; used as the default gateway address |
| Unique local | `fc00::/7` | private (like 10.x) |
| Loopback | `::1` | |
| Multicast | `ff00::/8` | `ff02::1` all nodes, `ff02::2` all routers |

**GUA structure:** `Global routing prefix (/48) | Subnet ID (16 bits) | Interface ID (64 bits)`, so a standard subnet is a **/64**.

### Getting an address
- **Static**, or
- **SLAAC** (Stateless Address Autoconfiguration): the router sends a **Router Advertisement (RA)** with the prefix; the host creates its own interface ID (via **EUI-64**, which builds it from the MAC, or randomly).
- **DHCPv6** (stateless or stateful); see section 10.

### ICMPv6 / Neighbor Discovery
| Message | Purpose |
|---|---|
| Router Solicitation (RS) / Router Advertisement (RA) | find routers and the prefix |
| Neighbor Solicitation (NS) / Neighbor Advertisement (NA) | address resolution (replaces ARP) and **Duplicate Address Detection** |

**IPv6 subnetting:** usually just increment the 16-bit subnet ID: `2001:db8:acad:1::/64`, `2001:db8:acad:2::/64`, …

**Transition from IPv4:** dual stack, tunnelling, translation (NAT64).

---

## 7. Application Layer & Network Services

The top three OSI layers (application, presentation, session) map to the TCP/IP **application** layer.
- **Presentation:** formatting, compression, encryption (JPEG, MP4, TLS).
- **Session:** start, maintain, and restart dialogues.

| Protocol | Port | Purpose |
|---|---|---|
| **HTTP / HTTPS** | 80 / 443 | web (GET, POST, PUT); HTTPS adds encryption |
| **SMTP** | 25 | **send** email |
| **POP3** | 110 | download email (usually deleted from the server) |
| **IMAP** | 143 | email synced on the server (multiple devices) |
| **DNS** | 53 | translate names → IP (`www.um.edu.my` → 103.x.x.x) |
| **DHCP** | 67/68 | assign IP addresses automatically |
| **FTP** | 20/21 | file transfer (21 control, 20 data) |
| **TFTP** | 69 | simple file transfer (UDP), e.g. IOS images |
| **SSH / Telnet** | 22 / 23 | remote access (secure / insecure) |
| **SMB** | 445 | Windows file and printer sharing |

> 📖 **DNS analogy:** the internet's phone book. You remember names; computers need numbers.

**DNS hierarchy:** root (`.`) → top-level domains (`.com`, `.my`) → second-level (`edu.my`) → … Record types: **A** (IPv4), **AAAA** (IPv6), **NS**, **MX** (mail server), **CNAME** (alias). Test with `nslookup`.

### Small networks & troubleshooting
- Design: plan IP addressing, redundancy, and device choice for growth.
- Useful commands: `ping`, `tracert`/`traceroute`, `ipconfig /all`, `arp -a`, `show ip interface brief`, `show running-config`, `show interfaces`, `show cdp neighbors`, `debug`.
- **Troubleshooting approaches:** bottom-up (start at the physical layer), top-down, divide-and-conquer, follow the path, substitution, educated guess.

---

## 8. Switch Configuration & Security

### Basic switch management
```
S1(config)# interface vlan 1
S1(config-if)# ip address 192.168.1.2 255.255.255.0     ← management IP (SVI)
S1(config-if)# no shutdown
S1(config)# ip default-gateway 192.168.1.1
S1(config)# interface f0/1
S1(config-if)# duplex full
S1(config-if)# speed 100
S1(config-if)# mdix auto
```
**Boot sequence:** POST → boot loader → load IOS from flash → load startup-config.

**Collision domains vs broadcast domains:** every switch port is its own collision domain; the whole switch (one VLAN) is one broadcast domain. Routers (and VLANs) separate broadcast domains.

### Configuring SSH
```
S1(config)# ip domain-name um.edu.my
S1(config)# crypto key generate rsa         ← choose modulus 1024+
S1(config)# username admin secret StrongPass1
S1(config)# line vty 0 15
S1(config-line)# transport input ssh
S1(config-line)# login local
S1(config)# ip ssh version 2
```

### Port security
Limits which/how many MAC addresses can use a port.
```
S1(config)# interface f0/5
S1(config-if)# switchport mode access
S1(config-if)# switchport port-security
S1(config-if)# switchport port-security maximum 2
S1(config-if)# switchport port-security mac-address sticky      ← learn and remember MACs
S1(config-if)# switchport port-security violation shutdown       ← or restrict / protect
S1# show port-security interface f0/5
```
| Violation mode | Drops traffic | Logs/alerts | Shuts the port |
|---|---|---|---|
| protect | ✅ | ❌ | ❌ |
| restrict | ✅ | ✅ | ❌ |
| shutdown (default) | ✅ | ✅ | ✅ (err-disabled) |

**Other best practices:** shut down unused ports (`shutdown`), put them in an unused VLAN, use strong passwords, use SSH instead of Telnet.

---

## 9. VLANs & Inter-VLAN Routing

A **VLAN** (Virtual LAN) splits one physical switch into multiple **logical** networks. Each VLAN = its own broadcast domain = its own IP subnet.

> 🏢 **Analogy:** one office building (switch) with different departments on different "virtual floors". HR's broadcasts don't disturb Engineering, even though they share the same building.

**Benefits:** smaller broadcast domains, better security, easier management, cost savings.

### VLAN types
| Type | Purpose |
|---|---|
| **Default VLAN** | VLAN 1; all ports start here |
| **Data VLAN** | user traffic |
| **Native VLAN** | untagged traffic on a trunk (default VLAN 1; change it for security) |
| **Management VLAN** | for switch management (SSH/SVI) |
| **Voice VLAN** | IP phones, prioritised by QoS |

### Access vs trunk ports
- **Access port:** belongs to one VLAN (connects to an end device).
- **Trunk port:** carries **multiple VLANs** between switches/routers using **802.1Q tags** (a 4-byte tag with the VLAN ID inserted into the frame).

```
S1(config)# vlan 10
S1(config-vlan)# name STUDENTS
S1(config)# interface f0/6
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 10

S1(config)# interface g0/1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 99
S1(config-if)# switchport trunk allowed vlan 10,20,99
S1# show vlan brief
S1# show interfaces trunk
```
**DTP** (Dynamic Trunking Protocol) can negotiate trunks automatically; disable it for security: `switchport nonegotiate`.

### Inter-VLAN routing
Devices in different VLANs need a **layer 3 device** to talk.
| Method | How | Notes |
|---|---|---|
| Legacy | one router interface per VLAN | doesn't scale |
| **Router-on-a-stick** | one trunk link to a router with **subinterfaces** | common for small networks |
| **Layer 3 switch (SVIs)** | the switch routes between VLAN interfaces | fastest, scalable |

```
! Router-on-a-stick
R1(config)# interface g0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config)# interface g0/0.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config)# interface g0/0
R1(config-if)# no shutdown

! Layer 3 switch
S1(config)# ip routing
S1(config)# interface vlan 10
S1(config-if)# ip address 192.168.10.1 255.255.255.0
```

### VLAN attacks
- **VLAN hopping (switch spoofing):** an attacker pretends to be a switch to form a trunk. **Fix:** set user ports to `switchport mode access` and disable DTP.
- **Double tagging:** an attacker adds two 802.1Q tags to reach another VLAN via the native VLAN. **Fix:** change the native VLAN to an unused VLAN.

---

## 10. DHCP

**DHCP** automatically gives hosts an IP address, subnet mask, default gateway, and DNS server.

> 🏨 **Analogy:** a hotel front desk giving you a room key (IP address) for the length of your stay (the **lease**).

### DHCPv4 process: **DORA**
```
Client                          Server
  │── DHCPDISCOVER (broadcast) ──►│   "Any DHCP servers?"
  │◄── DHCPOFFER ─────────────────│   "You can have 192.168.1.10"
  │── DHCPREQUEST (broadcast) ───►│   "I'll take it!"
  │◄── DHCPACK ───────────────────│   "Confirmed, lease 24h"
```
Client port **68**, server port **67** (UDP). Lease renewal: REQUEST/ACK at 50% of the lease.

```
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10   ← reserve for static devices
R1(config)# ip dhcp pool LAN1
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# dns-server 8.8.8.8
R1(dhcp-config)# domain-name um.edu.my
R1# show ip dhcp binding

! If the server is on another network, forward the broadcasts:
R1(config)# interface g0/1
R1(config-if)# ip helper-address 10.1.1.2
```

### DHCPv6
| Method | Address from | Other info (DNS) from | RA flags |
|---|---|---|---|
| **SLAAC only** | RA prefix (self-made) | RA | A=1, O=0, M=0 |
| **Stateless DHCPv6** | RA prefix (self-made) | DHCPv6 server | A=1, O=1 |
| **Stateful DHCPv6** | DHCPv6 server | DHCPv6 server | M=1 |

DHCPv6 messages: SOLICIT → ADVERTISE → REQUEST → REPLY.
```
R1(config)# ipv6 unicast-routing
R1(config)# ipv6 dhcp pool STATELESS
R1(config-dhcpv6)# dns-server 2001:db8::8
R1(config)# interface g0/0
R1(config-if)# ipv6 nd other-config-flag
R1(config-if)# ipv6 dhcp server STATELESS
```

### DHCP attacks
- **DHCP starvation:** an attacker requests every address with fake MACs, so legitimate users get none.
- **DHCP spoofing:** a rogue DHCP server gives out a fake gateway/DNS (man-in-the-middle).
- **Fix: DHCP snooping.** Only **trusted** ports (toward the real server) can send DHCP offers; rate-limit the others.
```
S1(config)# ip dhcp snooping
S1(config)# ip dhcp snooping vlan 10
S1(config)# interface g0/1
S1(config-if)# ip dhcp snooping trust
S1(config)# interface range f0/1-24
S1(config-if-range)# ip dhcp snooping limit rate 6
```

---

## Cisco IOS cheat sheet
| Task | Command |
|---|---|
| Interface summary | `show ip interface brief` |
| See config | `show running-config` |
| Save config | `copy running-config startup-config` |
| Routing table | `show ip route` |
| MAC table | `show mac address-table` |
| VLANs | `show vlan brief` |
| Trunks | `show interfaces trunk` |
| Neighbours | `show cdp neighbors` |
| Test reachability | `ping`, `traceroute` |
| Leave a mode | `exit` (one level), `end` (to privileged EXEC) |

---

### ✅ Quick self-check
1. At which OSI layer does a switch work? A router? *(2; 3)*
2. How many usable hosts in a /27? *(30)*
3. Compress `2001:0db8:0000:0000:0001:0000:0000:0001`. *(2001:db8::1:0:0:1)*
4. TCP or UDP for a video call? *(UDP)*
5. What are the four DHCP DORA messages? *(Discover, Offer, Request, Acknowledge)*
6. What's the difference between an access port and a trunk port? *(one VLAN vs many VLANs with 802.1Q tags)*
