# SNMP (Simple Network Management Protocol) - Beginner's Guide

## What is SNMP?

SNMP stands for **Simple Network Management Protocol**. It's a standardized protocol that allows network administrators to remotely monitor and manage network devices (routers, switches, servers, firewalls, access points) from a central management station.

### Networking Scenario
In a typical enterprise network with 50 switches and 20 routers:
- **SNMP Manager** = Network Management System (NMS) running on administrator's workstation
- **SNMP Agents** = Software running on each of the 70 network devices
- **Communication** = Manager polls devices every 5 minutes to collect performance data
- **Alerts** = When a switch port goes down, the switch sends an SNMP trap to the NMS immediately

Without SNMP, you'd need to manually log into each device via CLI to check status. With SNMP, all 70 devices report to one central location automatically.

---

## Key Components of SNMP

### 1. **SNMP Manager** (The Boss)
- The central monitoring system/software
- Asks questions and collects information
- Examples: SolarWinds, PRTG, Nagios
- Usually runs on an administrator's computer

### 2. **SNMP Agent** (The Reporter)
- Software running on each network device
- Listens for requests from the manager
- Sends back information about the device
- Can also send alerts when something goes wrong

### 3. **MIB (Management Information Base)**
- Think of it as a **dictionary** or **menu**
- Contains all the information that can be monitored
- Each piece of information has a unique ID (OID)
- Example: "CPU usage" might be at OID 1.3.6.1.4.1.9.2.1.58

---

## How SNMP Works (Simple Explanation)

### Scenario 1: Manager Polling Router
```
NMS sends GET request to 192.168.1.1 (Router) on UDP port 161
Request: "What is your CPU usage?" (OID: 1.3.6.1.4.1.9.2.1.56.0)
Router Agent responds: "45%" 
NMS logs data and creates graph
Process repeats every 5 minutes
```

### Scenario 2: Switch Sends Trap
```
Switch port Gi0/24 goes down
Switch Agent immediately sends TRAP to NMS on UDP port 162
Trap message: "linkDown on interface index 24"
NMS receives trap and sends email alert to admin
Admin investigates and finds unplugged cable
```

## Simple SNMP Workflow

```
1. Install SNMP manager software on your computer
2. Enable SNMP agent on network devices
3. Configure community strings (or SNMPv3 credentials)
4. Manager starts polling devices every few minutes
5. Manager stores data and creates graphs
6. If something goes wrong, device sends trap
7. Administrator receives alert and investigates
```
---

## SNMP Operations (Commands)

| Operation | What It Does | Network Example |
|-----------|-------------|-----------------|
| **GET** | Retrieve a specific value | Get interface Gi0/1 status |
| **GET-NEXT** | Get the next value in sequence | Get next interface in the list |
| **GET-BULK** | Get multiple values at once | Get all interface statistics |
| **SET** | Change a configuration | Change interface description |
| **TRAP** | Device sends urgent alert | Switch sends "link down" alert |

---

## SNMP Versions

### SNMPv1 (Original - 1988)
- ✅ Simple and widely supported
- ❌ **No security** (passwords sent in plain text)
- ❌ Limited error handling
- 🔑 Uses "community strings" (like passwords)

### SNMPv2c (Improved - 1993)
- ✅ Better performance
- ✅ Bulk operations (faster data collection)
- ❌ Still **no encryption**
- 🔑 Still uses community strings

