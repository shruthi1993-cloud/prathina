# MAC Address Learning Process

## Network Topology

```
        PC-A                    PC-B                    PC-C
    MAC: aaaa.aaaa.aaaa     MAC: bbbb.bbbb.bbbb     MAC: cccc.cccc.cccc
    IP: 192.168.1.10        IP: 192.168.1.20        IP: 192.168.1.30
           |                        |                        |
           |                        |                        |
         Fa0/1                    Fa0/2                    Fa0/3
           |                        |                        |
           +------------------------+------------------------+
                                    |
                              [ SWITCH ]
                            MAC Address Table
                            +----------+------+
                            |   MAC    | Port |
                            +----------+------+
                            |          |      |
                            +----------+------+
```

## How MAC Learning Works

A switch learns MAC addresses by examining the **SOURCE MAC address** of incoming frames and associating it with the port on which the frame arrived.

---

## Step-by-Step MAC Learning Process

### Initial State
```
MAC Address Table: EMPTY

PC-A ─────── Fa0/1 ──┐
                     │
PC-B ─────── Fa0/2 ──┤  [ SWITCH ]
                     │  Table: Empty
PC-C ─────── Fa0/3 ──┘
```

---

### Step 1: PC-A sends frame to PC-B

**PC-A → PC-B**: PC-A sends an ARP request or data frame

```
Frame Details:
┌─────────────────────────────────┐
│ Source MAC: aaaa.aaaa.aaaa     │  ← Switch learns this
│ Dest MAC:   bbbb.bbbb.bbbb     │  ← Switch doesn't know this port yet
└─────────────────────────────────┘

PC-A ─────── Fa0/1 ──┐  Frame arrives on Fa0/1
        📨           │
PC-B ─────── Fa0/2 ──┤  [ SWITCH ]
                     │  
PC-C ─────── Fa0/3 ──┘  
```

**What the switch does:**

1. **LEARNS SOURCE MAC**: 
   - Reads source MAC: aaaa.aaaa.aaaa
   - Associates it with port Fa0/1
   - Adds to MAC address table

2. **CHECKS DESTINATION MAC**:
   - Looks for bbbb.bbbb.bbbb in table
   - NOT FOUND!

3. **FLOODS THE FRAME**:
   - Forwards frame out ALL ports except the incoming port (Fa0/1)
   - Sends to Fa0/2 and Fa0/3

```
MAC Address Table:
+-------------------+-------+
|       MAC         | Port  |
+-------------------+-------+
| aaaa.aaaa.aaaa    | Fa0/1 | ← LEARNED
+-------------------+-------+

PC-A ─────── Fa0/1      (incoming port - no flood here)
        
PC-B ─────── Fa0/2 ──┐  📨 Frame flooded
                     │
PC-C ─────── Fa0/3 ──┘  📨 Frame flooded
```

---

### Step 2: PC-B replies to PC-A

**PC-B → PC-A**: PC-B sends a reply frame

```
Frame Details:
┌─────────────────────────────────┐
│ Source MAC: bbbb.bbbb.bbbb     │  ← Switch learns this
│ Dest MAC:   aaaa.aaaa.aaaa     │  ← Switch knows this! (Fa0/1)
└─────────────────────────────────┘

PC-A ─────── Fa0/1      
                     
PC-B ─────── Fa0/2 ──┐  Frame arrives on Fa0/2
        📨           │  
PC-C ─────── Fa0/3 ──┘  [ SWITCH ]
```

**What the switch does:**

1. **LEARNS SOURCE MAC**:
   - Reads source MAC: bbbb.bbbb.bbbb
   - Associates it with port Fa0/2
   - Adds to MAC address table

2. **CHECKS DESTINATION MAC**:
   - Looks for aaaa.aaaa.aaaa in table
   - FOUND on port Fa0/1!

3. **FORWARDS FRAME**:
   - Sends frame ONLY out port Fa0/1 (unicast)
   - Does NOT flood

```
MAC Address Table:
+-------------------+-------+
|       MAC         | Port  |
+-------------------+-------+
| aaaa.aaaa.aaaa    | Fa0/1 |
| bbbb.bbbb.bbbb    | Fa0/2 | ← LEARNED
+-------------------+-------+

PC-A ─────── Fa0/1  📨   (frame forwarded here only)
                     
PC-B ─────── Fa0/2       (incoming port)
                         
PC-C ─────── Fa0/3       (no frame sent - efficient!)
```

---

### Step 3: PC-C sends frame to PC-A

**PC-C → PC-A**: PC-C wants to communicate with PC-A

```
Frame Details:
┌─────────────────────────────────┐
│ Source MAC: cccc.cccc.cccc     │  ← Switch learns this
│ Dest MAC:   aaaa.aaaa.aaaa     │  ← Switch knows this! (Fa0/1)
└─────────────────────────────────┘

PC-A ─────── Fa0/1      
                     
PC-B ─────── Fa0/2       
                     
PC-C ─────── Fa0/3 ──┐  Frame arrives on Fa0/3
        📨           │
                     └── [ SWITCH ]
```

**What the switch does:**

1. **LEARNS SOURCE MAC**:
   - Reads source MAC: cccc.cccc.cccc
   - Associates it with port Fa0/3
   - Adds to MAC address table

2. **CHECKS DESTINATION MAC**:
   - Looks for aaaa.aaaa.aaaa in table
   - FOUND on port Fa0/1!

