<div align="center">
  <h1>🚀 Enterprise Ring Network Simulation: RIPv2, Multi-Site VoIP, Wireless & ACL</h1>
  <p>
    <b>Advanced Cisco Packet Tracer Simulation featuring a 4-Node Ring Backbone, RIPv2 Dynamic Routing, VoIP (CME), VLAN Segmentation, and Access Control Lists (ACL).</b>
  </p>
</div>

<br />

## 📸 Network Topology

<img width="716" height="376" alt="{5EC2D93E-D5C4-4CDA-B482-77208F302E8A}" src="https://github.com/user-attachments/assets/8389b79d-ffea-403b-8cab-0edd8a65aa92" />

## 📖 Project Overview
This repository contains a complete configuration guide and documentation for a mid-to-large scale Enterprise network topology. The backbone is designed using a **Ring Topology** across 4 routers to ensure high availability and redundancy. The network integrates centralized data center services, department segmentation (VLANs), dynamic wireless networks (DHCP), and cross-site IP telephony (VoIP) using Cisco CallManager Express (CME). 

---

## 🚀 Getting Started (How to Use)

### Prerequisites
To open and interact with this simulation, you will need:
* **Cisco Packet Tracer** (Version 8.0 or newer recommended)
* Basic understanding of Cisco IOS CLI

### Installation & Execution
1. **Open the Simulation:**
   Launch Cisco Packet Tracer and open the `.pkt` file included in this repository.
2. **Wait for Convergence:**
   Allow a few seconds for the Spanning Tree Protocol (STP) to transition switch ports to a forwarding state (green arrows) and for RIPv2 to exchange routing tables.
3. **Explore the Network:**
   You can hover over devices to see their IP allocations, open the CLI tabs of the routers to inspect configurations, or use the PC terminals to run ping and tracert commands.

---

## 💡 Architecture & Design Choices (The "Why")
To demonstrate enterprise-grade network engineering principles, several specific design choices were implemented:

1. **Ring Topology Backbone for Redundancy:** By connecting the 4 routers in a loop (`200.20...` -> `200.30...` -> `200.40...` -> `200.50...`), the network achieves high availability. If one WAN cable is severed, dynamic routing automatically recalculates and shifts traffic to the opposite side of the ring.
2. **RIPv2 over RIPv1:** RIPv2 was chosen because it supports **VLSM (Variable Length Subnet Masking)**. This is crucial since the architecture mixes `/30` subnets for point-to-point WAN links (saving IP space) and `/24` subnets for local LANs.
3. **Router-on-a-Stick (802.1Q Trunking):** Instead of using multiple physical router interfaces for Voice, Data, and Wireless networks, VLANs are trunked into a single Gigabit interface using sub-interfaces. This drastically reduces hardware costs and cable clutter.
4. **DHCP Option 150 for VoIP:** IP Phones require a TFTP server to download their configuration and firmware. Option 150 is explicitly configured in the DHCP pool to direct the phones to the router's IP address where the `telephony-service` is hosted.
5. **Dedicated Voice VLANs:** Segregating voice traffic (VLAN 50) from data traffic prevents broadcast storms and allows for future implementation of QoS (Quality of Service) to prioritize voice packets and prevent audio jitter.

---

## 🗺️ IP Allocation & Segmentation Table

| Site / Zone | VLAN / Service | Subnet / Network | Description & Purpose |
| :--- | :--- | :--- | :--- |
| **Site 1 (Top-Left)** | OB Premium / ACL | `192.50.33.0/24` | Data LAN with Access Control List implementation. |
| **Site 2 (Top-Right)** | Server Center | `192.70.33.0/24` | Centralized Servers (`Qyv.com` at `192.70.33.2`). |
| **Site 2 (Top-Right)** | Printer Network | `192.33.60.0/24` | Dedicated VLAN for Network Printers. |
| **Site 3 (Bot-Left)** | Data LAN | `192.30.33.0/24` | Wired Workstation / PC network. |
| **Site 3 (Bot-Left)** | Wireless (DHCP-2) | `192.40.33.0/24` | DHCP Pool for mobile devices (Tablets/Smartphones). |
| **Site 4 (Bot-Right)**| Voice (IP Phone) | `192.10.33.0/24` | Cisco CallManager Express (CME) for VoIP services. |
| **Site 4 (Bot-Right)**| Wireless (DHCP-1) | `192.20.33.0/24` | DHCP Pool for additional mobile devices. |
| **Backbone WAN** | Inter-Router Serial | `200.X.X.0/30` | `200.20.20.0`, `200.30.30.0`, `200.40.40.0`, `200.50.50.0` |