### SNMPv3 (Most Secure - 2004)
- ✅ **Encryption** (data is protected)
- ✅ **Authentication** (verify who's asking)
- ✅ **Privacy** (messages are encrypted)
- ✅ Most recommended today
- 🔑 Uses usernames, passwords, and encryption keys
Community strings act as **shared passwords** between NMS and network devices:

- **Read-Only Community** (default: "public")
  - Allows GET operations only
  - Example: `snmpget -v2c -c public 192.168.1.1 sysUpTime`
  - Safe for monitoring, cannot modify configurations
  
- **Read-Write Community** (default: "private")
  - Allows GET and SET operations
  - Example: `snmpset -v2c -c private 192.168.1.1 ifAdminStatus.1 down`
  - Can change device configurations remotely
  - ⚠️ Highly sensitive - can be used to reconfigure or disable devices!

**Security Warning**: Community strings are sent in plaintext. Anyone with a packet sniffer can capture them. Default strings "public" and "private" should always be changed to complex strings.
  - Allows viewing AND changing settings
  - Like giving administrator access
  - ⚠️ Should be protected carefully!

**Security Warning**: Default community strings like "public" and "private" are well-known and insecure. Always change them!

---

## SNMP Port Numbers

| Port | Protocol | Purpose |
|------|----------|---------|
| **161** | UDP | Manager → Agent (requests) |
| **162** | UDP | Agent → Manager (traps/alerts) |

---

## Configuring SNMP on Routers

### Basic SNMPv2c Configuration (Cisco IOS)

```cisco
! Enter global configuration mode
Router> enable
Router# configure terminal

! Set read-only community string
Router(config)# snmp-server community NetworkMon_RO RO

! Set read-write community string (use with caution!)
Router(config)# snmp-server community NetworkAdmin_RW RW


! Enable SNMP traps for various events
Router(config)# snmp-server enable traps cpu threshold

! Specify trap receiver (NMS server)
Router(config)# snmp-server host 192.168.100.10 version 2c NetworkMon_RO

! Enable SNMP trap source interface
Router(config)# snmp-server trap-source GigabitEthernet0/0

! Save configuration
Router(config)# end
Router# write memory
```

### SNMPv3 Configuration (More Secure)

```cisco
! Enter global configuration mode
Router# configure terminal

! Create SNMPv3 view (defines what OIDs can be accessed)
Router(config)# snmp-server view FULL iso included

! Create SNMPv3 group with authentication and encryption
Router(config)# snmp-server group SNMPV3-RO v3 priv read FULL

! Create SNMPv3 user with authentication and encryption
! Authentication: SHA, Privacy: AES 128-bit
Router(config)# snmp-server user snmpadmin SNMPV3-RO v3 auth sha AuthPass123! priv aes 128 PrivPass456!

! Enable SNMPv3 traps to NMS
Router(config)# snmp-server host 192.168.100.10 version 3 priv snmpadmin


! Enable specific traps for SNMPv3
Router(config)# snmp-server enable traps cpu threshold
Router(config)# snmp-server enable traps config

! Save configuration
Router(config)# end
Router# write memory
```




### Verification Commands

```cisco
! Verify SNMP configuration
Router# show snmp

! Show SNMP community strings
Router# show snmp community

! Show SNMP groups (for SNMPv3)
Router# show  snmp group

! Show SNMP users (for SNMPv3)
Router# show snmp user

! Show SNMP host (trap receivers)
Router# show snmp host

! Show SNMP statistics
Router# show snmp stats

! Show SNMP sessions (active SNMP queries)
Router# show snmp sessions

! Test SNMP from router itself (if supported)
Router# show snmp mib ifmib ifindex GigabitEthernet0/0
```

### Testing SNMP from NMS/Linux Server

```bash
# Test SNMPv2c connectivity
snmpwalk -v2c -c NetworkMon_RO 192.168.1.1

# Get specific OID (system description)
snmpget -v2c -c NetworkMon_RO 192.168.1.1 sysDescr.0

# Test SNMPv3 with authentication and encryption
snmpwalk -v3 -l authPriv -u snmpadmin -a SHA -A "AuthPass123!" -x AES -X "PrivPass456!" 192.168.1.1

# Test SNMPv3 GET operation
snmpget -v3 -l authPriv -u snmpadmin -a SHA -A "]!" -x AES -X "PrivPass456!" 192.168.1.1 sysUpTime.0

# Bulk GET (get multiple OIDs at once)
snmpbulkget -v2c -c NetworkMon_RO 192.168.1.1 ifDescr ifSpeed
```







---


Router>enable 
Router#
Router#
Router#
Router#config
Configuring from terminal, memory, or network [terminal]? 
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#
Router(config)#
Router(config)#


