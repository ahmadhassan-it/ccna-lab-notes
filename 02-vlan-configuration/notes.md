# VLAN Configuration

## Overview
Three-stage VLAN lab progression: Part 1 covers basic VLAN creation and 
isolation on a single switch. Part 2 extends this to multi-switch trunking 
and inter-VLAN routing via Router on a Stick (ROAS). Part 3 replaces ROAS 
with native Layer 3 switching using SVIs on a multilayer switch, removing 
the router from the inter-VLAN traffic path entirely.

---

## Part 1: Basic VLAN Configuration

### Summary
Configured a single switch with 3 VLANs (Engineering, HR, Sales) to 
demonstrate how VLANs create separate Layer-2 broadcast domains. 
Verified that devices in the same VLAN can communicate, while devices 
in different VLANs cannot — even when connected to the same physical 
switch, since the switch alone does not route between VLANs.

### Topology
- 1 switch (SW1), 6 PCs, no router
- VLAN 10 (ENGINEERING): PC1, PC2 — 192.168.10.0/24
- VLAN 20 (HR): PC3, PC4 — 192.168.20.0/24
- VLAN 30 (SALES): PC5, PC6 — 192.168.30.0/24

### Key Commands
```text
vlan 10
name ENGINEERING
exit

interface fa0/1
switchport mode access
switchport access vlan 10

show vlan brief
```

### What I Practiced
- Creating and naming VLANs
- Assigning access ports to VLANs
- Configuring IP addressing per VLAN
- Verifying VLAN assignment with `show vlan brief`
- Testing same-VLAN and cross-VLAN connectivity with `ping`

### Connectivity Test Results
| Test | Result | Reason |
|---|---|---|
| PC1 → PC2 (same VLAN) | Success | Both in VLAN 10 |
| PC3 → PC4 (same VLAN) | Success | Both in VLAN 20 |
| PC5 → PC6 (same VLAN) | Success | Both in VLAN 30 |
| PC1 → PC3 (different VLAN) | Failed | Switch doesn't route between VLANs |
| PC5 → PC1 (different VLAN) | Failed | Switch doesn't route between VLANs |

### Challenge and How I Solved It
Understanding why same-switch devices in different VLANs couldn't 
communicate. Confirmed this by assigning correct VLANs per port and 
testing with ping — the pattern of successful same-VLAN pings and 
failed cross-VLAN pings made the broadcast-domain separation concrete.

---

## Part 2: VLAN Trunking & Inter-VLAN Routing (Router on a Stick)

### Overview
Designed a multi-VLAN network from scratch, covering 802.1Q trunking between 
switches and inter-VLAN routing via Router on a Stick (ROAS). Built as an 
independent exercise with a custom topology, VLAN numbers, and IP scheme — 
not copied from a reference lab — to test real understanding rather than 
memorized steps.

### Topology
- 2 Layer 2 switches (SW1, SW2) connected via an 802.1Q trunk
- 1 router (R1) connected to SW2 via a second trunk, using subinterfaces for ROAS
- 8 end hosts across 3 VLANs

### VLANs
| VLAN | Name    | Subnet              | Gateway (R1 subinterface) |
|------|---------|----------------------|----------------------------|
| 15   | SALES   | 192.168.50.0/26      | 192.168.50.62              |
| 25   | IT      | 192.168.50.64/26     | 192.168.50.126             |
| 35   | FINANCE | 192.168.50.128/26    | 192.168.50.190             |

### Key Configurations
- Access port VLAN assignment
- 802.1Q trunk configuration with restricted allowed-VLAN lists
- Native VLAN hardening (moved off default VLAN 1 to an unused VLAN)
- Router-on-a-Stick: single physical interface split into per-VLAN 
  subinterfaces with `encapsulation dot1q` and unique IP addressing per subnet

### Verification
Used `show vlan brief`, `show interfaces trunk`, `show ip interface brief` 
at each stage. Ping tests confirmed intra-VLAN, inter-VLAN via R1 on the 
same switch, and full inter-VLAN connectivity across both switches and the router.

### Troubleshooting Encountered
- VLANs allowed on a trunk but not existing in the local VLAN database — 
  traffic silently dropped despite trunk config looking correct
- Native VLAN mismatch between trunk ends (`%CDP-4-NATIVE_VLAN_MISMATCH`)
- Router subinterface required `encapsulation dot1q` before an IP could be assigned
- A trunk link permitting only a subset of required VLANs, blocking traffic 
  to the router for VLANs not explicitly allowed
- STP forwarding-state delay after modifying allowed VLANs on an active trunk

---

## Part 3: Layer 3 Switching & Inter-VLAN Routing via SVI

### Overview
Extended the multi-VLAN network by replacing Router on a Stick with native 
Layer 3 switching, using Switch Virtual Interfaces (SVIs) on a Catalyst 3560 
multilayer switch. Built independently with a custom topology, VLAN scheme, 
and IP addressing — not copied from a reference lab.

### Topology
- SW1: Layer 2 switch, trunk-connected to SW2
- SW2: Layer 3 (multilayer) switch — performs inter-VLAN routing internally via SVIs
- R1: connected to SW2 via a routed point-to-point link (not a trunk) — 
  represents the gateway for traffic leaving the LAN
- 6 end hosts across 3 VLANs

### VLANs & Addressing
| VLAN | Name    | Subnet              | SVI (Gateway) on SW2  |
|------|---------|----------------------|------------------------|
| 15   | SALES   | 172.20.30.0/26       | 172.20.30.62           |
| 25   | IT      | 172.20.30.64/26      | 172.20.30.126          |
| 35   | FINANCE | 172.20.30.128/26     | 172.20.30.190          |

Point-to-point link (SW2 ↔ R1): 172.20.30.192/30

### Key Configurations
- 802.1Q trunk between SW1 and SW2 (restricted allowed-VLAN list, hardened native VLAN)
- `ip routing` enabled globally on SW2 to activate Layer 3 switching capability
- SVIs (`interface vlan X`) configured per VLAN, each requiring `no shutdown` 
  (disabled by default)
- Routed port (`no switchport`) converting SW2's link to R1 from a Layer 2 
  switchport into a true Layer 3 interface
- Static default route on SW2 pointing to R1 for traffic outside the local VLANs

### Why SVI-Based Routing Instead of Router on a Stick
With ROAS, every inter-VLAN packet must travel out to the router and back 
over a single shared trunk — a bottleneck under load. A Layer 3 switch 
performs that routing internally, at hardware speed, removing the router 
entirely from the inter-VLAN path. The router is only needed for traffic 
leaving the LAN altogether.

### Verification
- Confirmed all four conditions required for an SVI to reach up/up: VLAN 
  exists locally, at least one active port (access or trunk) carries it, 
  the VLAN isn't administratively shut down, and the SVI itself has `no shutdown`
- Ping tests confirmed full inter-VLAN connectivity handled entirely by 
  SW2, with zero reliance on R1 for any VLAN-to-VLAN traffic
- Verified the SW2↔R1 routed link reaches up/up only after configuring 
  both ends (a reminder that router interfaces are shut down by default)

### Troubleshooting Encountered
- A switch defaulting to DTP auto-negotiation instead of a manually 
  configured trunk — masked as "working" until checked closely, but with 
  wrong native VLAN and an unrestricted allowed-VLAN list
- `switchport mode trunk` rejected until `switchport trunk encapsulation 
  dot1q` was set first (switch defaulted to "Auto" encapsulation)
- Routed link showing down/down on one side purely because the far-end 
  router interface hadn't been enabled yet
