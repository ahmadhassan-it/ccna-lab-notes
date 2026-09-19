# Basic VLAN Configuration

## Summary
Configured a single switch with 3 VLANs (Engineering, HR, Sales) to 
demonstrate how VLANs create separate Layer-2 broadcast domains. 
Verified that devices in the same VLAN can communicate, while devices 
in different VLANs cannot — even when connected to the same physical 
switch, since the switch alone does not route between VLANs.

## Topology
- 1 switch (SW1), 6 PCs, no router
- VLAN 10 (ENGINEERING): PC1, PC2 — 192.168.10.0/24
- VLAN 20 (HR): PC3, PC4 — 192.168.20.0/24
- VLAN 30 (SALES): PC5, PC6 — 192.168.30.0/24

## Key Commands
```text
vlan 10
name ENGINEERING
exit

interface fa0/1
switchport mode access
switchport access vlan 10

show vlan brief
```

## What I Practiced
- Creating and naming VLANs
- Assigning access ports to VLANs
- Configuring IP addressing per VLAN
- Verifying VLAN assignment with `show vlan brief`
- Testing same-VLAN and cross-VLAN connectivity with `ping`

## Connectivity Test Results
| Test | Result | Reason |
|---|---|---|
| PC1 → PC2 (same VLAN) | Success | Both in VLAN 10 |
| PC3 → PC4 (same VLAN) | Success | Both in VLAN 20 |
| PC5 → PC6 (same VLAN) | Success | Both in VLAN 30 |
| PC1 → PC3 (different VLAN) | Failed | Switch doesn't route between VLANs |
| PC5 → PC1 (different VLAN) | Failed | Switch doesn't route between VLANs |

## Challenge and How I Solved It
Understanding why same-switch devices in different VLANs couldn't 
communicate. Confirmed this by assigning correct VLANs per port and 
testing with ping — the pattern of successful same-VLAN pings and 
failed cross-VLAN pings made the broadcast-domain separation concrete.
