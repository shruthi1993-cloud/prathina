# Inter-VLAN Routing and L3 Switches - Complete Guide

## Table of Contents
1. [The Inter-VLAN Routing Problem](#the-inter-vlan-routing-problem)
2. [Two Methods for Inter-VLAN Routing](#two-methods-for-inter-vlan-routing)
3. [What is an L3 Switch?](#what-is-an-l3-switch)
4. [L3 Switch Configuration](#l3-switch-configuration)
5. [How Inter-VLAN Routing Works](#how-inter-vlan-routing-works)
6. [L2 vs L3 Switch Comparison](#l2-switch-vs-l3-switch)
7. [When to Use Each Method](#when-to-use-each-method)

---

## The Inter-VLAN Routing Problem

**Problem**: VLANs create isolated broadcast domains. Devices in different VLANs (different subnets) **cannot communicate** with each other by default, even if they're on the same physical switch.

### Example Scenario:
```
VLAN 10 (HR)              VLAN 20 (Sales)
192.168.10.0/24          192.168.20.0/24
[PC-HR]                  [PC-Sales]
192.168.10.5             192.168.20.5
      │                       │
      └───────────────────────┘
           [ L2 Switch ]

❌ PC-HR cannot ping PC-Sales
❌ They're in different subnets
✓ Need Layer 3 routing to communicate
```

**Why?**
- VLANs operate at **Layer 2** (Data Link)
- Different VLANs = Different subnets
- Communication between subnets requires **Layer 3** (Network/IP) routing
- A Layer 2 switch only switches based on MAC addresses, not IP addresses

**Solution**: Inter-VLAN routing enables communication between VLANs using routing functionality.

---

## Two Methods for Inter-VLAN Routing

### Method 1: Router-on-a-Stick (Legacy Approach)

Uses a **traditional router** with a single physical interface divided into multiple logical sub-interfaces (one per VLAN).

#### Topology:
```
VLAN 10          VLAN 20          VLAN 30
[PC HR]          [PC Sales]       [PC IT]
192.168.10.5     192.168.20.5     192.168.30.5
  │                 │                │
  └─────────────────┴────────────────┘
              [ L2 Switch ]
                    │
                 Trunk Link (Gi0/1)
                    │ Carries all VLANs (Tagged frames)
              ┌─────┴─────┐
              │  Router   │  Sub-interfaces:
              │  Gi0/0    │  - Gi0/0.10 (VLAN 10 - 192.168.10.1)
              │           │  - Gi0/0.20 (VLAN 20 - 192.168.20.1)
              │           │  - Gi0/0.30 (VLAN 30 - 192.168.30.1)
              └───────────┘
```

#### How It Works:
1. PC in VLAN 10 wants to reach PC in VLAN 20
2. Traffic goes from PC → Switch → Trunk link → Router sub-interface (Gi0/0.10)
3. Router performs routing between sub-interfaces (Gi0/0.10 → Gi0/0.20)
4. Traffic returns via Gi0/0.20 → Trunk → Switch → VLAN 20 → Destination PC

#### Router Configuration Example:
```cisco
! Create sub-interfaces for each VLAN
Router(config)# interface gigabitEthernet 0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# description Gateway for VLAN 10

Router(config)# interface gigabitEthernet 0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# description Gateway for VLAN 20

Router(config)# interface gigabitEthernet 0/0.30
Router(config-subif)# encapsulation dot1Q 30
Router(config-subif)# ip address 192.168.30.1 255.255.255.0
Router(config-subif)# description Gateway for VLAN 30

! Enable the physical interface
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# no shutdown
```

#### Drawbacks:
- **Bottleneck!** All inter-VLAN traffic must traverse the same physical link **twice** (up to router, down from router)
- **Slower performance** compared to L3 switching
- **Single point of failure** on the trunk link
- **Not scalable** for high inter-VLAN traffic environments

---

### Method 2: L3 Switch / Multilayer Switch (Modern/Preferred)

An **L3 (Layer 3) switch** combines switching (Layer 2) and routing (Layer 3) functionality in a single device.

#### Topology:
```
VLAN 10          VLAN 20          VLAN 30
[PC HR]          [PC Sales]       [PC IT]
192.168.10.5     192.168.20.5     192.168.30.5
  │                 │                │
  └─────────────────┴────────────────┘
     [ L3 Multilayer Switch ]
         
    ✓ Switches at Layer 2 (within VLANs)
    ✓ Routes at Layer 3 (between VLANs)
    ✓ No external router needed!
    ✓ Routing happens internally at wire speed
```

#### Benefits:
- **Much faster** - routing performed in hardware (ASIC chips)
- **No bottleneck** - routing happens internally within the switch
- **Reduced cost** - no separate router needed for inter-VLAN routing
- **Lower latency** - wire-speed routing between VLANs
- **Scalable** - handles high inter-VLAN traffic efficiently
- **Additional features** - ACLs, QoS, routing protocols

---

## What is an L3 Switch?

An **L3 switch** (also called a **multilayer switch**) is a network device that can perform both:

### 1. **Layer 2 Switching** (Data Link Layer):
- MAC address learning and forwarding
- VLAN segmentation
- Spanning Tree Protocol (STP)
- Standard Ethernet switching within VLANs

### 2. **Layer 3 Routing** (Network Layer):
- IP routing between VLANs
- Routing protocols (OSPF, EIGRP, RIP, BGP)
- Static and dynamic routing
- Access Control Lists (ACLs)
- IP routing table management

### Key Components:

#### SVI (Switched Virtual Interface)
- A **virtual Layer 3 interface** for each VLAN
- Acts as the **default gateway** for devices in that VLAN
- Enables IP routing between VLANs
- Example: `interface vlan 10`

#### Routed Port
- A physical switch port configured as a Layer 3 interface
- Used for point-to-point Layer 3 connections
- No VLAN association
- Example: Connecting to another router or L3 switch

---

## L3 Switch Configuration

### Complete Configuration Example:

```cisco
! ========================================
! Step 1: Enable IP Routing (CRITICAL!)
! ========================================
Switch(config)# ip routing
! ^ This command turns the switch into a router for VLANs
! Without this command, SVIs won't route traffic!

! ========================================
! Step 2: Create VLANs
! ========================================
Switch(config)# vlan 10
Switch(config-vlan)# name HR
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name Sales
Switch(config-vlan)# exit

Switch(config)# vlan 30
Switch(config-vlan)# name IT
Switch(config-vlan)# exit

Switch(config)# vlan 99
Switch(config-vlan)# name Management
Switch(config-vlan)# exit

! ========================================
! Step 3: Assign Ports to VLANs (Layer 2)
! ========================================

! HR Department - VLAN 10
Switch(config)# interface range fastEthernet 0/1-5
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# description HR Department PCs
Switch(config-if-range)# exit

! Sales Department - VLAN 20
Switch(config)# interface range fastEthernet 0/6-10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Switch(config-if-range)# description Sales Department PCs
Switch(config-if-range)# exit

! IT Department - VLAN 30
Switch(config)# interface range fastEthernet 0/11-15
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 30
Switch(config-if-range)# description IT Department PCs
Switch(config-if-range)# exit

! ========================================
! Step 4: Create SVIs for Inter-VLAN Routing
! ========================================

! SVI for VLAN 10 (Default Gateway for HR)
Switch(config)# interface vlan 10
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# description Gateway for HR Department
Switch(config-if)# exit

! SVI for VLAN 20 (Default Gateway for Sales)
Switch(config)# interface vlan 20
Switch(config-if)# ip address 192.168.20.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# description Gateway for Sales Department
Switch(config-if)# exit

! SVI for VLAN 30 (Default Gateway for IT)
Switch(config)# interface vlan 30
Switch(config-if)# ip address 192.168.30.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# description Gateway for IT Department
Switch(config-if)# exit

! SVI for Management VLAN
Switch(config)# interface vlan 99
Switch(config-if)# ip address 192.168.99.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# description Management Interface
Switch(config-if)# exit

! ========================================
! Step 5: (Optional) Configure Trunk Ports
! ========================================
Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30,99
Switch(config-if)# description Trunk to Access Switch
Switch(config-if)# exit

! ========================================
! Step 6: Save Configuration
! ========================================
Switch# copy running-config startup-config
```

### End Device Configuration:

```
PC-HR (VLAN 10):
IP Address: 192.168.10.5
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1  ← SVI VLAN 10

PC-Sales (VLAN 20):
IP Address: 192.168.20.5
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1  ← SVI VLAN 20

PC-IT (VLAN 30):
IP Address: 192.168.30.5
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.30.1  ← SVI VLAN 30
```

---

## How Inter-VLAN Routing Works

### Detailed Step-by-Step Example

**Scenario**: PC-HR (VLAN 10) pings PC-Sales (VLAN 20)

```
Source:
PC-HR: 192.168.10.5
Default Gateway: 192.168.10.1 (SVI VLAN 10)

Destination:
PC-Sales: 192.168.20.5
Default Gateway: 192.168.20.1 (SVI VLAN 20)
```

### Traffic Flow:

#### Step 1: PC-HR Decision
```
PC-HR checks: Is 192.168.20.5 in my subnet (192.168.10.0/24)?
Answer: No (different subnet)
Action: Forward packet to default gateway (192.168.10.1)
```

#### Step 2: ARP for Gateway (First Time Only)
```
PC-HR needs MAC address of 192.168.10.1
Sends: ARP request "Who has 192.168.10.1?"
Switch SVI VLAN 10 responds with its MAC address
PC-HR caches: 192.168.10.1 = MAC_SVI_10
```

#### Step 3: Layer 2 Switching to SVI
```
Source MAC: PC-HR MAC
Dest MAC: MAC_SVI_10
VLAN: 10
Frame forwarded to L3 switch in VLAN 10
```

#### Step 4: Layer 3 Routing Decision
```
Switch receives frame at SVI VLAN 10
Strips Layer 2 header, examines IP packet
Destination IP: 192.168.20.5
Routing table lookup:
  192.168.20.0/24 is directly connected via Vlan20
Next hop: SVI VLAN 20 (192.168.20.1)
```

#### Step 5: ARP for Destination (First Time Only)
```
Switch needs MAC address of 192.168.20.5 in VLAN 20
Switch sends ARP via SVI VLAN 20: "Who has 192.168.20.5?"
PC-Sales responds with its MAC address
Switch caches: 192.168.20.5 = MAC_PC_Sales
```

#### Step 6: Layer 2 Switching to Destination
```
Source MAC: MAC_SVI_20 (rewritten)
Dest MAC: MAC_PC_Sales
VLAN: 20
Frame forwarded to PC-Sales in VLAN 20
```

#### Step 7: Reply Path
```
Same process in reverse:
PC-Sales → VLAN 20 → SVI VLAN 20 → Routing → SVI VLAN 10 → VLAN 10 → PC-HR
```

### Visual Representation:
```
[PC-HR]                                    [PC-Sales]
192.168.10.5                              192.168.20.5
VLAN 10                                   VLAN 20
    │                                          ▲
    │ 1. Packet to gateway                    │
    │    Dst: 192.168.20.5                    │ 6. Packet arrives
    │    Via: 192.168.10.1                    │
    ▼                                          │
┌───────────────────────────────────────────────────┐
│         L3 Multilayer Switch                      │
│                                                   │
│  2. Received at SVI VLAN 10 (192.168.10.1)       │
│         │                                         │
│         ▼                                         │
│  3. Layer 3 Routing Table Lookup                 │
│     Dest: 192.168.20.0/24 via VLAN 20            │
│         │                                         │
│         ▼                                         │
│  4. Forward via SVI VLAN 20 (192.168.20.1)       │
│         │                                         │
│         ▼                                         │
│  5. Layer 2 Switch to Port in VLAN 20            │
│                                                   │
└───────────────────────────────────────────────────┘
```

**Key Point**: All this happens **inside the switch** at wire speed using dedicated hardware ASICs, not software processing!

---

## Verification Commands

### 1. Check if IP Routing is Enabled
```cisco
Switch# show running-config | include ip routing
ip routing

! Or check with:
Switch# show ip protocols
Routing Protocol is "connected"
```

### 2. View Routing Table
```cisco
Switch# show ip route

Codes: C - connected, S - static, O - OSPF, ...

Gateway of last resort is not set

C    192.168.10.0/24 is directly connected, Vlan10
C    192.168.20.0/24 is directly connected, Vlan20
C    192.168.30.0/24 is directly connected, Vlan30
C    192.168.99.0/24 is directly connected, Vlan99
```

### 3. Show SVI Interfaces
```cisco
Switch# show ip interface brief | include Vlan

Vlan10         192.168.10.1    YES manual up                    up
Vlan20         192.168.20.1    YES manual up                    up
Vlan30         192.168.30.1    YES manual up                    up
Vlan99         192.168.99.1    YES manual up                    up
```

### 4. Detailed SVI Information
```cisco
Switch# show interface vlan 10

Vlan10 is up, line protocol is up
  Hardware is Ethernet SVI, address is 0011.2233.4455
  Internet address is 192.168.10.1/24
  MTU 1500 bytes, BW 1000000 Kbit/sec
  ...
```

### 5. Check Inter-VLAN Routing (from PC)
```bash
# From PC-HR (VLAN 10)
C:\> ping 192.168.20.5

Pinging 192.168.20.5 with 32 bytes of data:
Reply from 192.168.20.5: bytes=32 time=1ms TTL=127
...

# Traceroute shows the hop through gateway
C:\> tracert 192.168.20.5

Tracing route to 192.168.20.5 ...
  1    <1 ms    192.168.10.1   (SVI VLAN 10)
  2    <1 ms    192.168.20.5   (Destination)
```

### 6. View ARP Cache on Switch
```cisco
Switch# show ip arp

Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.10.1            -   0011.2233.4455  ARPA   Vlan10
Internet  192.168.10.5           10   0050.7966.6801  ARPA   Vlan10
Internet  192.168.20.1            -   0011.2233.4456  ARPA   Vlan20
Internet  192.168.20.5            5   0050.7966.6802  ARPA   Vlan20
```

### 7. Verify VLAN Configuration
```cisco
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/16-24
10   HR                               active    Fa0/1-5
20   Sales                            active    Fa0/6-10
30   IT                               active    Fa0/11-15
99   Management                       active
```

---

## L2 Switch vs L3 Switch

| Feature | L2 Switch | L3 Switch (Multilayer) |
|---------|-----------|------------------------|
| **OSI Layer** | Layer 2 only (Data Link) | Layer 2 + Layer 3 (Network) |
| **Primary Function** | Switching based on MAC addresses | Switching + Routing based on IP addresses |
| **Inter-VLAN Routing** | ❌ Cannot route between VLANs | ✅ Can route between VLANs |
| **Routing Table** | ❌ No routing table | ✅ Has IP routing table |
| **SVI Capability** | Management only (single SVI) | Multiple SVIs with routing |
| **IP Routing** | No | Yes (requires `ip routing` command) |
| **Routing Protocols** | ❌ No support | ✅ OSPF, EIGRP, RIP, BGP, etc. |
| **Default Gateway** | Must point to external router | Switch itself acts as gateway |
| **ACLs** | MAC-based ACLs only | IP-based ACLs (standard & extended) |
| **Performance** | Fast switching (wire speed) | Fast switching + hardware routing (wire speed) |
| **Cost** | Lower ($) | Higher ($$-$$$) |
| **Power Consumption** | Lower | Higher |
| **Use Case** | Access layer, simple networks | Distribution/Core layer, enterprise |
| **Examples** | Cisco 2960, 2960-X | Cisco 3560, 3750, 3850, 9300 |
| **Configuration** | Simpler | More complex |
| **Command** | No `ip routing` | Requires `ip routing` to enable routing |

### Visual Comparison:

```
L2 Switch (Access Layer):
┌─────────────────────────┐
│    Layer 2 Switch       │
│                         │
│  • MAC Address Table    │
│  • VLAN Segmentation    │
│  • STP                  │
│  • Port Security        │
│                         │
│  ❌ No IP Routing       │
│  ❌ No Routing Table    │
└─────────────────────────┘

L3 Switch (Distribution/Core):
┌─────────────────────────┐
│  Multilayer Switch      │
│                         │
│  Layer 2:               │
│  • MAC Address Table    │
│  • VLAN Segmentation    │
│  • STP                  │
│                         │
│  Layer 3:               │
│  ✅ IP Routing Table     │
│  ✅ Routing Protocols    │
│  ✅ SVIs                 │
│  ✅ ACLs                 │
└─────────────────────────┘
```

---

## When to Use Each Method

### Router-on-a-Stick

**Best For:**
- ✅ Small networks (fewer than 5 VLANs)
- ✅ Limited budget / cost constraints
- ✅ Low inter-VLAN traffic requirements
- ✅ Already have an existing router
- ✅ Learning environments / labs

**Avoid When:**
- ❌ High inter-VLAN traffic volume
- ❌ Need for wire-speed routing
- ❌ Scalability is important
- ❌ Large enterprise networks

**Example Use Case:**
```
Small office with 3 VLANs:
- VLAN 10: Employees (15 users)
- VLAN 20: Guest WiFi (10 users)
- VLAN 30: Servers (3 servers)

Inter-VLAN traffic is minimal.
```

---

### L3 Switch (Multilayer Switch)

**Best For:**
- ✅ Medium to large enterprise networks
- ✅ High inter-VLAN traffic requirements
- ✅ Need for wire-speed routing performance
- ✅ Modern campus/data center environments
- ✅ Need for ACLs between VLANs
- ✅ Complex routing requirements
- ✅ Scalability and growth
- ✅ Distribution and Core layers

**Avoid When:**
- ❌ Very tight budget constraints
- ❌ Simple home/small office networks
- ❌ Minimal routing requirements

**Example Use Case:**
```
Corporate campus with multiple buildings:
- VLAN 10: HR (50 users)
- VLAN 20: Sales (100 users)
- VLAN 30: IT (30 users)
- VLAN 40: Finance (40 users)
- VLAN 50: Guest (200 users)
- VLAN 100: Servers (50 servers)
- VLAN 200: VoIP (220 phones)

High inter-VLAN traffic, need for ACLs, QoS, and routing protocols.
```

---

## Three-Tier Network Architecture

Modern enterprise networks typically use L3 switches in a hierarchical design:

```
                    Internet
                        │
                        ▼
            ┌───────────────────────┐
            │     Core Layer        │  ← L3 Switch (High-speed backbone)
            │   (L3 Switches)       │     - Campus-wide routing
            └───────────┬───────────┘     - Redundancy
                        │                 - High bandwidth
            ┌───────────┴───────────┐
            │                       │
    ┌───────▼────────┐     ┌───────▼────────┐
    │ Distribution   │     │ Distribution   │  ← L3 Switches
    │   Layer        │     │   Layer        │     - Inter-VLAN routing
    │ (L3 Switches)  │     │ (L3 Switches)  │     - Policy enforcement
    └───────┬────────┘     └───────┬────────┘     - ACLs, QoS
            │                      │
    ┌───────┴─────┬────────────────┴───────┬─────┐
    │             │                        │     │
┌───▼───┐   ┌───▼───┐                ┌───▼───┐ ┌▼─────┐
│Access │   │Access │                │Access │ │Access│ ← L2 Switches
│Switch │   │Switch │                │Switch │ │Switch│    - End device
│  (L2) │   │  (L2) │                │  (L2) │ │ (L2) │      connections
└───┬───┘   └───┬───┘                └───┬───┘ └┬─────┘    - VLANs
    │           │                        │      │
  [PCs]       [PCs]                    [PCs]  [PCs]
```

**Role Assignment:**
- **Core Layer**: L3 switches for high-speed routing between distribution layers
- **Distribution Layer**: L3 switches for inter-VLAN routing, policy enforcement
- **Access Layer**: L2 switches for end-device connectivity

---

## Advanced Features with L3 Switches

### 1. Access Control Lists (ACLs) Between VLANs

```cisco
! Block HR VLAN from accessing Sales VLAN
Switch(config)# ip access-list extended BLOCK_HR_TO_SALES
Switch(config-ext-nacl)# deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
Switch(config-ext-nacl)# permit ip any any
Switch(config-ext-nacl)# exit

! Apply to SVI VLAN 10 (outbound)
Switch(config)# interface vlan 10
Switch(config-if)# ip access-group BLOCK_HR_TO_SALES out
```

### 2. Quality of Service (QoS)

```cisco
! Prioritize Voice VLAN traffic
Switch(config)# interface vlan 200
Switch(config-if)# ip address 192.168.200.1 255.255.255.0
Switch(config-if)# description Voice VLAN
Switch(config-if)# service-policy output VOICE_PRIORITY
```

### 3. Dynamic Routing Protocols

```cisco
! Configure OSPF for inter-VLAN routing
Switch(config)# router ospf 1
Switch(config-router)# network 192.168.10.0 0.0.0.255 area 0
Switch(config-router)# network 192.168.20.0 0.0.0.255 area 0
Switch(config-router)# network 192.168.30.0 0.0.0.255 area 0
```

### 4. DHCP Server on L3 Switch

```cisco
! Create DHCP pools for each VLAN
Switch(config)# ip dhcp pool VLAN10
Switch(dhcp-config)# network 192.168.10.0 255.255.255.0
Switch(dhcp-config)# default-router 192.168.10.1
Switch(dhcp-config)# dns-server 8.8.8.8

Switch(config)# ip dhcp pool VLAN20
Switch(dhcp-config)# network 192.168.20.0 255.255.255.0
Switch(dhcp-config)# default-router 192.168.20.1
Switch(dhcp-config)# dns-server 8.8.8.8
```

---

## Common Issues and Troubleshooting

### Issue 1: Inter-VLAN Routing Not Working

**Symptom**: Devices in different VLANs cannot communicate

**Checklist**:
```cisco
1. Is IP routing enabled?
   Switch# show running-config | include ip routing
   ! Should show: ip routing

2. Are SVIs up/up?
   Switch# show ip interface brief | include Vlan
   ! Status and Protocol should both be "up"

3. Is the VLAN active and has ports?
   Switch# show vlan brief
   ! VLANs must have status "active"

4. Do PCs have correct default gateway?
   ! Default gateway must be the SVI IP address

5. Check routing table
   Switch# show ip route
   ! Should see connected routes for all VLANs
```

### Issue 2: SVI is Down

**Symptom**: `Vlan10 is down, line protocol is down`

**Causes**:
- VLAN not created: `vlan 10`
- No active ports in the VLAN
- All ports in VLAN are shutdown
- Interface not enabled: `no shutdown`

**Solution**:
```cisco
! Ensure VLAN exists
Switch(config)# vlan 10

! Ensure at least one port is up in the VLAN
Switch(config)# interface fa0/1
Switch(config-if)# switchport access vlan 10
Switch(config-if)# no shutdown

! Ensure SVI is enabled
Switch(config)# interface vlan 10
Switch(config-if)# no shutdown
```

### Issue 3: Can Ping Gateway but Not Other VLANs

**Symptom**: PC can ping its default gateway (SVI) but not devices in other VLANs

**Check**:
```cisco
1. Verify routing table has all VLAN networks
   Switch# show ip route connected

2. Check for ACLs blocking traffic
   Switch# show ip interface vlan 10 | include access list
   
3. Verify destination VLAN SVI is up
   Switch# show ip interface brief

4. Check ARP table
   Switch# show ip arp
```

---

## Summary

### Key Concepts:
- **Inter-VLAN Routing**: Enables communication between different VLANs/subnets
- **L3 Switch (Multilayer Switch)**: A switch that performs both Layer 2 switching and Layer 3 routing
- **SVI (Switched Virtual Interface)**: Virtual interface per VLAN that acts as the default gateway
- **Critical Command**: `ip routing` - enables routing functionality on L3 switch

### Two Methods Comparison:

| Aspect | Router-on-a-Stick | L3 Switch |
|--------|-------------------|-----------|
| **Performance** | Slower (bottleneck) | Wire-speed (hardware) |
| **Scalability** | Limited | Excellent |
| **Cost** | Lower initial cost | Higher initial cost |
| **Complexity** | Moderate | Moderate to High |
| **Best For** | Small networks | Enterprise networks |

### L3 Switch Advantages:
✅ Wire-speed routing without external router bottleneck  
✅ Hardware-based routing using ASICs  
✅ Reduced latency for inter-VLAN traffic  
✅ Simplified network design  
✅ Additional features (ACLs, QoS, routing protocols)  
✅ Scalable for growing networks  

### Remember:
- L2 switches operate at the Data Link layer (MAC addresses)
- L3 switches operate at both Data Link and Network layers (MAC + IP)
- SVIs are virtual interfaces that provide Layer 3 functionality for VLANs
- Always enable `ip routing` on L3 switches for inter-VLAN routing
- Modern enterprise networks prefer L3 switches for distribution and core layers


shruthi
====
On a stick method switch (config)
========
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
!
interface FastEthernet0/2
 switchport access vlan 20
 switchport mode access
!
interface FastEthernet0/3
 switchport trunk allowed vlan 10,20


router (on a stick mode)
=======
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0


 l3 switch mthod
 ==============
 switch
 ====
 interface Vlan10
 ip address 192.168.10.1 255.255.255.0
!
interface Vlan20
 ip address 192.168.20.1 255.255.255.0


 witch(config)#interface f
Switch(config)#interface fastEthernet 0/1
Switch(config-if)#swi
Switch(config-if)#switchport mo
Switch(config-if)#switchport mode  ac
Switch(config-if)#switchport mode  access 
Switch(config-if)#sw
Switch(config-if)#switchport ac
Switch(config-if)#switchport access vlan
Switch(config-if)#switchport access vlan 10
% Access VLAN does not exist. Creating vlan 10
Switch(config-if)#no shut
Switch(config-if)#
Switch(config-if)#
Switch(config-if)#exit
Switch(config)#
Switch(config)#
Switch(config)#sw
Switch(config)#in
Switch(config)#interface gi
Switch(config)#interface f
Switch(config)#interface fastEthernet 0/2
Switch(config-if)#sw
Switch(config-if)#switchport mo
Switch(config-if)#switchport mode a
Switch(config-if)#switchport mode access 
Switch(config-if)#sw
Switch(config-if)#switchport a
Switch(config-if)#switchport access vl
Switch(config-if)#switchport access vlan 20
% Access VLAN does not exist. Creating vlan 20
Switch(config-if)#no shut
Switch(config-if)#
Switch(config-if)#
Switch(config-if)#exit
Switch(config)#
Switch(config)#
Switch(config)#ip ro
Switch(config)#ip ro
Switch(config)#ip ro
Switch(config)#ip rou
Switch(config)#ip routing
Switch(config)#
Switch(config)#
Switch(config)#in
Switch(config)#interface vl
Switch(config)#interface vlan 10
Switch(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan10, changed state to up
ip add
Switch(config-if)#ip address 192.168.10.1 255.255.255.0
Switch(config-if)#no shut
Switch(config-if)#
Switch(config-if)#exit
Switch(config)#
Switch(config)#in
Switch(config)#interface vl
Switch(config)#interface vlan 20
Switch(config-if)#
%LINK-5-CHANGED: Interface Vlan20, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan20, changed state to up
i
% Ambiguous command: "i"
Switch(config)#ip ad
Switch(config)#in
Switch(config)#interface vl
Switch(config)#interface vlan 20
Switch(config-if)#
Switch(config-if)#ip ad
Switch(config-if)#ip address 192.168.20.1 255.255.255.0
Switch(config-if)#no shut
Switch(config-if)#exit
Switch(config)#
Switch(config)#
Switch(config)#exit
Switch#
%SYS-5-CONFIG_I: Configured from console by console


