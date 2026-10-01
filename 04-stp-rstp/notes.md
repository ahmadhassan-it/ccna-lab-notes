# Spanning Tree Protocol (STP & RSTP)

## Topic Overview

In this topic, I learned and practiced **Spanning Tree Protocol (STP)** and **Rapid Spanning Tree Protocol (RSTP)** using Cisco Packet Tracer.

The main purpose of STP/RSTP is to prevent **Layer 2 switching loops** while keeping redundant physical paths available for failover.

I completed two practical labs:

* **Part 1 — STP Redundancy & Failover**
* **Part 2 — RSTP Rapid Failover**

---

# Part 1 — STP Redundancy & Failover

## Objective

I built a three-switch triangle topology to observe how STP prevents a Layer 2 loop by logically blocking one redundant path.

I also changed the Root Bridge and tested what happened when an active link failed.

---

## Topology

I used three Cisco 2960 switches.

```text
                 SW2
              ROOT BRIDGE
              /         \
         Fa0/1           Fa0/2
           |               |
        Fa0/1            Fa0/2
          SW1-------------SW3
              Fa0/2   Fa0/1
```

### Connections

```text
SW1 Fa0/1 ↔ SW2 Fa0/1
SW1 Fa0/2 ↔ SW3 Fa0/1
SW2 Fa0/2 ↔ SW3 Fa0/2
```

No PCs were required because this lab focused on STP behavior between switches.

---

## Initial STP Verification

I checked STP using:

```text
show spanning-tree vlan 1
```

Initially, **SW2 became the Root Bridge**.

The Root Bridge is selected using the lowest Bridge ID.

```text
Bridge ID = Bridge Priority + MAC Address
```

The default configured priority was 32768. The output displayed 32769 for VLAN 1 because of the System ID Extension.

---

## Initial Port Roles

### SW2 — Root Bridge

```text
Fa0/1 → Designated / Forwarding
Fa0/2 → Designated / Forwarding
```

### SW1

```text
Fa0/1 → Root / Forwarding
Fa0/2 → Alternate / Blocking
```

### SW3

```text
Fa0/2 → Root / Forwarding
Fa0/1 → Designated / Forwarding
```

STP kept the physical triangle connected but logically prevented one redundant path from forwarding.

---

## Why Was SW1 Fa0/2 Blocking?

The SW1–SW3 link was the redundant path:

```text
SW1 Fa0/2 ───────── Fa0/1 SW3
```

Both switches had a Root Path Cost of 19 toward SW2.

Because the Root Path Cost was equal, Bridge ID was used as the tie-breaker.

```text
SW3 = 00D0.BA71.05CD
SW1 = 00E0.8F00.EA13
```

SW3 had the lower Bridge ID.

Therefore:

```text
SW3 Fa0/1 = Designated
SW1 Fa0/2 = Alternate / Blocking
```

This helped me understand that **a non-root switch can have a Designated Port**.

A non-root switch has exactly one Root Port, but it can also have Designated Ports on other segments.

---

## Changing the Root Bridge

I changed the Root Bridge by lowering SW1's STP priority.

On SW1:

```text
enable
configure terminal
spanning-tree vlan 1 priority 24576
end
```

I then verified the result:

```text
show spanning-tree vlan 1
```

SW1 became the Root Bridge.

The output confirmed:

```text
This bridge is the root
```

### Why did SW1 become Root?

SW1's priority was changed from the default 32768 to 24576.

Since the lower Bridge ID wins, SW1 became the Root Bridge.

---

## Port Roles After Changing the Root

### SW1 — Root Bridge

```text
Fa0/1 → Designated / Forwarding
Fa0/2 → Designated / Forwarding
```

### SW3

```text
Fa0/1 → Root / Forwarding
Fa0/2 → Alternate / Blocking
```

Changing the Root Bridge changed the port roles throughout the topology.

---

## STP Failover Test

I tested the redundant path by shutting down SW1 Fa0/2:

```text
enable
configure terminal
interface fa0/2
shutdown
end
```

Before the failure, SW3 had:

```text
Fa0/1 → Root / Forwarding
Fa0/2 → Alternate / Blocking
```

