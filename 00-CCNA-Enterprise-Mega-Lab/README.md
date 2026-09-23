# 🏢 Enterprise Multi-Tier Campus Network (CCNA Mega Lab)

A complete end-to-end enterprise network deployment designed and validated in Cisco Packet Tracer based on the **Jeremy's IT Lab CCNA 200-301** curriculum. 

The architecture encompasses a dual-office campus (Office A & Office B) connected through a redundant Layer 3 Core, terminated by an Edge Router (R1) providing dual-homed Internet connectivity.

---

## 🗺️ Topology Diagram
![Enterprise Network Topology](./NetworkTopology)

---

## 📑 Network Concepts & Technologies Covered

| Domain | Implemented Technologies |
| :--- | :--- |
| **Layer 2 Infrastructure** | VLAN Trunking (802.1Q), VTPv2, PAgP & LACP EtherChannels, DTP Disablement |
| **Spanning Tree Protocol** | Rapid-PVST+ (802.1w), Root Bridge Alignment, PortFast, BPDU Guard |
| **Layer 3 & First-Hop Redundancy** | Layer 3 Routed Port-Channels, SVI Inter-VLAN Routing, HSRPv2 Gateway Load Sharing |
| **Dynamic & Static Routing** | Single-Area OSPFv2 (Point-to-Point, Passive Interfaces), Floating Static IPv4/IPv6 Default Routes |
| **Network Services** | Cisco IOS DHCP Server, DHCP Relay (`ip helper-address`), DNS Mapping, NTPv3 Authentication, SNMP RO, Syslog Logging |
| **Infrastructure Hardening & Security** | Scrypt Type-9 & MD5 Type-5 Hashing, Standard & Extended Named ACLs, Port Security (Sticky MACs), DHCP Snooping (Rate Limiting), Dynamic ARP Inspection (DAI) |
| **Wireless LAN Controller** | Centralized Lightweight AP (LWAP) Architecture, Dynamic Interfaces, WPA2-PSK (AES) Encryption |
| **IPv6 Integration** | Global Unicast & Link-Local Addressing, EUI-64 Autoconfiguration, Fully-Specified Floating Routes |

---

## 🏗️ Architecture Breakdown

### 1. Layer 2 Trunking, VTP & Aggregation
* **Native VLAN Security:** Dot1Q trunks configured with unused Native VLAN 1000 and DTP disabled via `switchport nonegotiate` to mitigate VLAN hopping attacks.
* **VTP Synchronization:** `DSW-A1` and `DSW-B1` operate as VTPv2 Servers in domain `Jeremy's IT Lab`, propagating VLANs (PCs, Phones, Wi-Fi/Servers, Management) across VTP Clients (`ASW` layer).
* **Link Bundles:**
  * **Office A Distribution:** PAgP EtherChannel (`Channel-group 1 mode desirable`).
  * **Office B Distribution:** LACP EtherChannel (`Channel-group 1 mode active`).

### 2. Spanning Tree & HSRP Topology Alignment
* Configured **Rapid-PVST+** across all distribution and access switches.
* Root bridge priorities were deterministically aligned with HSRP active gateways (Priority `0` for primary/active, `4096` for secondary/standby):
  * **Office A:** `DSW-A1` is Active/Root for VLANs 10 & 99; `DSW-A2` is Active/Root for VLANs 20 & 40.
  * **Office B:** `DSW-B1` is Active/Root for VLANs 10 & 99; `DSW-B2` is Active/Root for VLANs 20 & 30.
* Edge ports hardened with `spanning-tree portfast` and `spanning-tree bpduguard enable`.

### 3. Dynamic Routing & Core Connectivity
* **Core Interconnect:** Routed L3 EtherChannel (`Port-Channel 1`) running between `CSW1` and `CSW2`.
* **OSPFv2 (Process 1, Area 0):** Physical transit links tuned to `ip ospf network point-to-point` to eliminate unnecessary DR/BDR elections. All SVIs (except management VLAN 99) and loopback interfaces configured as `passive-interface`.
* **Default Route Redistribution:** Edge router `R1` injects default route upstream via `default-information originate` with floating fallback route (AD 2).

### 4. Edge NAT & Layer 2 Security
* **Static NAT & PAT:** 1:1 Static NAT for Internal Server 1 (`10.5.0.4` -> `203.0.113.13`) and Overload PAT (`pool1`) for client subnet Internet breakout.
* **Access Hardening:** Port Security enabled with `violation restrict` and `mac-address sticky`. DHCP Snooping rate limits untrusted ports to 15 PPS (100 PPS on WLC) with Option 82 disabled. Dynamic ARP Inspection enabled with full validation checks (`dst-mac`, `src-mac`, `ip`).
