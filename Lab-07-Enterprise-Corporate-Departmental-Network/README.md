# 🏢 Enterprise Corporate Departmental Infrastructure (VLAN Segmentation & Core Switching)

## 📌 Project Overview
This project simulates a professional **Enterprise Corporate Network** architecture designed in **Cisco Packet Tracer**. The network focuses on departmental segmentation between the **IT Department** and the **HR Department**, ensuring secure inter-departmental communication through a centralized **Cisco 3650 Multilayer Core Switch**.

To optimize resource utilization, the infrastructure uses **VLSM (Variable Length Subnet Masking)**, providing precise IP allocation for different departments while preventing address wastage. Routing between these departments is managed at the distribution/core layer, demonstrating robust internal network design standards.

---

## 🏗️ Architectural & Technical Highlights
* **Departmental VLAN Segmentation:** Isolated the IT and HR departments into distinct network segments to improve broadcast domain efficiency and security.
* **VLSM /26 Subnetting:** Engineered a precise IP allocation schema:
    * **IT Department:** `192.168.1.64/26`
    * **HR Department:** `192.168.1.128/26`
* **Multilayer Core Switching:** Deployed a **Cisco 3650-24PS Switch** to act as the central distribution hub, handling inter-VLAN routing (SVI) at wire speed.
* **Inter-VLAN Routing:** Configured Switch Virtual Interfaces (SVIs) on the Core Switch to facilitate routing between IT (`VLAN 20`) and HR (`VLAN 30`) without requiring an external router hop.
* **Scalable Topology:** Designed for growth, allowing additional departments to be added with minimal reconfiguration of the core switching logic.

---

## 🧮 IP Allocation Schema

| Department | Network | Subnet Mask | Gateway (SVI) | VLAN ID |
| :--- | :--- | :--- | :--- | :--- |
| **IT Department** | `192.168.1.64/26` | `255.255.255.192` | `192.168.1.65` | VLAN 20 |
| **HR Department** | `192.168.1.128/26` | `255.255.255.192` | `192.168.1.129` | VLAN 30 |

---

## 🛡️ Core Switch SVI Configuration
The Core Switch handles inter-departmental traffic via pre-configured SVIs:

```text
! IT Department SVI
interface Vlan20
 ip address 192.168.1.65 255.255.255.192
 no shutdown
!
! HR Department SVI
interface Vlan30
 ip address 192.168.1.129 255.255.255.192
 no shutdown
!
! Enable Routing on 3650 Core Switch
ip routing
