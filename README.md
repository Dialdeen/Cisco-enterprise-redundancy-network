Here is the updated `README.md` file code with the **Native VLAN Security Customization** (changing default VLAN 1 to Native VLAN 100 on trunks/EtherChannels) integrated into the architecture, security features, and configuration verification sections.

Copy and paste this markdown directly into your GitHub `README.md`:

```markdown
# Cisco Enterprise Redundancy & Security Network

A highly available, secure, and redundant enterprise campus network architecture emulated in **GNS3** utilizing the **GNS3 VM** on **VMware Workstation**, and rigorously validated using **Wireshark** for deep packet analysis. 

This project demonstrates core enterprise network engineering principles, including Layer 3 switching, Hot Standby Router Protocol (HSRP), Open Shortest Path First (OSPF), Spanning Tree Protocol (STP) root bridge control, Link Aggregation (LACP), centralized DHCP services, extended Access Control Lists (ACLs), Native VLAN security hardening, and full device AAA/SSH management.

---

## 📐 Network Topology


```
<img width="1024" height="888" alt="image" src="https://github.com/user-attachments/assets/7aecac16-2a50-4a6c-81b3-4a2155ac9f8a" />


<img width="945" height="925" alt="image" src="https://github.com/user-attachments/assets/c1db0437-f64b-4cb5-a1ea-2f55333b9022" />


```

```

---

## 🛠️ Key Technologies & Implementation Details

### 1. Layer 3 Switching & First-Hop Redundancy
* **Layer 3 Inter-VLAN Routing:** Enabled `ip routing` on core Multilayer Switches to handle local intra-VLAN and inter-VLAN routing natively without overloading edge routers.
* **HSRP (Hot Standby Router Protocol v2):** 
  * **Root-Switch (Active Gateway):** Configured as the primary Virtual Gateway (`192.168.1.1` and `192.168.2.1`) with a priority of `110` and preempt enabled.
  * **Sec-Switch (Standby Gateway):** Configured as the backup Virtual Gateway with a default priority of `100` and preempt enabled for instant failover upon core switch or link offline events.

### 2. Spanning Tree & Link Aggregation
* **Spanning Tree Protocol (PVST+):** Explicitly declared `Root-Switch` as the **Primary Root Bridge** (`spanning-tree vlan 10,20 root primary`) and `Sec-Switch` as the **Secondary Root Bridge** to ensure deterministic L2 loop-free paths.
* **PortFast:** Enabled on access edge ports connected to end-host workstations to bypass listening/learning states and immediately transition to forwarding.
* **EtherChannel (LACP - Channel-Group 1):** Configured 802.3ad dynamic LACP link aggregation between core switches across multiple physical links to increase backbone bandwidth and ensure trunk link failure tolerance.

### 3. Native VLAN Security & Trunk Hardening
* **Native VLAN Migration (VLAN 100):** Default VLAN 1 was explicitly changed to **VLAN 100** across all EtherChannel trunk links (`switchport trunk native vlan 100`) to mitigate **VLAN Hopping attacks** and **double-tagging exploits**.
* **Trunk VLAN Pruning:** Trunk ports were pruned to strictly allow active production VLANs and the custom native VLAN (`switchport trunk allowed vlan 10,20,100`), ensuring unused broadcast domains are isolated.

### 4. Dynamic & Static Routing Architecture
* **OSPF (Open Shortest Path First v2):** Configured Area 0 across all Layer 3 routed interfaces (`10.1.1.0/30`, `10.1.2.0/30`, `10.2.1.0/30`, `10.2.2.0/30`) and client subnets for real-time dynamic route distribution and sub-second convergence.
* **Static WAN Routing:** Standard default routes (`0.0.0.0 0.0.0.0`) configured on perimeter edge routers pointing toward the ISP/Internet subnets (`10.0.1.0/30` and `10.0.2.0/30`).

### 5. Core Network Services
* **Centralized DHCP Server:** Configured `Root-Switch` as the primary DHCP Server providing dynamic lease distribution (IP, Gateway, DNS) for **VLAN 10 (Staff)** and **VLAN 20 (Admins)**.
* **DHCP Relay (IP Helper):** Configured `ip helper-address 192.168.1.2` on `Sec-Switch` SVIs to forward DHCP Discover broadcasts across Layer 3 boundaries back to the central DHCP server.

