#  VLAN & Inter-VLAN Routing Lab (Router-on-a-Stick)

## Overview

This project demonstrates VLAN segmentation and inter-VLAN routing using Cisco Packet Tracer.

It simulates a small enterprise network using:
- Cisco 2811 Router (Layer 3 routing)
- Cisco 2960 Switch (Layer 2 switching)
- Multiple VLANs with trunking and Router-on-a-Stick configuration

The lab focuses on:
- Layer 2 segmentation (VLANs)
- Layer 3 routing (inter-VLAN communication)
- Structured troubleshooting methodology

---

#  Network Topology (Logical)

            (Layer 3 - Routing)
             CISCO 2811 ROUTER
                    |
                    | 802.1Q TRUNK (Layer 2/3 boundary)
                    |
            CISCO 2960 SWITCH (Layer 2)
    ------------------------------------------------
    |                    |                         |

VLAN 10 VLAN 20 VLAN 30
PC1 PC2 PC3
192.168.10.10 192.168.20.10 192.168.30.10


---

#  VLAN DESIGN

| VLAN | Role | Network | Gateway |
|------|------|--------|---------|
| 10 | Users A | 192.168.10.0/24 | 192.168.10.1 |
| 20 | Users B | 192.168.20.0/24 | 192.168.20.1 |
| 30 | Users C | 192.168.30.0/24 | 192.168.30.1 |

---

#  SWITCH CONFIGURATION (Layer 2 Device - Cisco 2960)

##  VLAN Creation (Layer 2 Switching)

```bash
enable
configure terminal

vlan 10
 name VLAN10

vlan 20
 name VLAN20

vlan 30
 name VLAN30
```

# Access Port Assignment (Layer 2)

Assigning PCs to VLANs:
```bash
interface fa0/1
 switchport mode access
 switchport access vlan 10

interface fa0/2
 switchport mode access
 switchport access vlan 20

interface fa0/3
 switchport mode access
 switchport access vlan 30
 ```

# Trunk Configuration (Layer 2 - 802.1Q tagging)
```bash
interface fa0/24
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30
 ```

# SWITCH TROUBLESHOOTING COMMANDS (Layer 2)
VLAN Verification
```bash
show vlan brief
Trunk Verification
show interfaces trunk
MAC Address Table (Layer 2 forwarding check)
show mac address-table
Interface Status
show interfaces status
```

# ROUTER CONFIGURATION (Layer 3 Device - Cisco 2811)
#  Physical Interface (Layer 3 boundary)
```bash
enable
configure terminal

interface g0/0
 no ip address
 no shutdown
 ```

# Subinterfaces (Router-on-a-Stick - Layer 3 routing per VLAN)
```bash
VLAN 10 Interface
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
VLAN 20 Interface
interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
VLAN 30 Interface
interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ```

# ROUTER TROUBLESHOOTING COMMANDS (Layer 3)
```bash
Interface Status
show ip interface brief
Routing / Subinterface Validation
show running-config
ARP Table (Layer 3 to Layer 2 mapping)
show arp
Interface-specific details
show interfaces g0/0
show interfaces g0/0.10
show interfaces g0/0.20
```
# END DEVICE CONFIGURATION (Layer 3 Host Level)
```bash
Device	IP	Subnet	Gateway
PC1	192.168.10.10	/24	192.168.10.1
PC2	192.168.20.10	/24	192.168.20.1
PC3	192.168.30.10	/24	192.168.30.1
```

# CONNECTIVITY TESTING (End-to-End Validation)
```bash
Layer 3 Gateway Tests
ping 192.168.10.1
ping 192.168.20.1
ping 192.168.30.1
Inter-VLAN Routing Tests
ping 192.168.20.10
ping 192.168.30.10
```

# TROUBLESHOOTING METHODOLOGY (REAL DEBUG FLOW)
```bash
Step 1 — Layer 1 (Physical)
Check cables
Check interface status
show interfaces status
Step 2 — Layer 2 (Switching Issues)
VLAN assignment
trunk status
MAC learning
show vlan brief
show interfaces trunk
show mac address-table
Step 3 — Layer 3 (Routing Issues)
Subinterface configuration
IP addressing
routing table / ARP
show ip interface brief
show arp
Step 4 — End Device (Most Common Failure Point)
IP address correctness
Subnet mask
Default gateway
```

# REAL ISSUE ENCOUNTERED
Symptom

PC2 could not ping its gateway (192.168.20.1)

Investigation Path
VLAN configuration verified (Layer 2 OK)
Trunk verified (Layer 2 OK)
Router subinterfaces verified (Layer 3 OK)
ARP tables verified (OK)
Root Cause

Incorrect host IP configuration:
```bash
WRONG: 192.169.20.10
CORRECT: 192.168.20.10
```
Resolution

Corrected PC IP addressing → full connectivity restored

# KEY LEARNINGS
VLANs operate at Layer 2 and isolate broadcast domains
Router-on-a-stick provides Layer 3 inter-VLAN routing
802.1Q tagging is required for trunk links
Most network failures in lab environments are not routing issues — they are host configuration issues
Systematic troubleshooting (L1 → L2 → L3 → Host) is critical in network engineering
#  FINAL STATUS

✔ VLAN segmentation successful
✔ Inter-VLAN routing functional
✔ End-to-end connectivity verified
✔ Real troubleshooting scenario documented

