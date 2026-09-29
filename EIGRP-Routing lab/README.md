
# EIGRP Routing Lab

## Overview

This lab demonstrates the configuration, verification, and troubleshooting of **Enhanced Interior Gateway Routing Protocol (EIGRP)** using Cisco CSR1000v routers in an **EVE-NG** virtual network environment.

The lab begins with a simple three-router topology and progressively develops into a more realistic routing scenario where multiple paths can be compared and EIGRP metric manipulation can be observed.

The objective is not only to configure EIGRP, but to understand how routing information is exchanged, how EIGRP selects paths, how metrics influence route selection, and how the network behaves when topology changes occur.

---

## Lab Objectives

* Build a three-router EIGRP topology in EVE-NG.
* Configure IPv4 addressing and Layer 3 connectivity.
* Establish EIGRP neighbor adjacencies.
* Advertise connected networks through EIGRP.
* Verify EIGRP-learned routes.
* Examine the EIGRP topology table.
* Understand successors and feasible successors.
* Investigate EIGRP metric calculation.
* Manipulate bandwidth and delay to influence path selection.
* Test routing convergence following a link failure.
* Capture troubleshooting procedures and verification results.
* Produce a GitHub-ready record of the complete lab evolution.

---

## Environment

| Component         | Details                                   |
| ----------------- | ----------------------------------------- |
| Network simulator | EVE-NG                                    |
| Router platform   | Cisco CSR1000v                            |
| IOS XE image      | `csr1000vng-universalk9.17.03.08a-serial` |
| Routers           | R1, R2, R3                                |
| Routing protocol  | EIGRP for IPv4                            |
| EIGRP AS          | 100                                       |
| Addressing        | IPv4                                      |
| Topology          | Initial 3-router linear topology          |

---

# Phase 1 — Initial Topology

The initial topology consists of three routers connected in a linear arrangement:

```text
R1 -------- R2 -------- R3
```

### Network design

```text
R1 Gi1 -------- Gi1 R2 Gi2 -------- Gi1 R3
10.0.12.1      10.0.12.2 10.0.23.1      10.0.23.2
    /                                      \
   /                                        \
192.168.1.0/24                         192.168.3.0/24
```

### IP Addressing

| Router | Interface | Address        | Purpose       |
| ------ | --------- | -------------- | ------------- |
| R1     | Gi1       | 10.0.12.1/30   | R1-R2 transit |
| R1     | Gi2       | 192.168.1.1/24 | R1 LAN        |
| R2     | Gi1       | 10.0.12.2/30   | R1-R2 transit |
| R2     | Gi2       | 10.0.23.1/30   | R2-R3 transit |
| R3     | Gi1       | 10.0.23.2/30   | R2-R3 transit |
| R3     | Gi2       | 192.168.3.1/24 | R3 LAN        |

Unused interfaces were left unassigned and administratively down.

---

# Phase 2 — Interface Configuration

Before configuring EIGRP, Layer 3 connectivity was established between the routers.

Example R1 configuration:

```text
configure terminal

interface GigabitEthernet1
 ip address 10.0.12.1 255.255.255.252
 no shutdown

interface GigabitEthernet2
 ip address 192.168.1.1 255.255.255.0
 no shutdown

end
```

Equivalent addressing was configured on R2 and R3.

### Verification

The interfaces were checked using:

```text
show ip interface brief
```

The transit interfaces were verified as:

```text
up/up
```

Direct router-to-router connectivity was then tested with ICMP.

This established the underlying IP connectivity before introducing dynamic routing.

---

# Phase 3 — Pre-EIGRP Baseline

Before EIGRP was configured, the routing tables were captured.

For example, R1 contained only its directly connected networks:

```text
C    10.0.12.0/30 is directly connected, GigabitEthernet1
L    10.0.12.1/32 is directly connected, GigabitEthernet1
C    192.168.1.0/24 is directly connected, GigabitEthernet2
L    192.168.1.1/32 is directly connected, GigabitEthernet2
```

R1 had no route to:

```text
192.168.3.0/24
```

Similarly, R3 had no route to:

```text
192.168.1.0/24
```

This provided a useful **before-EIGRP baseline** for comparison.

---

# Phase 4 — EIGRP Configuration

EIGRP was configured using Autonomous System 100.

## R1

```text
configure terminal

router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 192.168.1.0 0.0.0.255
 no auto-summary

end
```

## R2

```text
configure terminal

router eigrp 100
 network 10.0.12.0 0.0.0.3
 network 10.0.23.0 0.0.0.3
 no auto-summary

end
```

## R3

```text
configure terminal

router eigrp 100
 network 10.0.23.0 0.0.0.3
 network 192.168.3.0 0.0.0.255
 no auto-summary

end
```

---

# Phase 5 — EIGRP Neighbor Verification

EIGRP neighbor relationships were verified using:

```text
show ip eigrp neighbors
```

### R1

R1 established an adjacency with R2:

```text
10.0.12.2
GigabitEthernet1
```

### R2

R2 established adjacencies with both neighboring routers:

```text
10.0.12.1  GigabitEthernet1
10.0.23.2  GigabitEthernet2
```

### R3

R3 established an adjacency with R2:

```text
10.0.23.1
GigabitEthernet1
```

All observed neighbor queues were:

```text
Q = 0
```

indicating that there were no pending EIGRP packets waiting to be processed.

