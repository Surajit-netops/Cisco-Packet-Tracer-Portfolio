
# 🏢 Enterprise Multi-Site Network Architecture: GRE Tunneling, OSPFv2, Inter-VLAN Routing & L2 Security

## 📌 Project Overview
This project showcases a complete, production-grade enterprise network topology simulated in **Cisco Packet Tracer**. The infrastructure connects a **Headquarters (HQ)** site to a remote **Branch Office** across an untrusted Internet Service Provider (ISP) WAN cloud. 

To achieve automated routing, data confidentiality, broadcast control, and endpoint security, the architecture integrates **Generic Routing Encapsulation (GRE) Tunnels**, **Single-Area OSPFv2 Dynamic Routing**, **Inter-VLAN Routing via Multilayer Switching**, **PAT Overload**, and **Layer 2 Sticky MAC Port-Security**.

---

## 🎯 Key Technologies & Protocols Implemented

### 🌐 Layer 3 Routing & WAN Connectivity
*   **GRE Tunneling (`Tunnel0`):** Established a virtual point-to-point private tunnel across the public WAN ISP link (`10.10.10.0/30`) to securely route internal subnets between HQ and Branch.
*   **Dynamic Routing (OSPFv2 Area 0):** Configured automated route discovery and adjacency maintenance across WAN gateways, Core switches, and the GRE tunnel.
*   **NAT / PAT Overload:** Implemented Port Address Translation on `Router0` using standard ACLs to mask internal RFC 1918 private IP addresses (`192.168.x.x`) behind a single public WAN IP for internet access.
*   **Inter-VLAN Routing:** Activated Layer 3 routing (`ip routing`) on the Core Multilayer Switch using Switch Virtual Interfaces (SVIs) as default gateways for departmental VLANs.

### ⚡ Layer 2 Switching & Automation
*   **VLAN Segmentation:** Dedicated VLANs isolated for HR (VLAN 10), Sales (VLAN 20), IT (VLAN 30), Server Farm (VLAN 40), WLAN (VLAN 50), and Management (VLAN 99).
*   **Centralized DHCP Relay (`ip helper-address`):** Configured broadcast relay agents across all SVIs pointing to the centralized DHCP/DNS Server (`192.168.40.2`).
*   **Spanning-Tree Protocol Optimization:** Implemented **Rapid PVST+** for fast convergence and manually tuned bridge priorities (`priority 24576`) to elect the Core Multilayer Switch as the **Root Bridge** for all VLANs.
*   **VLAN Trunking & Pruning:** Configured IEEE 802.1Q trunk links with explicit `allowed vlan` lists to prevent broadcast storms and eliminate unnecessary VLAN flooding.

### 🛡️ Network Security & Monitoring
*   **Layer 2 Port Security:** Implemented `switchport port-security mac-address sticky` on access ports (e.g., Fa0/2) to dynamically lock user endpoints and mitigate unauthorized rogue device connections.
*   **Layer 3 Traffic Filtering (Extended ACLs):** Configured Extended Access List `110` on the Multilayer Core to restrict lateral movement (e.g., blocking HR VLAN 10 from directly accessing sensitive Server Farm subnets in VLAN 40).
*   **VLAN Hardening:** Administratively disabled default `Vlan1` across access switches to prevent VLAN hopping vulnerabilities.
*   **SNMP Monitoring:** Configured Read-Only (RO) SNMP community strings (`community cisco RO`) on core switches for integration with external network monitoring systems (NMS).

---

## 🗺️ Master IP Addressing & Device Schema

