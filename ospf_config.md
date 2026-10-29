# Enterprise Network Architecture Simulation (Cisco Packet Tracer)

A production-grade network topology lab simulating a segmented corporate headquarters connected across an isolated WAN link to a remote branch office. This configuration models industry-best practices for structural automation, network segmentation, access control firewall policies, dynamic path selection, and wireless integration.

---

## 🗺️ Architectural Topology Overview

The infrastructure framework covers the following functional components:
* **Core Switch Layer (Switch0):** Cisco 2960 multi-VLAN distribution deployment managing `VLAN 10 (Sales)` and `VLAN 20 (Marketing)`.
* **Inter-VLAN Gateway Routing Engine (Router0):** Cisco 4331 Router hosting 802.1Q sub-interfaces providing Router-on-a-Stick (ROAS) transit.
* **Infrastructure Core Services:** Unified HTTP Intra-web platform and an institutional DNS server mapped to a clean domain mapping layout (`yusufkhaukha.com`).
* **Automated Network Parameterization:** High-availability DHCP server engines mapping independent network blocks.
* **Access-Layer Hardening Policies:** Active MAC-address validation via Sticky Port Security, along with selective state packet filtering using Extended Access Control Lists (ACLs).
* **WAN Core & Translation Engines:** Isolated point-to-point backbone running dynamic pathing via OSPF Backbone Area 0 and Network Address Translation (NAT/PAT Overload).
* **WLAN Infrastructure Expansion:** WPA2-PSK protected Access Point topology supporting local endpoint scaling.

---

## 🛠️ Network Parameter Map

| Device Layer | Physical/Logical Port | Assigned IPv4 Address | Subnet Mask Profile | Default Gateway Mapping |
| :--- | :--- | :--- | :--- | :--- |
| **PC0 (Sales)** | FastEthernet0 | DHCP Assigned (`192.168.1.X`) | `255.255.255.0` | `192.168.1.1` |
| **PC1 (Sales)** | FastEthernet0 | DHCP Assigned (`192.168.1.X`) | `255.255.255.0` | `192.168.1.1` |
| **PC2 (Marketing)** | FastEthernet0 | DHCP Assigned (`192.168.2.X`) | `255.255.255.0` | `192.168.2.1` |
| **Server0 (Services)** | FastEthernet0 | `192.168.2.50` | `255.255.255.0` | `192.168.2.1` |
| **PC4 (Branch)** | FastEthernet0 | `192.168.3.10` | `255.255.255.0` | `192.168.3.1` |
| **Router0 (Gateway)** | GigabitEthernet0/0/0.10 | `192.168.1.1` | `255.255.255.0` | *N/A (Logical Root)* |
| **Router0 (Gateway)** | GigabitEthernet0/0/0.20 | `192.168.2.1` | `255.255.255.0` | *N/A (Logical Root)* |
| **Router0 (WAN Link)** | GigabitEthernet0/0/1 | `10.0.0.1` | `255.255.255.252` | *N/A (Point-to-Point)* |
| **Router1 (ISP Link)** | GigabitEthernet0/0/0 | `10.0.0.2` | `255.255.255.252` | *N/A (Point-to-Point)* |
| **Router1 (Branch Link)**| GigabitEthernet0/0/1 | `192.168.3.1` | `255.255.255.0` | *N/A (Local Gateway)* |

---

## 💻 Cisco IOS Implementation Script Matrix

### 1. Switch Framework Layer-2 Settings (Switch0)
```text
enable
configure terminal
vlan 10
 name Sales
exit
interface range FastEthernet0/1-2
 switchport mode access
 switchport access vlan 10
exit
vlan 20
 name Marketing
exit
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
exit
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 20
exit
interface FastEthernet0/6
 switchport mode access
 switchport access vlan 10
exit
interface GigabitEthernet0/1
 switchport mode trunk
exit
```

### 2. Layer-2 Access Hardening (Switch0 - Fa0/1 Port Security)
```text
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
exit
```

### 3. Core Routing & Automation Parameters (Router0)
```text
enable
configure terminal
interface GigabitEthernet0/0/0
 no ip address
 no shutdown
exit
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.0
exit
interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.2.1 255.255.255.0
exit
ip dhcp excluded-address 192.168.1.1
ip dhcp excluded-address 192.168.2.1
ip dhcp pool VLAN10_POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 192.168.2.50
exit
ip dhcp pool VLAN20_POOL
 network 192.168.2.0 255.255.255.0
 default-router 192.168.2.1
 dns-server 192.168.2.50
exit
```

### 4. Extended Access Control Perimeter Firewalls (Router0)
```text
ip access-list extended SERVER_FIREWALL
 deny ip host 192.168.1.2 host 192.168.2.50
 permit ip any any
exit
interface GigabitEthernet0/0/0.20
 ip access-group SERVER_FIREWALL out
exit
```

### 5. WAN Link Configuration, OSPF Engine & NAT/PAT Framework
```text
# Configure WAN Link Interface Address
interface GigabitEthernet0/0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit

# Configure Dynamic OSPF Framework (Excluding Internal Networks for NAT)
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
exit

# Configure PAT Overload Operational Scope
access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 permit 192.168.2.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/0/1 overload

# Apply Structural NAT Boundary Definitions
interface GigabitEthernet0/0/0.10
 ip nat inside
exit
interface GigabitEthernet0/0/0.20
 ip nat inside
exit
interface GigabitEthernet0/0/1
 ip nat outside
exit

# Build Static Default Route out Toward the ISP
ip route 0.0.0.0 0.0.0.0 10.0.0.2
exit
```

### 6. ISP Remote Site Operational Parameters (Router1)
```text
enable
configure terminal
interface GigabitEthernet0/0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/0/1
 ip address 192.168.3.1 255.255.255.0
 no shutdown
exit
router ospf 1
 network 192.168.3.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
exit
```

### 7. Permanent Committal to Non-Volatile Memory (All System Elements)
```text
exit
copy running-config startup-config
```

---

## 🔍 Validation Command Logs

To verify traffic flows, topology convergence, and security constraints, run the following verification commands in the CLI:
* `show ip route`: Validates structural visibility across converged OSPF dynamic paths and default signposts.
* `show ip nat translations`: Verifies proper PAT translation mapping logs when internal hosts are actively pinging cross-WAN devices.
* `show port-security interface FastEthernet0/1`: Reviews operational states, sticky MAC addresses, and total counts of physical link violations.
* `show access-lists`: Monitors firewall tracking hits across assigned permit and deny extended parameter ranges.

---
*Developed inside Cisco Packet Tracer for Network Infrastructure Lab Documentation.*