3. **FORWARDS FRAME**:
   - Sends frame ONLY to Fa0/1

```
MAC Address Table: COMPLETE!
+-------------------+-------+
|       MAC         | Port  |
+-------------------+-------+
| aaaa.aaaa.aaaa    | Fa0/1 |
| bbbb.bbbb.bbbb    | Fa0/2 |
| cccc.cccc.cccc    | Fa0/3 | ← LEARNED
+-------------------+-------+

PC-A ─────── Fa0/1  📨   (frame forwarded here only)
                     
PC-B ─────── Fa0/2       (no frame - efficient!)
                         
PC-C ─────── Fa0/3       (incoming port)
```

---

## MAC Learning Summary

### Key Points:

1. **Learning**: Switch learns from SOURCE MAC addresses
2. **Forwarding Decision**: Switch uses DESTINATION MAC addresses
3. **Unknown Unicast Flooding**: If destination MAC is unknown, flood to all ports except incoming
4. **Known Unicast**: If destination MAC is known, forward only to that specific port
5. **Aging**: MAC addresses age out after 300 seconds (default) of inactivity

### Frame Processing Steps:

```
┌─────────────────────────────────────────────┐
│  1. Frame arrives on port                   │
│                                             │
│  2. Learn SOURCE MAC + Port                │
│     └─> Add/Update MAC Address Table       │
│                                             │
│  3. Check DESTINATION MAC                   │
│     ├─> Known? Forward to specific port    │
│     ├─> Unknown? Flood to all ports        │
│     ├─> Broadcast? Flood to all ports      │
│     └─> Multicast? Forward per config      │
│                                             │
│  4. Forward frame                           │
└─────────────────────────────────────────────┘
```

---

## MAC Address Table Behavior

### Aging Timer (Default: 300 seconds = 5 minutes)
```
If no frames with a particular source MAC are seen for 300 seconds,
that entry is removed from the MAC address table.

Example:
Time 0s:   MAC aaaa.aaaa.aaaa learned on Fa0/1
Time 150s: PC-A sends another frame → Timer resets to 0
Time 300s: No activity → Entry removed from table
```

### Cisco Commands to Verify:
```cisco
! View MAC address table
Switch# show mac address-table

! View dynamic MAC addresses only
Switch# show mac address-table dynamic

! View MAC addresses for specific VLAN
Switch# show mac address-table vlan 10

! View MAC addresses for specific interface
Switch# show mac address-table interface fastEthernet 0/1

! Clear MAC address table
Switch# clear mac address-table dynamic

! Change aging time (in seconds)
Switch(config)# mac address-table aging-time 600

! Disable aging
Switch(config)# mac address-table aging-time 0
```

---

## Broadcast vs Unicast Examples

### Broadcast Frame (FF:FF:FF:FF:FF:FF)
```
PC-A sends broadcast (ARP request):

┌──────────────────────────────────────┐
│ Source MAC: aaaa.aaaa.aaaa          │
│ Dest MAC:   ffff.ffff.ffff (Broadcast) │
└──────────────────────────────────────┘

PC-A ─────── Fa0/1 ──┐  📨 incoming
                     │
PC-B ─────── Fa0/2 ──┤  📨 flooded (broadcast)
                     │
PC-C ─────── Fa0/3 ──┘  📨 flooded (broadcast)

Result: Frame sent to ALL ports except Fa0/1
```

### Unknown Unicast
```
PC-A sends to unknown MAC (dddd.dddd.dddd):

┌──────────────────────────────────────┐
│ Source MAC: aaaa.aaaa.aaaa          │
│ Dest MAC:   dddd.dddd.dddd (Unknown)│
└──────────────────────────────────────┘

PC-A ─────── Fa0/1 ──┐  📨 incoming
                     │
PC-B ─────── Fa0/2 ──┤  📨 flooded (unknown)
                     │
PC-C ─────── Fa0/3 ──┘  📨 flooded (unknown)

Result: Flooded because dddd.dddd.dddd not in MAC table
```

### Known Unicast
```
PC-A sends to PC-B (bbbb.bbbb.bbbb) - already in table:

┌──────────────────────────────────────┐
│ Source MAC: aaaa.aaaa.aaaa          │
│ Dest MAC:   bbbb.bbbb.bbbb (Known) │
└──────────────────────────────────────┘

PC-A ─────── Fa0/1 ──┐  📨 incoming
                     │
PC-B ─────── Fa0/2 ──┤  📨 forwarded (unicast) ✓
                     │
PC-C ─────── Fa0/3 ──┘  ❌ no frame (efficient!)

Result: Forwarded ONLY to Fa0/2 (specific port)
```

---

## Additional Concepts

### MAC Address Table Size
- Typical switches support 8,000 - 128,000 MAC addresses
- Enterprise switches can support millions of entries
- View capacity: `show mac address-table count`

### Static vs Dynamic Entries
- **Dynamic**: Learned automatically, subject to aging
- **Static**: Manually configured, never age out
```cisco
! Add static MAC entry
Switch(config)# mac address-table static aaaa.aaaa.aaaa vlan 10 interface fa0/1
```

### Security Consideration - MAC Flooding Attack
If an attacker floods the switch with thousands of fake MAC addresses:
- MAC address table fills up
- Switch may fail to "fail-open" mode
- Starts flooding ALL frames like a hub
- **Defense**: Port security feature limits MAC addresses per port