After the SW1–SW3 link failed, SW3 lost its current Root Port.

The previously blocked path became active:

```text
SW3 Fa0/2 → Root / Forwarding
```

The new path was:

```text
SW3 → SW2
```

This demonstrated STP failover.

---

## Restoring the Link

I restored SW1 Fa0/2:

```text
enable
configure terminal
interface fa0/2
no shutdown
end
```

After restoration, STP returned the topology to its normal state.

The redundant path was again placed into a non-forwarding state.

---

## Part 1 — Main Lessons

I learned that:

* STP prevents Layer 2 loops.
* The lowest Bridge ID becomes the Root Bridge.
* A non-root switch has exactly one Root Port.
* A non-root switch can also have Designated Ports.
* Alternate paths can be placed into a non-forwarding state.
* A blocked redundant path can become active when the primary path fails.
* Changing the Root Bridge can change port roles.

---

# Part 2 — RSTP Rapid Failover

## Objective

In the second lab, I used the same three-switch triangle topology to practice **Rapid Spanning Tree Protocol (RSTP)** using Cisco **Rapid PVST+**.

The main goal was to observe how the redundant path is used when the active path fails.

---

## Topology

```text
                 SW2
              ROOT BRIDGE
              /         \
         Fa0/1           Fa0/2
           |               |
        Fa0/1            Fa0/2
          SW1-------------SW3
              Fa0/2   Fa0/1
```

### Connections

```text
SW1 Fa0/1 ↔ SW2 Fa0/1
SW1 Fa0/2 ↔ SW3 Fa0/1
SW2 Fa0/2 ↔ SW3 Fa0/2
```

---

## Enabling Rapid PVST+

Initially, the switches were running PVST.

I enabled Rapid PVST+ on all three switches.

On each switch:

```text
enable
configure terminal
spanning-tree mode rapid-pvst
end
```

I verified the mode using:

```text
show spanning-tree summary
```

The output confirmed:

```text
Switch is in rapid-pvst mode
```

I also verified the VLAN:

```text
show spanning-tree vlan 1
```

The output confirmed:

```text
Spanning tree enabled protocol rstp
```

---

## RSTP Root Bridge

After enabling Rapid PVST+, SW2 was the Root Bridge.

```text
SW2
MAC: 0001.43A5.BDB6
```

The Root Bridge output showed:

```text
This bridge is the root
```

---

## RSTP Port Roles

### SW2 — Root Bridge

```text
Fa0/1 → Designated / Forwarding
Fa0/2 → Designated / Forwarding
```

### SW1

```text
Fa0/1 → Root / Forwarding
Fa0/2 → Alternate / Discarding
```

### SW3

```text
Fa0/2 → Root / Forwarding
Fa0/1 → Designated / Forwarding
```

Cisco's output displayed `BLK` for the non-forwarding port, but RSTP's actual state model is:

```text
Discarding
Learning
Forwarding
```

---

## Why Is SW3 Fa0/1 Designated?

This was an important point I investigated during the lab.

The physical connections were:

```text
SW2 Fa0/2 ↔ SW3 Fa0/2
SW1 Fa0/2 ↔ SW3 Fa0/1
```

SW3 Fa0/2 was already the Root Port because it provided the best path to the Root Bridge.

Therefore, SW3 Fa0/1 could not also be a Root Port.

On the SW1–SW3 segment:

```text
SW1 Fa0/2 ───────── Fa0/1 SW3
```

Both switches had the same Root Path Cost:

```text
SW1 → SW2 = 19
SW3 → SW2 = 19
```

The Bridge ID was then used as the tie-breaker.

```text
SW3 = 0004.9A8C.8D50
SW1 = 0090.0C22.279E
```

SW3 had the lower Bridge ID.

Therefore:

```text
SW3 Fa0/1 = Designated / Forwarding
SW1 Fa0/2 = Alternate / Non-forwarding
```

This reinforced the rule:

> A non-root switch has exactly one Root Port, but it can have one or more Designated Ports.

---

## RSTP Failover Test

I tested RSTP failover by shutting down SW2 Fa0/2.

On SW2:

```text
enable
configure terminal
interface fa0/2
shutdown
end
```

