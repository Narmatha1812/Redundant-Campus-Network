# 🔗 Redundant Campus Network

A resilient Cisco campus network designed to provide **high availability, fault tolerance, and network redundancy** using **EtherChannel, RSTP, HSRP, and OSPF**.

This project was designed and simulated using **Cisco Packet Tracer** to demonstrate how a campus network can maintain connectivity when links or gateway routers experience failures.

---

## 📌 Project Overview

In a traditional network, the failure of a single link or gateway device can interrupt communication between users.

This project addresses that problem by implementing multiple redundancy mechanisms:

- **EtherChannel** – Provides link redundancy and increased bandwidth by combining multiple physical links.
- **RSTP** – Provides rapid Layer 2 convergence and prevents switching loops.
- **HSRP** – Provides a redundant default gateway for end devices.
- **OSPF** – Provides dynamic routing and routing information exchange between routers.

The network was tested by intentionally disconnecting links and shutting down the primary gateway to verify that connectivity can be maintained or restored through redundant paths.

---

## 🎯 Objectives

The main objectives of this project are:

- Design a fault-tolerant campus network.
- Implement Layer 2 and Layer 3 redundancy.
- Configure EtherChannel using LACP.
- Implement Rapid PVST+ for fast spanning-tree convergence.
- Configure HSRP for gateway redundancy.
- Configure OSPF for dynamic routing.
- Test network behavior during link and gateway failures.
- Verify network operation using Cisco IOS commands.

---

## 🏗️ Network Topology

The network consists of:

- 2 Cisco 2911 Routers
- 4 Cisco 2960 Switches
- 4 PCs
- Redundant switch-to-switch links
- Redundant router gateways
- EtherChannel links
- RSTP-based alternate paths

### Topology

```text
                         R1                 R2
                         │                  │
                         │                  │
                       ┌─┴─┐              ┌─┴─┐
                       │SW1│══════════════│SW2│
                       └─┬─┘  EtherChannel └─┬─┘
                         │                   │
                         │                   │
                       ┌─┴─┐              ┌─┴─┐
                       │SW3│══════════════│SW4│
                       └─┬─┘  EtherChannel └─┬─┘
                        / \                 / \
                      PC1 PC2             PC3 PC4
```

The topology contains redundant paths between the switches so that traffic can be redirected when an active link fails.

---

## 🌐 IP Addressing

### Router Interfaces

| Device | Interface | IP Address | Purpose |
|--------|-----------|------------|---------|
| R1 | G0/0 | 192.168.10.2/24 | Campus LAN |
| R2 | G0/0 | 192.168.10.3/24 | Campus LAN |
| R1 | G0/1 | 10.0.0.1/30 | Router-to-router link |
| R2 | G0/1 | 10.0.0.2/30 | Router-to-router link |
| HSRP | Virtual IP | 192.168.10.1 | Default Gateway |

### End Devices

| Device | IP Address | Default Gateway |
|--------|------------|-----------------|
| PC1 | 192.168.10.11 | 192.168.10.1 |
| PC2 | 192.168.10.12 | 192.168.10.1 |
| PC3 | 192.168.10.13 | 192.168.10.1 |
| PC4 | 192.168.10.14 | 192.168.10.1 |

---

## 🧩 Technologies Used

| Technology | Purpose |
|------------|---------|
| Cisco Packet Tracer | Network simulation |
| VLAN | Logical network segmentation |
| 802.1Q Trunking | VLAN transport |
| EtherChannel | Link redundancy and aggregation |
| LACP | EtherChannel negotiation |
| RSTP / Rapid PVST+ | Loop prevention and fast convergence |
| HSRP | Default gateway redundancy |
| OSPF | Dynamic routing |

---

# 🔧 Network Configuration

## 1️⃣ VLAN and Trunking

A dedicated campus VLAN was configured:

```text
VLAN 10 → CAMPUS
```

Trunk links were configured between the switches to allow VLAN traffic to traverse the redundant network paths.

---

## 2️⃣ EtherChannel

EtherChannel was implemented using **LACP**.

### SW1 ↔ SW2

```text
Fa0/2
Fa0/3
   ↓
Port-Channel 1
```

### SW3 ↔ SW4

```text
Fa0/2
Fa0/3
   ↓
Port-Channel 2
```

LACP allows multiple physical interfaces to operate as a single logical link.

### Verification

```cisco
show etherchannel summary
```

Expected status:

```text
Po1(SU)
Po2(SU)
```

Where:

- `S` = Layer 2 EtherChannel
- `U` = Port-Channel is in use
- `P` = Interface is successfully bundled

---

## 3️⃣ RSTP

Rapid PVST+ was configured on all switches:

