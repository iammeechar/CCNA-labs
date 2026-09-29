# Overview

This lab demonstrates the design, configuration, and verification of a multi-VLAN network with inter-VLAN routing and traffic control using extended ACLs.

The focus is on:

Building from scratch
Verifying each layer before progressing
Applying security policies using ACL logic
Troubleshooting real misconfigurations

# PHASE 1: VLAN & INTER-VLAN ROUTING
# Topology (Describe + Screenshot)

Devices used:

1 Router (Router-on-a-Stick)
1 Layer 2 Switch
4 PCs (one per VLAN)
1 Server

# VLAN & IP SCHEME
```bash
VLAN	Name	Network	Gateway
10	HR	192.168.10.0/24	192.168.10.1
20	Sales	192.168.20.0/24	192.168.20.1
30	IT	192.168.30.0/24	192.168.30.1
40	Server	192.168.40.0/24	192.168.40.1
```

# CONFIGURATION

🔹 Switch Configuration
Create VLANs
```bash
enable
configure terminal

vlan 10
 name HR
vlan 20
 name Sales
vlan 30
 name IT
vlan 40
 name Server
 ```
Assign Access Ports
```bash
interface f0/1
 switchport mode access
 switchport access vlan 10

interface f0/2
 switchport mode access
 switchport access vlan 20

interface f0/3
 switchport mode access
 switchport access vlan 30

interface f0/4
 switchport mode access
 switchport access vlan 40
 ```

Configure Trunk (Switch → Router)
```bash
interface f0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 ```

🔹 Router Configuration (Router-on-a-Stick)
```bash
enable
configure terminal

interface g0/0
 no shutdown
Subinterfaces
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0

interface g0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
 ```

# End Device Configuration

Example (VLAN 10 PC):

IP: 192.168.10.10
Subnet: 255.255.255.0
Gateway: 192.168.10.1

# BASELINE TESTING (CRITICAL STEP)

Before ACLs, full connectivity must be verified:

ping 192.168.20.10
ping 192.168.30.10
ping 192.168.40.10

# Expected Results
Source	Destination	Result
VLAN 10	VLAN 20	Success
VLAN 20	VLAN 30	Success
VLAN 10	Server	Success

# Issue Encountered
Problem:

Server VLAN unreachable.

Root Cause:

Incorrect physical connection (wrong switch port).

Resolution:
Verified connections using topology + interface checks
Reconnected correct port

# Key Learning

Always verify Layer 1 before troubleshooting configurations.

# PHASE 2: ACL IMPLEMENTATION
# Objective

Restrict traffic between VLANs while maintaining required access.

# Policy Definition
VLAN 20 (Sales) ❌ cannot access VLAN 10 (HR)
VLAN 20 ✅ can access VLAN 30 and Server
All other VLANs unrestricted

# ACL CONFIGURATION
🔹 Create ACL
ip access-list extended BLOCK_SALES_HR
 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 permit ip any any
🔹 Apply ACL
interface g0/0.20
 ip access-group BLOCK_SALES_HR in

# POST-ACL TESTING
 Expected Block
ping 192.168.10.10

From VLAN 20 → should FAIL

# Expected Allow
ping 192.168.30.10
ping 192.168.40.10

# VERIFICATION
show access-lists
Expected:
Match counters increasing on deny rule

# ACL LOGIC BREAKDOWN
🔹 Wildcard Mask
192.168.20.0 0.0.0.255

Matches all IPs in VLAN 20.

🔹 Top-Down Processing
ACL evaluated line-by-line
First match wins
No further checks after match
🔹 Implicit Deny

If no rule matches:

deny ip any any

is automatically applied.

🔹 Why permit ip any any is Required

Prevents unintended blocking of all other traffic.

# COMMON MISTAKES
Mistake	Example	Impact
Rule order wrong	permit before deny	ACL ineffective
Missing permit	No permit any any	Network outage
Wrong interface	Applied on g0/0.10	No effect
Wrong direction	out instead of in	Traffic bypasses ACL
Wrong wildcard	0.0.255.255	Overblocking

# TROUBLESHOOTING COMMANDS
```bash
show vlan brief
show interfaces trunk
show ip interface brief
show access-lists
```