Before the failure:

```text
SW3 Fa0/2 = Root / Forwarding
SW3 Fa0/1 = Designated / Forwarding
```

After SW2 Fa0/2 failed, SW3 lost its direct path to the Root Bridge.

SW3 then used the path through SW1.

The output changed to:

```text
Fa0/1    Root FWD    19
```

The new path became:

```text
SW3 → SW1 → SW2
```

---

## Root Path Cost Change

Before the failure:

```text
SW3 → SW2
Cost = 19
```

After the failure:

```text
SW3 → SW1 → SW2

SW3 → SW1 = 19
SW1 → SW2 = 19

Total = 38
```

The Root Path Cost therefore changed:

```text
19 → 38
```

This was direct evidence that SW3 had moved to the redundant path.

---

## Restoring the RSTP Link

I restored SW2 Fa0/2:

```text
enable
configure terminal
interface fa0/2
no shutdown
end
```

The direct path became available again.

SW3 returned to:

```text
Fa0/2 → Root / Forwarding
Fa0/1 → Designated / Forwarding
```

The Root Path Cost returned to:

```text
38 → 19
```

SW2 remained the Root Bridge.

---

# STP vs RSTP — What I Practiced

| Feature         | STP                                       | RSTP                                          |
| --------------- | ----------------------------------------- | --------------------------------------------- |
| Protocol        | STP                                       | RSTP                                          |
| Cisco mode used | PVST                                      | Rapid PVST+                                   |
| Main purpose    | Prevent Layer 2 loops                     | Prevent Layer 2 loops with faster convergence |
| Root Port       | Yes                                       | Yes                                           |
| Designated Port | Yes                                       | Yes                                           |
| Alternate Port  | Conceptually redundant/non-forwarding     | Explicit RSTP role                            |
| States          | Blocking, Listening, Learning, Forwarding | Discarding, Learning, Forwarding              |
| Lab tested      | Redundancy & failover                     | Rapid failover                                |

---

# Verification Commands

The main commands I used throughout these labs were:

```text
show spanning-tree
show spanning-tree vlan 1
show spanning-tree summary
```

To change the STP priority:

```text
spanning-tree vlan 1 priority 24576
```

To enable Rapid PVST+:

```text
spanning-tree mode rapid-pvst
```

To shut down a link:

```text
interface fa0/2
shutdown
```

To restore a link:

```text
interface fa0/2
no shutdown
```

---

# Key Concepts I Learned

## Root Bridge

The switch with the lowest Bridge ID becomes the Root Bridge.

```text
Bridge ID = Priority + MAC Address
```

## Root Port

A non-root switch has exactly **one Root Port**.

It is the port providing the best path toward the Root Bridge.

## Designated Port

A segment has a Designated Port that provides the forwarding path toward the Root.

The Root Bridge's active ports are Designated Ports, but **non-root switches can also have Designated Ports**.

## Alternate Port

An Alternate Port provides a backup path toward the Root.

It remains non-forwarding while the better path is available.

## Path Cost

STP/RSTP uses Root Path Cost to select the best path toward the Root Bridge.

Lower total cost is preferred.

## Failover

A redundant physical path can become active when the current path fails.

---

# Final Results

## Part 1 — STP

* [x] Built three-switch redundant topology
* [x] Verified Root Bridge
* [x] Identified Root, Designated and Alternate ports
* [x] Changed Root Bridge
* [x] Tested link failure
* [x] Observed STP failover
* [x] Restored the failed link

## Part 2 — RSTP

* [x] Enabled Rapid PVST+
* [x] Verified RSTP operation
* [x] Identified Root, Designated and Alternate ports
* [x] Tested link failure
* [x] Observed SW3 move to the redundant path
* [x] Observed Root Path Cost change from 19 to 38
* [x] Restored the direct path
* [x] Verified SW2 remained the Root Bridge

---

# Final Takeaway

This topic helped me understand STP and RSTP through actual Packet Tracer experiments rather than only theory.

The most important concept I learned is:

> **STP/RSTP does not remove redundant physical links. It controls which ports are allowed to forward so that the network remains loop-free, while redundant paths remain available for failover.**