---

## 📝 Complete Hardware Configurations

### PART A: ROUTER CONFIGURATIONS (RIPv2 & Services)

#### 1. Router Top-Left (OB Premium)
```text
enable
configure terminal
hostname R_TopLeft

! --- Serial WAN Interfaces ---
interface Serial0/3/1
 ip address 200.20.20.1 255.255.255.252
 no shutdown

interface Serial0/3/0
 ip address 200.50.50.2 255.255.255.252
 no shutdown

! --- LAN Interface ---
interface GigabitEthernet0/1
 ip address 192.50.33.1 255.255.255.0
 no shutdown

! --- RIPv2 Dynamic Routing ---
router rip
 version 2
 network 192.50.33.0
 network 200.20.20.0
 network 200.50.50.0
 no auto-summary
```

#### 2. Router Top-Right (Server & Printer)
```text
enable
configure terminal
hostname R_TopRight

! --- Serial WAN Interfaces ---
interface Serial0/3/1
 ip address 200.20.20.2 255.255.255.252
 no shutdown

interface Serial0/3/0
 ip address 200.30.30.1 255.255.255.252
 no shutdown

! --- Inter-VLAN Sub-interfaces (Router-on-a-Stick) ---
interface GigabitEthernet0/0
 no shutdown

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.70.33.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.33.60.1 255.255.255.0

! --- RIPv2 Dynamic Routing ---
router rip
 version 2
 network 192.70.33.0
 network 192.33.60.0
 network 200.20.20.0
 network 200.30.30.0
 no auto-summary
```

#### 3. Router Bottom-Left (Data LAN & Wireless DHCP)
```text
enable
configure terminal
hostname R_BotLeft

! --- Serial WAN Interfaces ---
interface Serial0/3/0
 ip address 200.50.50.1 255.255.255.252
 no shutdown

interface Serial0/3/1
 ip address 200.40.40.2 255.255.255.252
 no shutdown

! --- Inter-VLAN Sub-interfaces ---
interface FastEthernet0/0
 no shutdown

interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.30.33.1 255.255.255.0

interface FastEthernet0/0.40
 encapsulation dot1Q 40
 ip address 192.40.33.1 255.255.255.0

! --- DHCP Server for Wireless (DHCP-2) ---
ip dhcp pool WIRELESS_DHCP_2
 network 192.40.33.0 255.255.255.0
 default-router 192.40.33.1
 dns-server 192.70.33.2

! --- RIPv2 Dynamic Routing ---
router rip
 version 2
 network 192.30.33.0
 network 192.40.33.0
 network 200.40.40.0
 network 200.50.50.0
 no auto-summary
```

#### 4. Router Bottom-Right (Voice VoIP & Wireless DHCP)
```text
enable
configure terminal
hostname R_BotRight

! --- Serial WAN Interfaces ---
interface Serial0/3/0
 ip address 200.30.30.2 255.255.255.252
 no shutdown

interface Serial0/3/1
 ip address 200.40.40.1 255.255.255.252
 no shutdown

! --- Inter-VLAN Sub-interfaces ---
interface FastEthernet0/0
 no shutdown

interface FastEthernet0/0.50
 encapsulation dot1Q 50
 ip address 192.10.33.1 255.255.255.0

interface FastEthernet0/0.60
 encapsulation dot1Q 60
 ip address 192.20.33.1 255.255.255.0

! --- DHCP Server for IP Phones & Wireless ---
ip dhcp pool VOICE_POOL
 network 192.10.33.0 255.255.255.0
 default-router 192.10.33.1
 option 150 ip 192.10.33.1

ip dhcp pool WIRELESS_DHCP_1
 network 192.20.33.0 255.255.255.0
 default-router 192.20.33.1
 dns-server 192.70.33.2

! --- Cisco CallManager Express (Telephony Service) ---
telephony-service
 max-ephones 5
 max-dn 5
 ip source-address 192.10.33.1 port 2000
 auto assign 1 to 5
 ephone-dn 1
  number 1001
 ephone-dn 2
  number 1002
 ephone-dn 3
  number 1003

! --- RIPv2 Dynamic Routing ---
router rip
 version 2
 network 192.10.33.0
 network 192.20.33.0
 network 200.30.30.0
 network 200.40.40.0
 no auto-summary
```

