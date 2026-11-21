# Advanced Network Technology (WIA2008)

> Building bigger, more reliable, more secure networks: redundancy, routing protocols, filtering, NAT, management, and the cloud. (Cisco CCNA SRWE/ENSA style.) Builds on [NTF](../year1-sem2/NTF-network-technology-foundation.md).

**Topics:** [STP](#1-spanning-tree-protocol-stp) · [EtherChannel](#2-etherchannel) · [LAN Security](#3-lan-security-concepts) · [OSPF](#4-ospf) · [ACLs](#5-access-control-lists-acls) · [NAT](#6-nat) · [VPN & IPsec](#7-vpn--ipsec) · [Network Management](#8-network-management-cdp-lldp-ntp-syslog-snmp) · [Network Design & Troubleshooting](#9-network-design--troubleshooting) · [Cloud, Virtualization, SDN, Automation](#10-cloud-virtualization-sdn--automation) · [Network Security](#11-network-security-threats--best-practices) · [Labs Recap](#labs-recap)

---

## 1. Spanning Tree Protocol (STP)

**Problem:** redundant links between switches are good for reliability, but they create **layer 2 loops**. Ethernet frames have no TTL, so they circle forever:
- **Broadcast storms** (broadcasts multiply until the network collapses)
- **MAC table instability** (the same MAC seen on different ports)
- **Duplicate frames**

**STP (IEEE 802.1D)** fixes this by **logically blocking** some ports, so there's exactly one active path, while keeping the backup links ready.

> 🛣️ **Analogy:** a city with several roads between two places. STP puts "road closed" barriers on some so traffic can't go in circles. If the main road collapses, a barrier is lifted.

### How STP chooses (the Spanning Tree Algorithm)
1. **Elect the root bridge:** the switch with the lowest **Bridge ID** = priority (default 32768) + VLAN ID + MAC. Lowest priority wins; on a tie, the lowest MAC wins.
2. **Elect root ports:** on each non-root switch, the port with the lowest **cost** to the root (10 Gbps = 2, 1 Gbps = 4, 100 Mbps = 19, 10 Mbps = 100).
3. **Elect designated ports:** on each segment, the port closest to the root. All root bridge ports are designated.
4. **Everything else = alternate/blocked.**

Tie-breakers: lowest root path cost → lowest sender bridge ID → lowest sender port priority → lowest port ID.

Switches exchange **BPDUs** (Bridge Protocol Data Units) every 2 s (hello timer).

### Port states (802.1D)
Blocking (20 s max age) → Listening (15 s) → Learning (15 s) → **Forwarding**. Classic STP can take **30–50 seconds** to converge.

### Evolution of STP
| Protocol | Standard | Notes |
|---|---|---|
| STP | 802.1D | one tree for all VLANs, slow |
| PVST+ | Cisco | one tree **per VLAN** |
| **RSTP** | 802.1w | **rapid** convergence (< 1 s); port roles: root, designated, alternate, backup |
| Rapid PVST+ | Cisco | RSTP per VLAN (the recommended choice) |
| MSTP | 802.1s | maps many VLANs onto a few trees |

### Configuration
```
S1(config)# spanning-tree mode rapid-pvst
S1(config)# spanning-tree vlan 1 root primary         ← make this switch the root
S2(config)# spanning-tree vlan 1 root secondary
S1(config)# spanning-tree vlan 10 priority 4096        ← or set the priority directly (multiples of 4096)

! Edge ports to PCs: skip listening/learning
S1(config-if)# spanning-tree portfast
S1(config-if)# spanning-tree bpduguard enable          ← shut the port if a BPDU arrives (rogue switch!)
S1# show spanning-tree
```

---

## 2. EtherChannel

**EtherChannel** bundles several physical links into **one logical link**.
- More bandwidth (e.g. 4 × 1 Gbps = 4 Gbps)
- Redundancy (one cable fails → the others carry on)
- STP sees it as one link, so it **doesn't block** the extra cables
- Load balancing across the links

> 🛤️ **Analogy:** merging several single-lane roads into one multi-lane highway.

**Rules:** all member ports must have the same speed, duplex, VLAN settings, and mode (access/trunk). Up to 8 active links.

### Negotiation protocols
| Protocol | Owner | Modes | Forms a channel when… |
|---|---|---|---|
| **PAgP** | Cisco proprietary | `desirable` (actively asks), `auto` (waits) | desirable–desirable, desirable–auto |
| **LACP** | IEEE 802.3ad (open) | `active` (actively asks), `passive` (waits) | active–active, active–passive |
| Static | none | `on` | on–on only |

`auto–auto` or `passive–passive` = no channel (both waiting!).

```
S1(config)# interface range f0/1 - 2
S1(config-if-range)# channel-group 1 mode active          ← LACP
S1(config)# interface port-channel 1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk allowed vlan 10,20
S1# show etherchannel summary
```

---

## 3. LAN Security Concepts

### Endpoint security
Endpoints (PCs, phones) are a common way in. Protect them with antivirus/antimalware, host firewalls, HIPS, and devices like **ESA** (Email Security Appliance), **WSA** (Web Security Appliance), and NAC.

### Access control: **AAA**
| | Question | Example |
|---|---|---|
| **Authentication** | Who are you? | username/password, certificate |
| **Authorization** | What are you allowed to do? | can only view, not configure |
| **Accounting** | What did you do? | logs of commands and session times |

> 🏦 **Analogy:** a bank. Show your ID (authentication), you can access only your own account (authorization), and every transaction is recorded (accounting).

Servers: **RADIUS** (open standard, UDP, encrypts only the password) and **TACACS+** (Cisco, TCP, encrypts everything, separates the three A's). **802.1X** = port-based authentication before a device can use the LAN.

### Layer 2 attacks and defences
| Attack | What happens | Defence |
|---|---|---|
| **MAC address table flooding** | attacker floods fake MACs → the switch table overflows → the switch floods all traffic like a hub (attacker sniffs everything) | **port security** |
| VLAN hopping / double tagging | access other VLANs | disable DTP, change the native VLAN |
| DHCP starvation / spoofing | exhaust the pool / rogue server | **DHCP snooping** |
| **ARP spoofing/poisoning** | fake ARP replies → man-in-the-middle | **Dynamic ARP Inspection (DAI)** |
| Address (IP/MAC) spoofing | pretend to be another host | **IP Source Guard** |
| STP manipulation | attacker becomes the root bridge | **BPDU Guard**, Root Guard |
| CDP/LLDP reconnaissance | learn device info | disable CDP/LLDP on edge ports |

```
S1(config)# ip arp inspection vlan 10
S1(config-if)# ip arp inspection trust          ← on uplinks
S1(config-if)# ip verify source                 ← IP Source Guard
```

---

## 4. OSPF

**OSPF** (Open Shortest Path First) is a **link-state** dynamic routing protocol. Every router builds a full map of the network and runs **Dijkstra's SPF algorithm** to find the shortest paths. (See [Dijkstra in CM](../year1-sem1/CM-computing-mathematics.md#dijkstras-algorithm-shortest-path-in-a-weighted-graph-no-negative-weights).)

> 🗺️ **Analogy:** distance-vector protocols (RIP) are like asking for directions at each junction. Link-state (OSPF) is like every driver having the full map and GPS.

### Key ideas
- **Metric = cost** = reference bandwidth / interface bandwidth (default reference 100 Mbps, so FastEthernet and anything faster = 1). Set `auto-cost reference-bandwidth 10000` for modern links.
- **Areas:** split big networks to limit updates. **Area 0** = backbone. Single-area OSPF = everything in area 0.
- **Router ID:** a 32-bit ID, chosen as: configured `router-id` → highest loopback IP → highest active interface IP.
- Administrative distance = **110**.

### Packet types
| Type | Name | Purpose |
|---|---|---|
| 1 | **Hello** | discover neighbours, keep adjacency alive (every 10 s on Ethernet; dead = 40 s) |
| 2 | DBD | database summary |
| 3 | LSR | link-state request |
| 4 | LSU | link-state update (contains LSAs) |
| 5 | LSAck | acknowledgment |

### Link-state operation
1. Establish neighbour adjacencies (Hello).
2. Exchange **LSAs** (link-state advertisements).
3. Build the **LSDB** (link-state database = topology table).
4. Run **SPF** → SPF tree.
5. Put the best routes in the routing table.

**Neighbour states:** Down → Init → Two-Way → ExStart → Exchange → Loading → **Full**.

**DR/BDR:** on multi-access networks (Ethernet), routers elect a **Designated Router** and a **Backup** to reduce LSA flooding. Others become **DROTHER**s. Highest **priority** wins (default 1; 0 = never), then highest router ID.

### Configuration
```
! OSPFv2 (IPv4)
R1(config)# router ospf 10
R1(config-router)# router-id 1.1.1.1
R1(config-router)# network 10.10.1.0 0.0.0.255 area 0      ← wildcard mask!
R1(config-router)# passive-interface g0/0/1                ← no hellos toward LAN hosts
R1(config-router)# default-information originate            ← share the default route
R1(config-router)# auto-cost reference-bandwidth 1000
!   or per interface:
R1(config-if)# ip ospf 10 area 0
R1(config-if)# ip ospf cost 30
R1(config-if)# ip ospf priority 255                         ← become DR

R1# show ip ospf neighbor
R1# show ip route ospf
R1# show ip protocols

! OSPFv3 (IPv6)
R1(config)# ipv6 unicast-routing
R1(config)# ipv6 router ospf 10
R1(config-rtr)# router-id 1.1.1.1
R1(config-if)# ipv6 ospf 10 area 0
```

**Wildcard mask** = the inverse of the subnet mask: `255.255.255.0` → `0.0.0.255`. 0 bits = "must match", 1 bits = "don't care". Quick rule: `255.255.255.255 − subnet mask`.

---

## 5. Access Control Lists (ACLs)

An **ACL** is an ordered list of `permit`/`deny` rules that filters traffic by addresses, protocols, and ports.

> 🚪 **Analogy:** a bouncer with a list. Checks each guest against the list **from top to bottom**; the **first matching** rule decides. If you're not on the list at all, you're turned away (**implicit deny any** at the end).

### Types
| Type | Number range | Filters on | Place it… |
|---|---|---|---|
| **Standard** | 1–99, 1300–1999 | **source IP only** | **close to the destination** |
| **Extended** | 100–199, 2000–2699 | source + destination IP, protocol, ports | **close to the source** |
| Named | a name instead of a number | either | |

**Why the placement?** Standard ACLs only know the source, so near the source they'd block that host from *everything*. Extended ACLs are precise, so dropping unwanted traffic early saves bandwidth.

**Rules of thumb:** one ACL per interface, per direction (in/out), per protocol. Put the **most specific** rules first.

### Configuration
```
! Standard: block host 192.168.10.10, allow everyone else
R1(config)# access-list 10 deny host 192.168.10.10
R1(config)# access-list 10 permit any
R1(config)# interface g0/1
R1(config-if)# ip access-group 10 out

! Extended named: allow only web from 192.168.10.0/24 to server 10.1.1.5
R1(config)# ip access-list extended WEB-ONLY
R1(config-ext-nacl)# permit tcp 192.168.10.0 0.0.0.255 host 10.1.1.5 eq 80
R1(config-ext-nacl)# permit tcp 192.168.10.0 0.0.0.255 host 10.1.1.5 eq 443
R1(config-ext-nacl)# deny ip any any                       ← (implicit anyway)
R1(config)# interface g0/0
R1(config-if)# ip access-group WEB-ONLY in

! Restrict SSH/Telnet access to the router itself
R1(config)# line vty 0 4
R1(config-line)# access-class 10 in

R1# show access-lists
```
Keywords: `host x.x.x.x` = `x.x.x.x 0.0.0.0`; `any` = `0.0.0.0 255.255.255.255`; `eq`, `gt`, `lt`, `range` for ports; `established` lets return TCP traffic back in.

**IPv6 ACLs:** always named, use prefix lengths, applied with `ipv6 traffic-filter`. Their implicit rules permit ICMPv6 neighbour discovery (`permit icmp any any nd-na`, `nd-ns`) before the final `deny ipv6 any any`.

---

## 6. NAT

**NAT (Network Address Translation)** translates **private** IPv4 addresses to **public** ones so many devices can share a few public addresses. It conserves IPv4 addresses and hides the internal network.

> 🏢 **Analogy:** an office with one public phone number and a receptionist. Outgoing calls all show the main number; the receptionist remembers who called whom so replies get to the right desk.

### Terminology
| Term | Meaning |
|---|---|
| **Inside local** | the private address of an inside host (192.168.1.10) |
| **Inside global** | the public address representing it (209.165.200.5) |
| Outside global | the real public address of the outside host |
| Outside local | how the outside host appears to the inside (usually the same) |

### Types
| Type | Mapping | Use |
|---|---|---|
| **Static NAT** | 1 private ↔ 1 public, permanent | servers that must be reachable from outside |
| **Dynamic NAT** | private → public from a **pool**, first come first served | |
| **PAT / NAT overload** | many private → **one** public, distinguished by **port numbers** | home routers, the most common |

```
! Static NAT
R1(config)# ip nat inside source static 192.168.1.10 209.165.200.5
! Dynamic NAT
R1(config)# ip nat pool PUBLIC 209.165.200.8 209.165.200.15 netmask 255.255.255.240
R1(config)# access-list 1 permit 192.168.0.0 0.0.255.255
R1(config)# ip nat inside source list 1 pool PUBLIC
! PAT using the outside interface's address
R1(config)# ip nat inside source list 1 interface g0/1 overload
! Mark the interfaces
R1(config)# interface g0/0
R1(config-if)# ip nat inside
R1(config)# interface g0/1
R1(config-if)# ip nat outside
R1# show ip nat translations
```

**Port forwarding:** send outside traffic arriving on a specific port to an inside server (e.g. public:8080 → 192.168.1.10:80).
```
R1(config)# ip nat inside source static tcp 192.168.1.10 80 209.165.200.5 8080
```

**NAT pros:** saves addresses, flexibility, hides internal addressing. **Cons:** breaks end-to-end addressing, adds delay, complicates some protocols (VoIP, IPsec) and tracing.

**NAT64:** translates between IPv6 and IPv4 networks during the transition. (IPv6 doesn't need NAT for address conservation.)

---

## 7. VPN & IPsec

A **VPN** creates a private, encrypted tunnel over a public network (the internet).

> 🚇 **Analogy:** a private underground tunnel between two buildings. You travel across the city (internet) but nobody on the street can see you.

**Benefits:** cost savings (no leased lines), security, scalability, compatibility.

**Types:**
- **Site-to-site:** connects whole networks (branch ↔ HQ); the end hosts don't know about it.
- **Remote access:** an individual user connects to the company network (client software or a clientless SSL/web VPN).
- Others: GRE over IPsec, DMVPN (dynamic spoke-to-spoke), MPLS VPNs.

### IPsec
A framework of protocols providing:
| Service | How |
|---|---|
| **Confidentiality** | encryption (AES, 3DES; avoid DES) |
| **Integrity** | hashing (SHA; MD5 is weak) |
| **Origin authentication** | pre-shared keys (PSK) or RSA certificates |
| **Secure key exchange** | **Diffie-Hellman** (DH groups) |

**Protocols:** **AH** (authentication header: integrity and authentication, **no encryption**) and **ESP** (encapsulating security payload: encryption **plus** integrity, the one normally used).

**Modes:**
| Mode | What's protected | Use |
|---|---|---|
| **Transport** | only the payload; the original IP header stays visible | host-to-host |
| **Tunnel** | the **entire original packet** wrapped in a new IP header | site-to-site VPN gateways |

**IKE (Internet Key Exchange)** negotiates the security associations (SAs) in two phases: phase 1 builds a secure management channel; phase 2 sets up the IPsec SAs for data.

---

## 8. Network Management (CDP, LLDP, NTP, Syslog, SNMP)

| Tool | Purpose | Key commands |
|---|---|---|
| **CDP** | Cisco proprietary: discover directly connected Cisco devices | `show cdp neighbors detail`, `no cdp run` |
| **LLDP** | vendor-neutral version of CDP | `lldp run`, `show lldp neighbors` |
| **NTP** | synchronise clocks (essential for accurate logs and certificates); **stratum** = distance from an authoritative clock | `ntp server 10.1.1.1`, `show ntp status` |
| **Syslog** | send log messages to a central server | `logging host 10.1.1.20`, `logging trap warnings` |
| **SNMP** | monitor and manage devices remotely | see below |

> 📹 **Analogy:** syslog = CCTV recordings, SNMP = a dashboard with gauges, NTP = making sure every camera's clock shows the same time so the recordings line up.

**Syslog severity levels (0 = most severe):**
`0 Emergency · 1 Alert · 2 Critical · 3 Error · 4 Warning · 5 Notification · 6 Informational · 7 Debugging`
💡 *Memory trick:* "**E**very **A**wesome **C**isco **E**ngineer **W**ill **N**eed **I**ce cream **D**aily".

### SNMP
- **Manager** (NMS software) ↔ **agents** (on devices) ↔ the **MIB** (Management Information Base, a database of variables identified by OIDs).
- Operations: `get` (read), `set` (change), **trap** (the agent alerts the manager unprompted), inform (a trap with acknowledgment).
- Versions: **v1/v2c** use plain-text "community strings" (like passwords). **v3** adds authentication and encryption ✅.

### Device maintenance
- **Back up configs** to a TFTP server or USB: `copy running-config tftp:`.
- **Password recovery:** interrupt boot → ROMMON → `confreg 0x2142` (ignore the startup config) → reset the password → `config-register 0x2102`.
- **IOS image management:** `copy tftp: flash:`, `boot system flash:<image>`.

---

## 9. Network Design & Troubleshooting

### Hierarchical design (three layers)
| Layer | Role |
|---|---|
| **Access** | connects end devices (switch ports, PoE, port security, VLANs) |
| **Distribution** | aggregates access switches; routing between VLANs, ACLs, policy |
| **Core** | high-speed backbone; fast and redundant, minimal processing |

Small networks often use a **collapsed core** (core + distribution combined).

> 🚗 **Analogy:** neighbourhood streets (access) → main roads (distribution) → highways (core).

**Design goals:** scalability, redundancy, performance, security, manageability. Switch selection: port density, speed, PoE, layer 2 vs layer 3, fixed vs modular, stackable.

### Troubleshooting process
1. Gather symptoms → 2. Isolate the problem → 3. Implement the fix → 4. Verify and document.

**Methods:** bottom-up, top-down, divide-and-conquer, follow-the-traffic-path, substitution, comparison (with a working config), educated guess.

**Tools:** `ping`, `traceroute`, `show` commands, `debug`, cable testers, protocol analysers (Wireshark), syslog, a network **baseline** (what's "normal" performance), and **documentation** (topology diagrams, IP addressing tables).

**Common symptoms by layer:**
- Physical: bad cable, interface down, CPU overload
- Data link: encapsulation mismatch, STP loops, VLAN mismatch
- Network: wrong IP/mask/gateway, routing issues, ACLs blocking traffic
- Transport: ACLs blocking ports, NAT issues
- Application: DNS failure, service down

---

## 10. Cloud, Virtualization, SDN & Automation

### Cloud computing
**Service models:**
| Model | You manage | Provider manages | Example |
|---|---|---|---|
| **SaaS** | just use the app | everything | Gmail, Microsoft 365 |
| **PaaS** | your code and data | OS, runtime, hardware | Heroku, Google App Engine |
| **IaaS** | OS, apps, data | hardware, network, virtualisation | AWS EC2, Azure VMs |

> 🍕 **Pizza analogy:** IaaS = buy the ingredients and use a rented kitchen; PaaS = order a take-and-bake pizza; SaaS = the pizza is delivered, ready to eat.

**Deployment models:** public, private, **hybrid**, community.
**Cloud vs data centre:** a data centre is a physical facility; the cloud is an on-demand service, often running *in* data centres.

### Virtualization
Run multiple **virtual machines** on one physical server using a **hypervisor**.
- **Type 1 (bare-metal):** runs directly on the hardware (VMware ESXi, Hyper-V, KVM); used in data centres.
- **Type 2 (hosted):** runs on top of an OS (VirtualBox, VMware Workstation).

**Benefits:** server consolidation (less hardware, power, and space), faster provisioning, easy testing, less downtime.

### Software-Defined Networking (SDN)
Traditional devices combine:
- **Control plane** ("the brain"): decides where traffic goes (routing tables, STP).
- **Data plane** ("the muscle"): actually forwards the packets.

**SDN separates them:** a central **SDN controller** holds the control plane for the whole network, and devices just forward. It uses **northbound APIs** (to applications) and **southbound APIs** (to devices, e.g. OpenFlow).

> ✈️ **Analogy:** instead of every pilot deciding their own route, air traffic control (the controller) plans the routes and the pilots fly them.

Cisco examples: **ACI** (Application Centric Infrastructure) and **DNA Center**.

### Automation
Configure networks with code instead of typing CLI commands device by device.
- **Data formats:** **JSON**, **XML**, **YAML**.
  ```json
  { "interface": { "name": "GigabitEthernet0/0", "ip": "192.168.1.1", "mask": "255.255.255.0" } }
  ```
- **APIs:** **REST** (HTTP: GET, POST, PUT, DELETE, often returning JSON), NETCONF, RESTCONF.
- **Configuration management tools:** **Ansible** (agentless, YAML playbooks), **Puppet**, **Chef**, **SaltStack**.
- **Intent-Based Networking (IBN):** describe *what* you want ("guest Wi-Fi can't reach finance servers") and the system works out *how*.

---

## 11. Network Security Threats & Best Practices

### Threat actors
Script kiddies, hacktivists, organised cybercriminals, state-sponsored attackers, insider threats.

### Malware
| Type | Behaviour |
|---|---|
| **Virus** | attaches to legitimate files; needs the user to run it |
| **Worm** | self-replicates across networks **without** user action |
| **Trojan horse** | disguised as legitimate software |
| Ransomware | encrypts files and demands payment |
| Spyware / adware / rootkit / botnet | spy, show ads, hide deep in the system, zombie networks |

### Network attacks
| Category | Examples |
|---|---|
| **Reconnaissance** | ping sweeps, port scans, information queries (WHOIS) |
| **Access** | password attacks (brute force, dictionary), spoofing, **man-in-the-middle**, trust exploitation, buffer overflow |
| **DoS / DDoS** | overwhelm a target so legitimate users can't get service (a DDoS uses a botnet) |
| **Social engineering** | phishing, spear phishing, pretexting, baiting, tailgating |
| IP/TCP/UDP attacks | ICMP attacks, TCP SYN flood, TCP reset, session hijacking, UDP flood, DNS/DHCP attacks |

### Best practices: **defence in depth**
Layers of security so one failure doesn't mean total compromise:
- Firewalls (stateful, next-gen), **IPS/IDS**, VPNs, ACLs
- AAA, strong passwords, MFA, least privilege
- Patching and updates, backups
- Network segmentation (VLANs, DMZ)
- Security awareness training
- Endpoint protection, ESA/WSA

> 🏰 **Analogy:** a castle with a moat, walls, gates, guards, and a locked treasure room. An attacker has to beat every layer.

### Securing communication (cryptography)
| Goal | Tool |
|---|---|
| **Integrity** | hash functions (SHA-256); MD5/SHA-1 are outdated |
| **Origin authentication** | **HMAC** (hash + secret key) |
| **Confidentiality** | **symmetric** encryption (AES: same key for both sides, fast) or **asymmetric** (RSA: public/private key pair, slower) |
| Key exchange | Diffie-Hellman |
| Digital signatures | sign with the private key, verify with the public key; certificates issued by a **CA** (PKI) |

> 🔑 **Asymmetric analogy:** a padlock you hand out freely (public key). Anyone can lock a box with it, but only you have the key that opens it (private key).

---

## Labs recap
The lab folders (Lab 2–10) build up one topology step by step. Typical skills:
- **Lab 2–3:** multilayer switch (DSW) config, VLANs, trunks, SVIs, IPv4 addressing tables.
- **Lab 3–4:** STP root bridge selection, **EtherChannel** between distribution switches and routers (R1–R3).
- **Lab 4–6:** **OSPF** between routers and an ISP, default routes, passive interfaces.
- **Lab 6–7:** **ACLs** to restrict traffic and management access.
- **Lab 8–9:** **NAT/PAT** on the edge router toward the ISP, DHCP, network management.

**Verification habit:** after each change, run `show ip interface brief`, `show vlan brief`, `show etherchannel summary`, `show ip ospf neighbor`, `show ip route`, `show access-lists`, `show ip nat translations`, then `ping` end to end.

---

### ✅ Quick self-check
1. How does STP choose the root bridge? *(lowest bridge ID: priority, then MAC)*
2. LACP `active` + `passive`: does a channel form? *(yes)*
3. What's the wildcard mask for /26? *(0.0.0.63)*
4. Where should a standard ACL be placed? *(close to the destination)*
5. Which NAT type lets a whole home network share one public IP? *(PAT/overload)*
6. Which IPsec mode encrypts the whole original packet? *(tunnel mode)*
7. Syslog level 4 is…? *(Warning)*