### 6. Network Security & Device Hardening
* **Traffic Control (Extended ACLs):** Configured `BLOCK_STAFF_TO_ADMIN` extended access lists directly on SVIs to enforce zero-trust policies, restricting Staff members in VLAN 10 from initiating connections into the Admin VLAN 20 network.
* **AAA & Local Authentication:** Configured local privilege 15 admin accounts (`admin`) paired with strong `enable secret` passwords and global `service password-encryption`.
* **Encrypted Management (SSH v2):** Disabled unencrypted Telnet management on VTY lines (`0 4`), initialized 2048-bit RSA encryption keys (`crypto key generate rsa`), and enforced SSH-only administrative access.

---

## 🦈 Wireshark Packet Capture & Verification

Using native **Wireshark** integration within GNS3, live packet analysis was performed across virtual interfaces to inspect control plane and data plane behavior:

* **Native VLAN Tagging Inspection:** Analyzed Ethernet frame headers traversing the EtherChannel trunk to verify standard untagged frames were properly mapped to Native VLAN 100 and that VLAN 1 frames were blocked/ignored.
* **HSRP Keepalives:** Captured version 2 multicast packets (`224.0.0.102` on UDP port 1985) verifying active state timers and virtual MAC (`00:00:0c:9f:f0:xx`) assignments.
* **LACP PDU Inspection:** Monitored IEEE 802.3ad negotiation control frames traversing the EtherChannel bundle to ensure duplex matching and active state agreement.
* **OSPF Adjacencies:** Captured OSPF Hello packets (`224.0.0.5`) and Link State Advertisements (LSAs) during topology convergence.
* **DHCP DORA Exchange:** Verified 4-way **Discover, Offer, Request, Acknowledgment** handshakes bridged across SVIs and IP helper interfaces.
* **ACL Verification:** Tested packet rejection via ICMP captures confirming *Destination Unreachable (Communication Administratively Prohibited)* responses when VLAN 10 attempted to contact VLAN 20.

---

## 🌐 Subnetting & Addressing Scheme

| Zone / Interface | Network Address | Subnet Mask | Gateway / Role |
| :--- | :--- | :--- | :--- |
| **VLAN 10 (Staff)** | `192.168.1.0` | `/24 (255.255.255.0)` | `192.168.1.1` (Virtual Gateway) |
| **VLAN 20 (Admins)** | `192.168.2.0` | `/24 (255.255.255.0)` | `192.168.2.1` (Virtual Gateway) |
| **VLAN 100 (Native)**| `N/A (Layer 2 Only)`| `N/A` | Security Trunk Management |
| **Router1 to Internet** | `10.0.1.0` | `/30 (255.255.255.252)` |  WAN Edge 1 |
| **Router2 to Internet** | `10.0.2.0` | `/30 (255.255.255.252)` |  WAN Edge 2 |
| **Router1 to Root-Switch** | `10.1.1.0` | `/30 (255.255.255.252)` | Primary Core Transit Link |
| **Router1 to Sec-Switch** | `10.2.1.0` | `/30 (255.255.255.252)` | Backup Core Transit Link |
| **Router2 to Root-Switch** | `10.1.2.0` | `/30 (255.255.255.252)` | Backup Core Transit Link |
| **Router2 to Sec-Switch** | `10.2.2.0` | `/30 (255.255.255.252)` | Primary Core Transit Link |

---

## 🧪 Verification & Failover Test Scenarios

- [x] **Trunk Native VLAN Match:** Confirmed `show interfaces trunk` outputs on both switches matched `Native vlan: 100` to prevent CDP Native VLAN mismatch warnings.
- [x] **HSRP Active Gateway Failover:** Administrative shutdown of `Root-Switch` SVIs forced `Sec-Switch` to transition from Standby to Active in <1 second with zero connection loss on client devices.
- [x] **EtherChannel Trunk Link Loss:** Severed one physical wire within Channel-Group 1 to verify frame forwarding continued seamlessly across the remaining LACP link.
- [x] **Security ACL Isolation:** Ping tests confirmed `User1` (VLAN 10) was denied access to `Admin1` (VLAN 20), while `Admin1` retained full outbound access.
- [x] **SSH Management Hardening:** Verified SSH terminal access via local authentication while confirming plaintext Telnet requests were rejected.

---

## 🧰 Software & Tools Stack
* **GNS3** (Graphical Network Simulator-3)
* **GNS3 VM**
* **VMware Workstation Pro**
* **Wireshark** (Network Protocol Analyzer)
* **Cisco IOSv / IOSvL2 Images**

```
