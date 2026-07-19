# 🏦 Highly Available & Secure Banking Infrastructure (HSRP & PCI-DSS VLANs)

## 📌 Project Overview
This project simulates an enterprise-grade, highly fault-tolerant core banking network infrastructure designed to meet **PCI-DSS (Payment Card Industry Data Security Standard)** compliance using **Cisco Packet Tracer**. The architecture eliminates single points of failure (SPOF) at the network gateway layer by deploying **Hot Standby Router Protocol (HSRP)** across redundant edge routers.

To safeguard sensitive financial transactions, the local area network enforces strict Layer 2 broadcast segmentation, isolating ATM machines and Core Banking Database servers from general branch staff and managerial traffic.

---

## 🏗️ Architectural & Security Highlights
* **First Hop Redundancy Protocol (HSRP):** Engineered an Active/Standby gateway failover cluster using Cisco HSRP. Configured a shared Virtual Gateway IP (`.1`) across redundant routers (`Router0` as Active with Priority 110, `Router1` as Standby with Priority 100) to ensure zero-downtime failover for critical banking operations.
* **Preemption Enabled:** Configured `standby preempt` on the primary active router to ensure automatic reclamation of the Active Gateway role upon recovery from an outage or hardware link failure.
* **PCI-DSS VLAN Segmentation:** Divided the banking network into strict security zones: **Branch Staff (VLAN 10)**, **ATM Transaction Zone (VLAN 20)**, and the **Core Banking Data Server Zone (VLAN 30)** to prevent unauthorized horizontal lateral movement.
* **Hierarchical Switch Topology:** Deployed a central distribution switch (`Switch1`) interconnecting dual access layer switches (`Switch0` and `Switch2`) via dedicated IEEE 802.1Q trunk links.
* **Router-on-a-Stick (ROAS) Integration:** Implemented 802.1Q sub-interface encapsulation on both HSRP routers to simultaneously route and protect multi-VLAN traffic across redundant links.

---

## 🧮 HSRP & IP Addressing Schema

| VLAN ID | Security Zone / Dept | Network Subnet | Virtual Gateway IP (HSRP) | R1 Active IP (Pri: 110) | R2 Standby IP (Pri: 100) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | **Branch Staff (Manager/Cashier)** | `192.168.10.0/24` | `192.168.10.1` | `192.168.10.2` | `192.168.10.3` |
| **VLAN 20** | **ATM Machine (PCI-DSS Zone)** | `192.168.20.0/24` | `192.168.20.1` | `192.168.20.2` | `192.168.20.3` |
| **VLAN 30** | **Core Data Server Zone** | `192.168.30.0/24` | `192.168.30.1` | `192.168.30.2` | `192.168.30.3` |

---

## 🔍 NOC Verification & Troubleshooting Commands
To audit HSRP state transitions, active router elections, and VLAN trunking operations, the following CLI commands were utilized:

```text
! Verify HSRP Active/Standby state, Virtual IP bindings, and Preempt status
Router# show standby
Router# show standby brief

! Verify active 802.1Q trunk ports and VLAN database on distribution switch
Switch# show interfaces trunk
Switch# show vlan brief

! Audit sub-interface IP status and encapsulation metrics
Router# show ip interface brief