### 1. WAN Gateways & GRE Tunnel Topology
| Device Name | Interface | IP Address / Subnet | Role & Configuration Highlights |
| :--- | :--- | :--- | :--- |
| **HQ Router (`Router0`)** | `Gig0/0` | `192.168.99.1 /24` | Core LAN Link (`ip nat inside`) |
| | `Gig0/1` | `200.1.1.1 /30` | Public WAN Link to ISP (`ip nat outside`) |
| | `Tunnel0` | `10.10.10.1 /30` | GRE Tunnel Source (`Gig0/1`) -> Dest (`200.1.1.6`) |
| **Branch Router (`Router2`)** | `Gig0/0` | `200.1.1.6 /30` | Public WAN Link to ISP |
| | `Gig0/1` | `192.168.60.1 /24` | Branch Internal LAN Gateway |
| | `Tunnel0` | `10.10.10.2 /30` | GRE Tunnel Source (`Gig0/0`) -> Dest (`200.1.1.1`) |
| **ISP WAN (`Router1`)** | `Gig0/0` & `Gig0/1` | `200.1.1.2/30` & `200.1.1.5/30`| Transparent ISP Cloud running OSPF Area 0 |

### 2. HQ VLAN & Subnet Allocation (Core Multilayer Switch)
| VLAN ID | VLAN Name | SVI Gateway IP | Connected Endpoints & Services |
| :---: | :--- | :--- | :--- |
| **VLAN 10** | HR Dept | `192.168.10.1 /24` | End-user PCs (Protected via Sticky MAC Port-Security) |
| **VLAN 20** | Sales Dept | `192.168.20.1 /24` | Sales departmental endpoints |
| **VLAN 30** | IT Dept | `192.168.30.1 /24` | IT administrative systems |
| **VLAN 40** | Server Farm | `192.168.40.1 /24` | Central DHCP & DNS Server (`192.168.40.2`) |
| **VLAN 50** | WLAN / Wireless | `192.168.50.1 /24` | WRT300N Wireless AP, Laptops, Mobile Devices |
| **VLAN 99** | Management | `192.168.99.2 /24` | Native VLAN & Switch Management (`192.168.60.2` on Access) |

---

## 🛠️ Deep-Dive Architecture Highlights

### 1. GRE Tunneling with OSPF Over WAN
To enable seamless inter-site routing without exposing private LAN subnets to the public ISP, a GRE tunnel was dynamically encapsulated and advertised within OSPF Area 0:
```text
! HQ Router0 GRE Configuration
interface Tunnel0
 ip address 10.10.10.1 255.255.255.252
 tunnel source GigabitEthernet0/1
 tunnel destination 200.1.1.6
!
router ospf 1
 network 10.10.10.0 0.0.0.3 area 0
 network 192.168.99.0 0.0.0.255 area 0
 network 200.1.1.0 0.0.0.3 area 0
```

### 2. Multilayer Switch DHCP Relay & Root Bridge Tuning
The core switch was engineered to act as the primary Layer 3 gateway, relaying broadcast DHCP requests to VLAN 40 while asserting root bridge dominance:
```text
ip routing
spanning-tree mode rapid-pvst
spanning-tree vlan 1,10,20,30,40,50,99 priority 24576
!
interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.40.2
```

### 3. Layer 2 Access Port Hardening (Sticky MAC)
Access ports connected to critical endpoints were locked down to prevent unauthorized rogue MAC addresses:
```text
interface FastEthernet0/2
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
```

---

## ✅ Verification & Testing Checklist
*   [x] **Inter-Site Ping:** Successful ICMP echo requests between HQ VLAN 10 (`192.168.10.x`) and Branch LAN (`192.168.60.x`) traversing virtual `Tunnel0`.
*   [x] **NAT Overload Verification:** Executed `show ip nat translations` on `Router0` confirming private inside-local addresses dynamically mapping to `200.1.1.1`.
*   [x] **DHCP Relay Validation:** Wireless devices on VLAN 50 successfully received dynamic leases (`192.168.50.x`) from `Server0` located in VLAN 40.
*   [x] **Spanning-Tree Health:** Verified via `show spanning-tree` that the Multilayer Switch serves as Root Bridge with rapid convergence enabled.

---
*Developed as part of an advanced networking portfolio demonstrating core competencies required for Network Operations Center (NOC) and Enterprise Network Engineering roles.*
