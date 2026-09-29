# Project Overview

This lab demonstrates the design and configuration of a small routed network using subnetting (VLSM), router interface configuration, and static routing. The objective was to divide a single network into multiple LANs and router-to-router links, configure routing between them, and verify connectivity using ping and traceroute.

This project demonstrates practical understanding of:

Subnetting and VLSM
Router interface configuration
Static routing
Packet flow and routing logic
Network troubleshooting
Basic network design methodology
# Network Topology

The network consists of:

3 Routers
3 Switches
6 PCs
1 Server
3 LAN networks
2 point-to-point router links
Topology Diagram

(Insert Packet Tracer topology screenshot here)

ASCII Topology
```bash
LAN1 (192.168.10.0/29)
PC1     PC2
 |       |
 +---Switch1---+
               |
             R1 Fa0/0
             |
      Fa0/1 .25 /30
             |
             .26 Fa0/1 R2
             |
         Switch2 (LAN2)
         PC3    PC4

R1 Fa1/0 .29 /30
             |
             .30 Gi0/0 R3
             |
         Switch3 (LAN3)
         PC5    PC6   Server
```

# Subnetting Plan (VLSM)

```bash
Base network: 192.168.10.0/24

Network	Subnet	Mask	Usable Hosts	Broadcast	Purpose
LAN1	192.168.10.0/29	255.255.255.248	.1–.6	.7	PCs LAN1
LAN2	192.168.10.8/29	255.255.255.248	.9–.14	.15	PCs LAN2
LAN3	192.168.10.16/29	255.255.255.248	.17–.22	.23	PCs LAN3
R1–R2	192.168.10.24/30	255.255.255.252	.25–.26	.27	Router link
R1–R3	192.168.10.28/30	255.255.255.252	.29–.30	.31	Router link
Subnetting Formula

Usable hosts formula:

Usable Hosts = 2^(Host Bits) - 2

Examples:

/29 → 6 usable hosts
/30 → 2 usable hosts (perfect for router links)
```

# IP Address Assignment

Routers
Router	Interface	IP Address	Subnet
```bash
R1	Fa0/0	192.168.10.1	/29
R1	Fa0/1	192.168.10.25	/30
R1	Fa1/0	192.168.10.29	/30
R2	Fa0/0	192.168.10.9	/29
R2	Fa0/1	192.168.10.26	/30
R3	Gi0/1	192.168.10.17	/29
R3	Gi0/0	192.168.10.30	/30
PCs
Device	IP Address	Gateway
PC1	192.168.10.2	192.168.10.1
PC2	192.168.10.3	192.168.10.1
PC3	192.168.10.10	192.168.10.9
PC4	192.168.10.11	192.168.10.9
PC5	192.168.10.18	192.168.10.17
PC6	192.168.10.19	192.168.10.17
```

# Router Configurations
R1
```bash
interface Fa0/0
 ip address 192.168.10.1 255.255.255.248
 no shutdown

interface Fa0/1
 ip address 192.168.10.25 255.255.255.252
 no shutdown

interface Fa1/0
 ip address 192.168.10.29 255.255.255.252
 no shutdown
R2
interface Fa0/0
 ip address 192.168.10.9 255.255.255.248
 no shutdown

interface Fa0/1
 ip address 192.168.10.26 255.255.255.252
 no shutdown
R3
interface Gi0/1
 ip address 192.168.10.17 255.255.255.248
 no shutdown

interface Gi0/0
 ip address 192.168.10.30 255.255.255.252
 no shutdown
 ```

(Insert router configuration screenshots here)

# Static Routing Configuration
```bash
R1
ip route 192.168.10.8 255.255.255.248 192.168.10.26
ip route 192.168.10.16 255.255.255.248 192.168.10.30
R2
ip route 192.168.10.0 255.255.255.248 192.168.10.25
ip route 192.168.10.16 255.255.255.248 192.168.10.25
R3
ip route 192.168.10.0 255.255.255.248 192.168.10.30
ip route 192.168.10.8 255.255.255.248 192.168.10.30
```

(Insert show ip route screenshots here)

# Connectivity Testing
Ping Tests

Test connectivity between different LANs.

Example:

PC5 → PC3
PC1 → PC6
PC2 → Server

(Insert ping screenshots here)

# Traceroute Testing

Traceroute shows the path packets take across routers.

Example:

PC5> tracert 192.168.10.10

Expected path:

PC5 → R3 → R1 → R2 → PC3

This confirms routing tables and next-hop configuration are correct.

(Insert traceroute screenshot here)

# Packet Flow Explanation

When PC5 sends a packet to PC3:

PC5 sends packet to its default gateway (R3).
R3 checks routing table → forwards packet to R1.
R1 checks routing table → forwards packet to R2.
R2 forwards packet to LAN2.
PC3 receives packet.
Reply follows reverse path.

This demonstrates how routers forward packets based on destination network and next-hop IP.

# Key Lessons Learned
Subnetting using VLSM
/29 for LANs, /30 for point-to-point links
Routers route networks, not individual hosts
Default gateway is router interface in LAN
Static routing requires destination network + mask + next-hop
Traceroute shows packet path through routers
Always test router links before end-to-end connectivity
Sequential IP allocation simplifies network design

# Skills Demonstrated

This lab demonstrates practical knowledge of:

Network design and planning
IPv4 subnetting
Router interface configuration
Static routing
Network troubleshooting
Packet flow analysis
Documentation and topology design

# Next Steps

Future labs to build on this project:

Dynamic Routing (RIP / OSPF)
VLANs and Inter-VLAN Routing
ACLs
DHCP Server Configuration
Wireless Networks
Network Automation (Python / Ansible)