# KEY TAKEAWAYS
ACLs are order-sensitive
Implicit deny enforces security
Placement determines effectiveness
Always test before and after changes
Troubleshooting must be layered

# FINAL REFLECTION

This lab demonstrates progression from:

Basic VLAN configuration

to:

Controlled, policy-based traffic filtering using ACLs🧠 Objective

Expand basic ACL configuration into a multi-rule, policy-driven traffic control system that simulates real-world enterprise access restrictions.

This phase focuses on:

Layered ACL logic (multiple rules)
Role-based access control (HR, Sales, IT, Server)
Directional traffic filtering
Interpreting network behavior through testing
Troubleshooting logical misconfigurations

 Network Roles & Security Model
VLAN	Role	Policy
10	HR	Protected (restricted inbound access)
20	Sales	Restricted (limited access)
30	IT	Full access (override role)
40	Server	Shared resource (no outbound initiation)

 Security Policies Implemented
🔹 Sales Restrictions
❌ Cannot access HR VLAN
❌ Cannot ping Server VLAN
✅ Can access IT VLAN
🔹 IT Override Rule
✅ Full access to all VLANs
Implemented as highest-priority ACL rule
🔹 Server Protection
❌ Cannot initiate traffic to any VLAN
✅ Can respond to incoming requests (limited by ACL behavior)

# ACL CONFIGURATION
🔹 Advanced VLAN Policy ACL
ip access-list extended ADVANCED_POLICY
 permit ip 192.168.30.0 0.0.0.255 any
 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny icmp 192.168.20.0 0.0.0.255 192.168.40.0 0.0.0.255
 permit ip any any
Applied to:
interface g0/0.20
 ip access-group ADVANCED_POLICY in
🔹 Server Protection ACL
ip access-list extended SERVER_PROTECTION
 deny ip 192.168.40.0 0.0.0.255 any
 permit ip any any
 Applied to:
interface g0/0.40
 ip access-group SERVER_PROTECTION in
 
# TESTING & RESULTS
Server-Initiated Traffic
Source	Destination	Result
Server	VLAN 10	❌ Destination unreachable
Server	VLAN 30	❌ Destination unreachable

 VLAN 10 / VLAN 30 → Server
Source	Destination	Result
VLAN 10	Server	⏳ Request timed out
VLAN 30	Server	⏳ Request timed out

 VLAN 20 (Sales) → Server
Source	Destination	Result
VLAN 20	Server	❌ Destination unreachable

# TRAFFIC BEHAVIOR ANALYSIS

This phase introduced a critical distinction in ICMP responses:

🔹 1. Destination Host Unreachable

Occurs when:

Traffic is blocked immediately by the router
ACL denies packet at entry point
Example:
Server → Router → ❌ Blocked

👉 Result:

Destination host unreachable
🔹 2. Request Timed Out

Occurs when:

Packet reaches destination
Return traffic is blocked
Example:
PC → Server (allowed)
Server → Router (blocked)

👉 Result:

Request timed out

# Key Insight

The type of ICMP response reveals where traffic is being dropped.

This was used to validate:

# ACL placement
Direction of filtering
Forward vs return path behavior

# TROUBLESHOOTING SCENARIOS
❌ Issue 1: Incorrect Wildcard Mask
0.0.0.25 ❌
Impact:
Partial traffic matching
Inconsistent ACL behavior
Fix:
0.0.0.255 ✅

❌ Issue 2: Wrong ACL Direction
ip access-group SERVER_PROTECTION out ❌
Impact:
Server could still initiate traffic
Fix:
ip access-group SERVER_PROTECTION in ✅

❌ Issue 3: Incorrect Rule Order
permit ip any any
deny ip ...
Impact:
Deny rule never executed
Fix:
deny ip ...
permit ip any any

❌ Issue 4: Typographical Error in Network
192.68.30.0 ❌
192.168.30.0 ✅
Impact:
IT override rule ineffective

# VERIFICATION COMMANDS
show access-lists
show running-config
Key validation:
ACL match counters increasing
Correct interface application
Correct rule order

# KEY LEARNINGS
ACLs are logic-driven, not just configuration-driven
Rule order directly determines behavior
Direction (in vs out) defines traffic control point
ICMP responses provide insight into traffic flow failures
Small mistakes (typos, masks) can silently break policies