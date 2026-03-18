# VLAN Concepts - Complete Guide

## What is a VLAN?

A **VLAN (Virtual Local Area Network)** is a logical grouping of network devices that creates separate broadcast domains at Layer 2, regardless of their physical location.

### Without VLANs:
```
All devices in ONE broadcast domain (VLAN 1 - default)

[PC-HR] ─────┐
[PC-Sales] ──┤
[PC-IT] ─────┤──── [ SWITCH ] ──── All traffic visible to all
[PC-Finance]─┤
[Server] ────┘

Problem: All broadcast traffic reaches ALL devices
```

### With VLANs:
```
Separate broadcast domains by department

VLAN 10 (HR)        VLAN 20 (Sales)      VLAN 30 (IT)
[PC-HR1] ──┐        [PC-Sales1] ──┐      [PC-IT1] ──┐
[PC-HR2] ──┤        [PC-Sales2] ──┤      [PC-IT2] ──┤
           │                      │                 │
           └──────────────────────┴─────────────────┘
                        [ SWITCH ]
                    (Multiple VLANs)

Benefit: Broadcasts in VLAN 10 do NOT reach VLAN 20 or 30
```

---

## Why Use VLANs?

### 1. **Security**
Isolate sensitive traffic (Finance, HR) from general network

### 2. **Performance**
Reduce broadcast domain size = less broadcast traffic

### 3. **Flexibility**
Group users logically, not by physical location

### 4. **Cost Savings**
No need for separate physical switches per department

### 5. **Simplified Management**
Easier to apply policies per VLAN

---

## VLAN Table Structure

The VLAN table stores information about configured VLANs on the switch.

### VLAN Database Fields:
```
+----------+-------------+--------+-----------+
| VLAN ID  | VLAN Name   | Status | Ports     |
+----------+-------------+--------+-----------+
| 1        | default     | active | All ports |
| 10       | HR          | active | Fa0/1-5   |
| 20       | Sales       | active | Fa0/6-10  |
| 30       | IT          | active | Fa0/11-15 |
| 99       | Management  | active | Vlan99    |
+----------+-------------+--------+-----------+

Stored in: vlan.dat file (flash memory)
```

### Cisco Command to View VLAN Table:
```cisco
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- ------------------------
1    default                          active    Fa0/16, Fa0/17, Fa0/18
10   HR                               active    Fa0/1, Fa0/2, Fa0/3
20   Sales                            active    Fa0/6, Fa0/7, Fa0/8
30   IT                               active    Fa0/11, Fa0/12
99   Management                       active    
```

---

## VLAN Types

### 1. Data VLAN
- Carries user-generated data traffic
- Most common VLAN type
- Example: VLAN 10 for Sales department

### 2. Voice VLAN
- Dedicated for VoIP traffic
- QoS priority for voice packets
- Example: VLAN 150 for IP phones

### 3. Management VLAN
- Used for switch management (SSH, Telnet, SNMP)
- Should NOT be VLAN 1 (security best practice)
- Example: VLAN 99 for network admin access

### 4. Default VLAN
- VLAN 1 - exists by default
- All ports are in VLAN 1 initially
- Cannot be deleted or renamed

---


## Network Topology with VLANs

### Scenario: Multi-Department Network

```
Floor 1:                              Floor 2:
┌─────────────────┐                  ┌─────────────────┐
│   VLAN 10 (HR)  │                  │  VLAN 10 (HR)   │
│   192.168.10.0  │                  │  192.168.10.0   │
├─────────────────┤                  ├─────────────────┤
│ PC-HR-1         │                  │ PC-HR-3         │
│ .10.11          │                  │ .10.13          │
│                 │                  │                 │
│ PC-HR-2         │                  │ PC-HR-4         │
│ .10.12          │                  │ .10.14          │
└────────┬────────┘                  └────────┬────────┘
         │                                    │
     Fa0/1-2                              Fa0/1-2
         │                                    │
    ┌────┴─────┐                        ┌────┴─────┐
    │ Switch 1 │                        │ Switch 2 │
    │          │                        │          │
    └────┬─────┘                        └────┬─────┘
         │                                    │
     Fa0/11-12                            Fa0/11-12
         │                                    │
┌────────┴────────┐                  ┌────────┴────────┐
│  VLAN 20 (Sales)│                  │ VLAN 20 (Sales) │
│  192.168.20.0   │                  │ 192.168.20.0    │
├─────────────────┤                  ├─────────────────┤
│ PC-Sales-1      │                  │ PC-Sales-3      │
│ .20.11          │                  │ .20.13          │
│                 │                  │                 │
│ PC-Sales-2      │                  │ PC-Sales-4      │
│ .20.12          │                  │ .20.14          │
└─────────────────┘                  └─────────────────┘

         Gi0/1                               Gi0/1
           │         TRUNK LINK                │
           │    (Carries all VLANs)            │
           └──────────────┬───────────────────┘
                          │
                    ┌─────┴─────┐
                    │  Router   │
                    │ (Layer 3) │
                    └───────────┘
                  Inter-VLAN Routing
```

