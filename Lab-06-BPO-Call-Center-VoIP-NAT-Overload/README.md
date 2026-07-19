# 🎧 Enterprise BPO Call Center Infrastructure (High-Density VoIP, VLSM /26 & NAT Overload)

## 📌 Project Overview
This project simulates a mission-critical, high-density **Business Process Outsourcing (BPO) Call Center** network architecture designed in **Cisco Packet Tracer**. The infrastructure supports multi-floor agent operations where agent workstations and **Cisco 7960 IP Phones** operate simultaneously to handle continuous inbound and outbound customer campaigns.

To optimize IP space across dense seating floors, the network schema utilizes **Variable Length Subnet Masking (VLSM /26)**. Centralized IP address distribution for both data and voice endpoints is managed by a core server in the BPO Data Center (`VLAN 30`) via **DHCP Relay Agents (`ip helper-address`)** configured across Router-on-a-Stick (ROAS) sub-interfaces.

To ensure uninterrupted access to international cloud CRMs and VoIP SIP trunks without exposing internal agent IPs, the edge gateway router enforces **NAT Overload (Port Address Translation - PAT)** over an enterprise serial WAN link.

---

## 🏗️ Architectural & Technical Highlights
* **High-Density BPO Floor Segmentation:** Carved precise `/26` (`255.255.255.192`) subnets to isolate operational call campaigns: **Campaign Floor 1 (VLAN 10)**, **Campaign Floor 2 (VLAN 20)**, and the **Dialer/Server Farm (VLAN 30)** without IP wastage.
* **VoIP & Unified Communications:** Daisy-chained agent workstations through Cisco IP Phones, simulating an enterprise call center deployment with voice traffic prioritized across access and distribution switches.
* **Centralized DHCP Relay Engine:** Deployed a primary DHCP Server (`192.168.10.130`) in the server room and configured `ip helper-address` on edge gateway sub-interfaces (`Gig0/0.10` and `Gig0/0.20`) to relay agent broadcast requests across Layer 3 boundaries.
* **NAT Overload (PAT) for Cloud CRM Access:** Implemented `ip nat inside` on internal 802.1Q sub-interfaces and `ip nat outside` on the ISP-facing Serial link (`Serial0/3/0`). Bound internal access lists to the public interface using the `overload` keyword for simultaneous multi-agent NAT translation.
* **Multilayer Core Distribution Hub:** Deployed a **Cisco 3650 Multilayer Core Switch** as a high-speed backplane aggregator to switch tagged 802.1Q trunks between campaign floor switches and the edge router.
* **WAN Edge Routing:** Configured a point-to-point `/30` serial WAN link connecting the BPO facility to an upstream ISP internet gateway (`8.8.8.8/32` loopback simulation) via a static default route (`0.0.0.0/0`).

---

## 🧮 BPO Subnetting & Campaign Schema

| VLAN ID | BPO Operational Zone | VLSM Subnet (/26) | Gateway Sub-interface IP | Network Role & Rule |
| :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | **Operations Floor 1 (Inbound Campaign)** | `192.168.10.0/26` | `192.168.10.1` (Gig0/0.10) | `ip helper-address` + `ip nat inside` |
| **VLAN 20** | **Operations Floor 2 (Outbound Campaign)** | `192.168.10.64/26` | `192.168.10.65` (Gig0/0.20) | `ip helper-address` + `ip nat inside` |
| **VLAN 30** | **Data Center (Dialer / Recording Farm)** | `192.168.10.128/26` | `192.168.10.129` (Gig0/0.30) | Central DHCP Server + `ip nat inside` |
| **WAN Link** | **Enterprise ISP Serial Link** | `200.1.1.0/30` | `200.1.1.1` (Serial0/3/0) | Public WAN Interface (`ip nat outside`) |

---

## 🛡️ Key Gateway Router Configuration (DHCP Relay & PAT)

```text
! Inter-VLAN Sub-interfaces with Centralized DHCP Relay & NAT Inside
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.192
 ip helper-address 192.168.10.130
 ip nat inside
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.10.65 255.255.255.192
 ip helper-address 192.168.10.130
 ip nat inside
!
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.10.129 255.255.255.192
 ip nat inside

! WAN Serial Interface with NAT Outside
interface Serial0/3/0
 ip address 200.1.1.1 255.255.255.252
 ip nat outside
 clock rate 2000000

! NAT Overload (PAT) Binding & Default Internet Routing
access-list 1 permit any
ip nat inside source list 1 interface Serial0/3/0 overload
ip route 0.0.0.0 0.0.0.0 200.1.1.2
