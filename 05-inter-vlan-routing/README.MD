# CCNA Lab 05 — Inter-VLAN Routing

## Objective

Configure inter-VLAN routing using **Router-on-a-Stick** so that devices in different VLANs can communicate through a router.

## Topology

```text
PC1 ── SW1 ── R1
       │
       └── PC2

SW1 G0/1 ↔ R1 G0/0
```

## VLANs and IP Addressing

| Device |       VLAN | IP Address       | Default Gateway |
| ------ | ---------: | ---------------- | --------------- |
| PC1    | 10 — SALES | 192.168.10.10/24 | 192.168.10.1    |
| PC2    |    20 — IT | 192.168.20.10/24 | 192.168.20.1    |

## Switch Configuration

Created two VLANs:

```text
VLAN 10 — SALES
VLAN 20 — IT
```

PC1 was assigned to VLAN 10:

```text
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
```

PC2 was assigned to VLAN 20:

```text
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
```

The switch-to-router link was configured as an 802.1Q trunk:

```text
interface GigabitEthernet0/1
 switchport mode trunk
```

## Router Configuration

The physical router interface was enabled:

```text
interface GigabitEthernet0/0
 no shutdown
```

### VLAN 10 Subinterface

```text
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
```

### VLAN 20 Subinterface

```text
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

## Verification

Verified the router interfaces and routing table using:

```text
show ip interface brief
show ip route
```

The router showed connected networks for:

```text
192.168.10.0/24
192.168.20.0/24
```

## Connectivity Test

From PC1:

```text
ping 192.168.20.10
```

### Result

**Successful.**

PC1 in VLAN 10 successfully communicated with PC2 in VLAN 20 through the router.

## Key Concepts Practiced

* VLAN configuration
* Access ports
* 802.1Q trunking
* Router-on-a-Stick
* Router subinterfaces
* Inter-VLAN routing
* Default gateways
* IPv4 addressing
* Connectivity verification
* Basic troubleshooting

## Tools

* Cisco Packet Tracer
* Cisco IOS CLI

## Lab Status

**Completed and tested successfully.**