```cisco
spanning-tree mode rapid-pvst


SW1 was configured as the primary root bridge:

```cisco
spanning-tree vlan 10 root primary
```

SW2 was configured as the secondary root bridge:

```cisco
spanning-tree vlan 10 root secondary
```

RSTP prevents Layer 2 switching loops while allowing an alternate path to become active when the primary path fails.

### Verification

```cisco
show spanning-tree vlan 10
```

The network intentionally contains a blocked alternate path.

For example:

```text
Po2   Altn   BLK
```

This blocked path can become active when the active path fails.

---

## 4️⃣ HSRP

HSRP was configured between R1 and R2 to provide a redundant default gateway.

### HSRP Configuration

| Device | IP Address | Priority | Role |
|--------|------------|----------|------|
| R1 | 192.168.10.2 | 110 | Active |
| R2 | 192.168.10.3 | 100 | Standby |

**Virtual Gateway:** `192.168.10.1`

All PCs use:

```text
Default Gateway: 192.168.10.1
```

The PCs do not need to know which physical router is currently active.

### Verification

```cisco
show standby
```

### Failover Test

When R1's LAN interface is shut down:

```cisco
configure terminal
interface gigabitEthernet 0/0
shutdown
end
```

R2 becomes the HSRP Active router.

The virtual gateway remains:

```text
192.168.10.1
```

Therefore, end devices can continue using the same default gateway.

---

## 5️⃣ OSPF

OSPF was configured between R1 and R2 using **Area 0**.

### R1

```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
```

### R2

```cisco
router ospf 1
 router-id 2.2.2.2
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
```

### Verification

```cisco
show ip ospf neighbor
```

The OSPF neighbor relationship should reach:

```text
FULL
```

OSPF dynamically establishes an adjacency and exchanges routing information between the routers.

> **Note:** In this topology, OSPF is demonstrated as a dynamic routing protocol and adjacency mechanism. The topology does not contain a second Layer 3 path, so this test should not be described as OSPF providing alternate-path failover.

---

# 🧪 Failure Demonstrations

The network was tested under different failure conditions.

---

## 🔴 Test 1 – EtherChannel Link Failure

### Scenario

One physical link between SW1 and SW2 was disconnected.

**Before:**

```text
SW1 ═════════ SW2
    Fa0/2
    Fa0/3
```

One member link was intentionally disconnected.

### Result

The remaining EtherChannel member continued carrying traffic.

```text
SW1 ───────── SW2
       Fa0/3
```

PC connectivity was maintained.

### Demonstrates

**EtherChannel provides physical link redundancy by combining multiple physical links into one logical connection.**

---

## 🔴 Test 2 – RSTP Path Failure

The active link between SW2 and SW4 was disconnected.

```text
SW2 ─────X───── SW4
```

RSTP detected the failure and activated the previously blocked alternate path through the lower EtherChannel.

```text
SW2
 │
SW1
 │
SW3 ═════ SW4
```

A short interruption may occur while RSTP reconverges.

### Demonstrates

**RSTP prevents Layer 2 loops while providing rapid recovery through an alternate path.**

---

## 🔴 Test 3 – HSRP Gateway Failure

R1 was configured as the HSRP Active router.

When R1's LAN interface was shut down:

```text
R1 → DOWN
```

R2 automatically became:

```text
HSRP Active
```

The virtual gateway remained:

```text
192.168.10.1
```

### Demonstrates

**HSRP provides first-hop gateway redundancy and prevents a single router failure from disconnecting end devices from their gateway.**

---

## 🔴 Test 4 – OSPF Adjacency Test

The router-to-router OSPF link was temporarily shut down.

The OSPF adjacency was lost.

After restoring the interface:

```cisco
configure terminal
interface gigabitEthernet 0/1
no shutdown
end
```

the routers re-established the OSPF adjacency.

### Demonstrates

**OSPF dynamically establishes and maintains routing adjacencies between routers.**

---

# 🔍 Verification Commands

### EtherChannel

```cisco
show etherchannel summary
```

### RSTP

```cisco
show spanning-tree vlan 10
```

### HSRP

```cisco
show standby
```

### OSPF Neighbors

```cisco
show ip ospf neighbor
```

### OSPF Routes

```cisco
show ip route ospf
```

### OSPF Configuration

```cisco
show ip protocols
```

### Trunk Status

```cisco
show interfaces trunk
```

### Interface Status

```cisco
show ip interface brief
```

---
# 🚀 Key Learning Outcomes

Through this project, I gained practical experience in:

- Cisco IOS configuration
- VLAN and trunk configuration
- Link aggregation using LACP
- Layer 2 redundancy using RSTP
- Gateway redundancy using HSRP
- Dynamic routing using OSPF
- Network troubleshooting
- Failure testing and fault analysis
- Network verification using Cisco IOS commands
- Understanding network convergence and redundancy

---

# 🎓 Project Highlights

This project demonstrates how multiple networking technologies can work together to improve **network availability and fault tolerance**.

```text
EtherChannel
     ↓
Link Redundancy

RSTP
     ↓
Layer 2 Path Redundancy

HSRP
     ↓
Gateway Redundancy

OSPF
     ↓
Dynamic Routing
```

Together, these technologies create a more resilient campus network architecture.

---

# 💡 Why Redundancy Matters

Network availability is critical in modern organizations such as:

- Universities
- Enterprises
- Data centers
- Hospitals
- Government organizations
- Telecommunication networks

A resilient network should be able to handle individual link or device failures without causing a complete network outage.

This project demonstrates the fundamental concepts used to achieve that resilience.

---

## ⭐ Project Status

**Completed ✅**

Network simulation, redundancy configuration, failure testing, and verification were performed using Cisco Packet Tracer.

---

⭐ If you found this project useful, feel free to explore the configurations and Packet Tracer topology.