---

# Phase 6 — EIGRP Route Verification

EIGRP-learned routes were identified with:

```text
show ip route
```

Routes marked with:

```text
D
```

were learned through EIGRP.

### R1

R1 learned:

```text
D 10.0.23.0/30 [90/3072] via 10.0.12.2, GigabitEthernet1
D 192.168.3.0/24 [90/3328] via 10.0.12.2, GigabitEthernet1
```

### R2

R2 learned:

```text
D 192.168.1.0/24 [90/3072] via 10.0.12.1, GigabitEthernet1
D 192.168.3.0/24 [90/3072] via 10.0.23.2, GigabitEthernet2
```

### R3

R3 learned:

```text
D 10.0.12.0/30 [90/3072] via 10.0.23.1, GigabitEthernet1
D 192.168.1.0/24 [90/3328] via 10.0.23.1, GigabitEthernet1
```

The EIGRP administrative distance observed in the routing table was:

```text
90
```

---

# Phase 7 — EIGRP Topology Table

The EIGRP topology database was examined using:

```text
show ip eigrp topology
```

Example from R1:

```text
P 192.168.3.0/24, 1 successors, FD is 3328
        via 10.0.12.2 (3328/3072), GigabitEthernet1
```

The output demonstrates several important EIGRP concepts:

* `P` — Passive state
* `1 successors` — one currently selected best path
* `FD` — Feasible Distance
* First value in parentheses — metric through the listed path
* Second value — reported distance from the neighboring router

At this stage, the topology contained only one available path between the two LANs, so there was no meaningful alternate-path comparison yet.

---

# Phase 8 — End-to-End Validation

After EIGRP convergence, end-to-end connectivity was tested.

## R1 → R3 LAN

```text
R1#ping 192.168.3.1

Sending 5, 100-byte ICMP Echos to 192.168.3.1, timeout is 2 seconds:
!!!!!

Success rate is 100 percent (5/5)
```

## R3 → R1 LAN

```text
R3#ping 192.168.1.1

Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!

Success rate is 100 percent (5/5)
```

Both directions successfully reached the remote LAN interfaces.

---

## Traceroute Verification

From R1:

```text
R1#traceroute 192.168.3.1
```

Observed path:

```text
1  10.0.12.2
2  10.0.23.2
```

This confirms the forwarding path:

```text
R1 → R2 → R3
```

The traceroute therefore matched the EIGRP route installed on R1.

---

# Current Lab Status

At this stage the initial EIGRP implementation is operational.

| Component                     | Status     |
| ----------------------------- | ---------- |
| EVE-NG topology               | Complete   |
| CSR1000v routers              | Complete   |
| IPv4 addressing               | Complete   |
| Interface connectivity        | Verified   |
| Pre-EIGRP baseline            | Captured   |
| EIGRP AS 100                  | Configured |
| Neighbor adjacencies          | Verified   |
| EIGRP routes                  | Verified   |
| EIGRP topology database       | Verified   |
| End-to-end connectivity       | Verified   |
| Traceroute                    | Verified   |
| Alternate path                | Pending    |
| Metric manipulation           | Pending    |
| Feasible successor analysis   | Pending    |
| Link failure/convergence test | Pending    |

---

# Next Phase — Alternate Path and Metric Analysis

The next topology modification will introduce a direct R1-R3 link:

```text
             R2
            /  \
           /    \
          R1----R3
```

This will create two possible paths between R1 and R3:

```text
Path A:
R1 → R2 → R3

Path B:
R1 → R3
```

The purpose is to observe how EIGRP evaluates multiple paths instead of simply discussing the theory.

The following will then be investigated:

1. EIGRP metric calculation.
2. Successor selection.
3. Feasible successor behavior.
4. Feasibility condition.
5. Bandwidth and delay.
6. Changes in the routing table.
7. Changes in the EIGRP topology table.
8. Link failure and convergence.
9. Restoration of the failed path.

The `bandwidth` command will be treated correctly as a **routing-metric input**, not as a command that physically changes the capacity of the virtual link.

---

# Troubleshooting / Lab Notes

This section will be expanded throughout the project.

The lab is intentionally being documented incrementally so that configuration mistakes, unexpected behavior, troubleshooting procedures, and design decisions are preserved rather than documenting only the final working configuration.

---

# Learning Outcomes

By completing this lab, the following concepts will be demonstrated:

* IPv4 Layer 3 connectivity
* Dynamic routing
* EIGRP neighbor formation
* EIGRP network advertisement
* EIGRP administrative distance
* Feasible Distance
* Reported Distance
* Successors
* Feasible successors
* EIGRP composite metrics
* Bandwidth and delay
* Routing-table analysis
* Topology-table analysis
* Path selection
* Routing convergence
* Network troubleshooting
* Cisco IOS XE routing configuration
* EVE-NG network simulation

---

## Lab Evidence

Screenshots and command outputs will be added progressively under:

```text
screenshots/
verification/
```

Configuration snapshots will be stored under:

```text
configs/
```

Topology diagrams and related network design material will be stored under:

```text
topology/
```

Additional troubleshooting notes and lessons learned will be stored under:

```text
notes/
```

---

## Status

**Current milestone:** Initial three-router EIGRP implementation and end-to-end validation complete.

**Next milestone:** Add an alternate R1-R3 path and investigate EIGRP metric-based path selection.
