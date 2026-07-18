
# 🌐 Enterprise Multi-Area OSPFv2 Network Architecture & High Availability Design

## 📌 Project Overview
This project demonstrates a production-grade **Multi-Area OSPFv2 (Open Shortest Path First)** dynamic routing architecture simulated in **Cisco Packet Tracer**. The network is engineered for high availability, deterministic failover, and strict hierarchical segmentation across a central **Backbone Area (Area 0)** and two remote branch networks (**Area 10** and **Area 20**).

To prevent single points of failure (SPOF) and optimize convergence times, both branch areas feature **Triangular Redundancy Loops** utilizing dedicated transit routers.

---

## 🏗️ Architectural & Routing Highlights
* **Hierarchical OSPF Design:** Strictly separates inter-area routing using **Area Border Routers (ABRs)** connecting remote branch areas (Area 10 & 20) to the core Backbone Area 0 (`10.0.0.0/24`).
* **High Availability & Redundant Transit Paths:** Implemented triangular WAN loops within branch areas. If a primary serial link fails, OSPF automatically converges and re-routes traffic via dedicated transit nodes without dropping user sessions.
* **LSA Type Management:** Engineered smooth propagation of **Router LSAs (Type 1)** within local areas and **Summary LSAs (Type 3)** across ABR boundaries for seamless inter-area communication.
* **LAN Hardening & Optimization:** Configured `passive-interface` on user-facing GigabitEthernet switch ports (`192.168.x.x`) to suppress OSPF Hello broadcast packets, mitigating topology snooping and optimizing CPU utilization.

---

## 🧮 IP Addressing & OSPF Area Schema

| Router Name | Role / Type | Active Interfaces & IP Subnets | OSPF Area | Key Functionality |
| :--- | :--- | :--- | :--- | :--- |
| **Router0** | **ABR** (Top-Left) | `10.0.0.1/24` (WAN) <br> `20.0.0.1/24`, `40.0.0.1/24` (WAN) <br> `192.168.10.1/24` (LAN) | **Area 0** <br> **Area 10** | Bridges Area 10 to Backbone; generates Type 3 Summary LSAs. |
| **Router1** | **Internal** (Left-Corner) | `20.0.0.2/24`, `30.0.0.1/24` (WAN) <br> `192.168.20.1/24` (LAN) | **Area 10** | Handles branch LAN traffic & primary serial connectivity. |
| **Router2** | **Transit / HA** (Left-Bottom)| `30.0.0.2/24`, `40.0.0.2/24` (WAN) <br> *No LAN Attached* | **Area 10** | Dedicated redundancy node for failover routing in Area 10. |
| **Router3** | **ABR** (Top-Right) | `10.0.0.2/24` (WAN) <br> `50.0.0.1/24`, `70.0.0.1/24` (WAN) <br> `192.168.40.1/24` (LAN) | **Area 0** <br> **Area 20** | Bridges Area 20 to Backbone; maintains inter-area routing table. |
| **Router4** | **Transit / HA** (Right-Bottom)| `50.0.0.2/24`, `60.0.0.1/24` (WAN) <br> *No LAN Attached* | **Area 20** | Dedicated redundancy node for failover routing in Area 20. |
| **Router5** | **Internal** (Right-Corner)| `60.0.0.2/24`, `70.0.0.2/24` (WAN) <br> `192.168.30.1/24` (LAN) | **Area 20** | Handles branch LAN traffic & secondary serial link termination. |

---

## 🔍 NOC Verification & Troubleshooting Commands
To verify routing convergence and OSPF adjacency in this topology, the following industry-standard CLI commands were utilized:

```text
! Verify OSPF neighbor adjacencies and state (FULL/DR/BDR/Point-to-Point)
Router# show ip ospf neighbor

! Inspect the OSPF Link-State Database (LSDB) for Type 1, 2, and 3 LSAs
Router# show ip ospf database

! Filter routing table to display only OSPF learned routes (O, O IA)
Router# show ip route ospf

! Verify OSPF interface timer intervals and passive-interface status
Router# show ip ospf interface GigabitEthernet0/1