---

## Access Ports vs Trunk Ports

### Access Port
- Belongs to **ONE VLAN only**
- Connects **end devices** (PCs, printers, phones)
- Sends/receives **untagged** frames
- Default: VLAN 1

#### Access Port Configuration:
```
         PC-A (VLAN 10)
             │
             │ Untagged frames
             │
         Fa0/1 (Access)
             │
        [ SWITCH ]
```

```cisco
Switch(config)# interface fastEthernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# description PC-HR-1
```

---

### Trunk Port
- Carries **MULTIPLE VLANs**
- Connects **switch-to-switch** or **switch-to-router**
- Uses **802.1Q tagging** (adds VLAN tag to frames)
- Transports traffic for all allowed VLANs

#### Trunk Port Configuration:
```
    [ SWITCH 1 ]                    [ SWITCH 2 ]
         │                               │
     Gi0/1 (Trunk)                   Gi0/1 (Trunk)
         │                               │
         └───────── Tagged frames ───────┘
            VLAN 10 | VLAN 20 | VLAN 30
```

```cisco
Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk encapsulation dot1q    ! Some platforms
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# description Trunk to Switch2
```

---

## 802.1Q Frame Tagging

### Untagged Frame (Access Port):
```
┌────────────┬────────────┬──────┬─────┬─────┐
│   Dest     │   Source   │ Type │ Data│ FCS │
│    MAC     │    MAC     │      │     │     │
└────────────┴────────────┴──────┴─────┴─────┘
  6 bytes      6 bytes    2 bytes      4 bytes
```

### Tagged Frame (Trunk Port - 802.1Q):
```
┌────────────┬────────────┬─────────────┬──────┬─────┬─────┐
│   Dest     │   Source   │  802.1Q Tag │ Type │ Data│ FCS │
│    MAC     │    MAC     │  (4 bytes)  │      │     │     │
└────────────┴────────────┴─────────────┴──────┴─────┴─────┘
  6 bytes      6 bytes        4 bytes    2 bytes      4 bytes

802.1Q Tag Breakdown:
┌─────────┬─────┬─────┬────────────┐
│   TPID  │ PCP │ DEI │  VLAN ID   │
│ 2 bytes │3bits│1bit │  12 bits   │
└─────────┴─────┴─────┴────────────┘
└─ 0x8100  └─ Priority  └─ 0-4095 VLANs
```

---

## Creating and Assigning VLANs

### Step-by-Step Example:

#### Scenario:
```
Department    VLAN ID    Subnet           Ports
---------     -------    --------------   -----------
HR            10         192.168.10.0/24  Fa0/1-5
Sales         20         192.168.20.0/24  Fa0/6-10
IT            30         192.168.30.0/24  Fa0/11-15
Management    99         192.168.99.0/24  SVI only
Trunk         -          All VLANs        Gi0/1
```

### Configuration Commands:

```cisco
! ========================================
! Step 1: Create VLANs
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
! Step 2: Assign Ports to VLANs (Access)
! ========================================

! HR Department - VLAN 10
Switch(config)# interface range fastEthernet 0/1-5
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 10
Switch(config-if-range)# description HR Department
Switch(config-if-range)# exit

! Sales Department - VLAN 20
Switch(config)# interface range fastEthernet 0/6-10
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 20
Switch(config-if-range)# description Sales Department
Switch(config-if-range)# exit

! IT Department - VLAN 30
Switch(config)# interface range fastEthernet 0/11-15
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 30
Switch(config-if-range)# description IT Department
Switch(config-if-range)# exit

! ========================================
! Step 3: Configure Trunk Port
! ========================================
Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30,99
Switch(config-if)# description Trunk to Distribution Switch
Switch(config-if)# exit

! ========================================
! Step 4: Create Management Interface (SVI)
! ========================================
Switch(config)# interface vlan 99
Switch(config-if)# ip address 192.168.99.10 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# description Management Interface
Switch(config-if)# exit

Switch(config)# ip default-gateway 192.168.99.1
Switch(config)# exit

! ========================================
! Step 5: Save Configuration
! ========================================
Switch# copy running-config startup-config
```

---

## VLAN Verification Commands

### 1. View All VLANs
```cisco
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/16, Fa0/17, Fa0/18, Fa0/19
10   HR                               active    Fa0/1, Fa0/2, Fa0/3, Fa0/4, Fa0/5
20   Sales                            active    Fa0/6, Fa0/7, Fa0/8, Fa0/9, Fa0/10
30   IT                               active    Fa0/11, Fa0/12, Fa0/13, Fa0/14
99   Management                       active    
```

### 2. View Specific VLAN
```cisco
Switch# show vlan id 10

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
10   HR                               active    Fa0/1, Fa0/2, Fa0/3, Fa0/4, Fa0/5

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
10   enet  100010     1500  -      -      -        -    -        0      0
```