---

### PART B: SWITCH CONFIGURATIONS (Trunking & Access)

#### 1. Switch Top-Left (OB Premium)
```text
enable
configure terminal
hostname SW_TopLeft

! Access port for PC
interface FastEthernet0/1
 switchport mode access
 no shutdown
```

#### 2. Switch Top-Right (Server & Printer)
```text
enable
configure terminal
hostname SW_TopRight

vlan 10
 name SERVER
vlan 20
 name PRINTER

! Trunk port to Router
interface GigabitEthernet0/1
 switchport mode trunk
 no shutdown

! Access port to Printer (VLAN 20)
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 20
 no shutdown

! Access port to Server (VLAN 10)
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 no shutdown
```

#### 3. Switch Bottom-Left (Data & Wireless)
```text
enable
configure terminal
hostname SW_BotLeft

vlan 30
 name DATA
vlan 40
 name WIRELESS

! Trunk port to Router
interface GigabitEthernet0/2
 switchport mode trunk
 no shutdown

! Access port to Access Point (VLAN 40)
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 40
 no shutdown

! Access ports to PCs (VLAN 30)
interface range FastEthernet0/1 - 3
 switchport mode access
 switchport access vlan 30
 no shutdown
```

#### 4. Switch Bottom-Right (Voice & Wireless)
```text
enable
configure terminal
hostname SW_BotRight

vlan 50
 name VOICE
vlan 60
 name WIRELESS

! Trunk port to Router
interface GigabitEthernet0/1
 switchport mode trunk
 no shutdown

! Access port to Access Point (VLAN 60)
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 60
 no shutdown

! Access ports to IP Phones (Voice VLAN 50)
interface range FastEthernet0/1 - 3
 switchport mode access
 switchport voice vlan 50
 no shutdown
```

---

## 🧠 Core Concepts & Technical Explanations

Here is a breakdown of the key networking concepts utilized to make this topology function smoothly:

### 1. Dynamic Routing Protocol (RIPv2)
Routing Information Protocol version 2 (RIPv2) is used to automatically share routing tables between the 4 routers. By advertising the connected networks (`network 192...` and `network 200...`), the routers dynamically learn the best paths to reach distant subnets. Because the backbone is a ring, if one path fails, RIPv2 will automatically recalculate and route traffic in the opposite direction.

### 2. Inter-VLAN Routing (802.1Q)
To allow different VLANs (such as Server VLAN 10 and Printer VLAN 20) to communicate, a method known as "Router-on-a-Stick" is used. A single physical interface on the router is divided into multiple virtual sub-interfaces (e.g., `GigabitEthernet0/0.10`). Each sub-interface uses **802.1Q encapsulation** to tag traffic with its respective VLAN ID, acting as the default gateway for that specific VLAN.

### 3. Cisco CallManager Express (CME)
The bottom-right router acts as a centralized PBX server for the IP telephony system. The `telephony-service` command enables the router to assign directory numbers (extensions like 1001, 1002) and manage active calls. The DHCP pool includes **Option 150**, which provides the IP Phones with the IP address of the CME router so they can download their configuration files upon booting.

---

## 🛠️ Troubleshooting & Verification

To verify the setup is running correctly, perform the following tests:
* **Check Routing Tables:** Run `show ip route` in any Router's privileged EXEC mode. You should see multiple routes marked with `R` (learned via RIP).
* **Test Ring Redundancy:** Open a constant ping from a PC in Site 1 to the Server in Site 2 (`ping 192.70.33.2 -t`). Delete one of the Serial cables linking the routers. The ping may time out briefly but will automatically resume once RIP updates its routing table.
* **Test VoIP:** Open the GUI of two different IP Phones. You should see a registered extension number at the top right of the phone screen (e.g., `1001`). Dial the extension of the other phone to initiate a call.
