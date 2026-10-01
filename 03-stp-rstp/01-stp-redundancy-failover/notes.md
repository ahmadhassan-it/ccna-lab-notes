# STP Redundancy & Failover Lab

## Objective

In this lab, I built a three-switch triangle topology in Cisco Packet Tracer to understand how **Spanning Tree Protocol (STP)** prevents Layer 2 loops while keeping a redundant path available for failover.

I specifically practiced:

* Root Bridge election
* Root Port selection
* Designated Port selection
* Alternate/Blocking Port
* Changing the Root Bridge
* Link failure and STP failover
* Link restoration
* STP verification

---

## 1. Topology

I used three Cisco 2960 switches and connected them in a triangle.

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

No PCs were used because the purpose of this lab was to observe STP behavior between switches.

---

## 2. Initial STP Verification

After building the topology, I checked STP on the switches using:

```text
do show spanning-tree vlan 1
```

Initially, **SW2 was elected as the Root Bridge**.

The Root Bridge was selected using the lowest Bridge ID.

```text
Bridge ID = Bridge Priority + MAC Address
```

The switches had the default STP priority of 32768. Cisco displayed 32769 for VLAN 1 because of the System ID Extension.

---

## 3. Initial Port Roles

After the initial STP election, I observed the following:

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

The important observation was that the physical triangle remained connected, but STP logically blocked one redundant path.

This prevents a Layer 2 loop.

---

## 4. Why Was SW1 Fa0/2 Blocking?

The SW1–SW3 link was the redundant path.

```text
SW1 Fa0/2 ───────── Fa0/1 SW3
```

Both SW1 and SW3 had a Root Path Cost of 19 toward SW2.

Because the Root Path Costs were equal, STP used the Bridge ID as a tie-breaker.

SW3 had the lower Bridge ID:

```text
SW3 = 00D0.BA71.05CD
SW1 = 00E0.8F00.EA13
```

Therefore, SW3 won that segment:

```text
SW3 Fa0/1 = Designated
SW1 Fa0/2 = Alternate / Blocking
```

This helped me understand that a **non-root switch can have a Designated Port**. A non-root switch has exactly one Root Port, but it can also have Designated Ports on other segments.

---

## 5. Changing the Root Bridge

Next, I intentionally changed the Root Bridge.

On SW1, I configured a lower STP priority:

```text
enable
configure terminal
spanning-tree vlan 1 priority 24576
end
```

Then I verified the result:

```text
do show spanning-tree vlan 1
```

SW1 became the new Root Bridge.

The output confirmed:

```text
This bridge is the root
```

### Why did SW1 become Root?

SW1 now had a priority of 24576 instead of the default 32768.

Since lower Bridge ID wins, SW1 became the Root Bridge.

---

## 6. Port Roles After Changing the Root

After SW1 became the Root Bridge, the topology changed.

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

This demonstrated that changing the Root Bridge can change the roles of ports throughout the topology.

---

## 7. STP Failover Test

I then tested whether the redundant link could provide connectivity if the active path failed.

I shut down SW1 Fa0/2:

```text
enable
configure terminal
interface fa0/2
shutdown
end
```

This caused the SW1–SW3 path to fail.

Before the failure:

```text
SW3 Fa0/1 = Root / Forwarding
SW3 Fa0/2 = Alternate / Blocking
```

After the failure, SW3 needed another path to reach the Root Bridge.

SW3 Fa0/2 changed to:

```text
Root / Forwarding
```

The path became:

```text
SW3 → SW2
```

The previously blocked redundant path was now being used.

---

## 8. Restoring the Failed Link

I restored SW1 Fa0/2:

```text
enable
configure terminal
interface fa0/2
no shutdown
end
```

After the link was restored, STP recalculated the topology and returned the ports to their appropriate roles.

The redundant path was again placed into a non-forwarding state to prevent a loop.

---

## 9. What I Observed

The most important thing I learned from this lab was that STP does **not** remove redundant physical connections.

Instead, it keeps the physical redundancy but logically prevents selected ports from forwarding.

```text
Physical topology:

SW1 ───── SW2
 \       /
  \     /
    SW3
```

STP creates a loop-free logical topology by blocking a redundant path.

If the active path fails, STP can use the redundant path.

---

## 10. Verification Commands

The main command I used to verify STP was:

```text
do show spanning-tree vlan 1
```

This allowed me to check:

* Root Bridge
* Root ID
* Bridge ID
* Root Port
* Designated Port
* Alternate Port
* Port state
* Root Path Cost

I also used:

```text
do show spanning-tree summary
```

to view an STP summary.

---

## 11. Key Lessons From the Lab

### Root Bridge

The switch with the lowest Bridge ID becomes the Root Bridge.

### Root Port

A non-root switch has exactly **one Root Port**, which provides its best path toward the Root Bridge.

### Designated Port

Each network segment has a Designated Port responsible for the forwarding path toward the Root.

A non-root switch **can have a Designated Port**.

### Alternate Port

An Alternate Port provides a redundant path toward the Root but remains non-forwarding while another better path is available.

### STP Failover

When the active path fails, STP can move the redundant path into forwarding so connectivity can continue.

---

## 12. Commands Used

### Change STP Priority

```text
spanning-tree vlan 1 priority 24576
```

### Shut Down a Link

```text
interface fa0/2
shutdown
```

### Restore a Link

```text
interface fa0/2
no shutdown
```

### Verify STP

```text
show spanning-tree vlan 1
show spanning-tree summary
```

---

## Result

**Lab completed successfully.**

I built a redundant three-switch topology, observed STP elect the Root Bridge and assign port roles, changed the Root Bridge, tested link failure, observed failover to the redundant path, and restored the original topology.

**Main takeaway:**

> STP prevents Layer 2 loops by logically blocking redundant paths while keeping those paths available for failover.
