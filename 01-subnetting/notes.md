# CCNA Subnetting

## Summary

Subnetting is the process of dividing an IP network into smaller networks called subnets. I practiced calculating subnet masks, block sizes, network addresses, broadcast addresses, usable host ranges, and host counts. Subnetting is important in real-world networks because it helps organize IP addresses, reduce unnecessary broadcast traffic, and allocate addresses efficiently.

## Key Concepts

* Prefix length → subnet mask
* Block size = `256 - mask value` in the changing octet
* Network address = first address of the subnet
* Broadcast address = last address of the subnet
* First usable address = network + 1
* Last usable address = broadcast - 1
* Usable hosts = `2^(host bits) - 2`
* VLSM allows different subnet sizes based on the number of required hosts.
* In VLSM, allocate the largest subnet first.

## Common Subnet Masks

| Prefix | Subnet Mask     | Block Size | Usable Hosts |
| ------ | --------------- | ---------: | -----------: |
| /25    | 255.255.255.128 |        128 |          126 |
| /26    | 255.255.255.192 |         64 |           62 |
| /27    | 255.255.255.224 |         32 |           30 |
| /28    | 255.255.255.240 |         16 |           14 |
| /29    | 255.255.255.248 |          8 |            6 |
| /30    | 255.255.255.252 |          4 |            2 |

## Key Calculations

```text
Block Size = 256 - subnet mask value

Host Bits = 32 - prefix length

Total Addresses = 2^(host bits)

Usable Hosts = 2^(host bits) - 2

Broadcast Address = next subnet boundary - 1
```

## What I Practiced

* Converted CIDR prefixes into subnet masks.
* Calculated block sizes and subnet boundaries.
* Found network and broadcast addresses.
* Calculated first and last usable addresses.
* Calculated the number of usable hosts.
* Solved subnetting questions with different prefix lengths.
* Practiced subnetting in different octets, including `/17`.
* Solved host-requirement questions by selecting the appropriate prefix.
* Practiced VLSM by allocating different subnet sizes according to host requirements.
* Completed a VLSM lab-style problem using `192.168.10.0/24`, allocating `/26`, `/27`, and `/28` subnets.

## Example

For:

```text
192.168.45.173/27
```

```text
Subnet Mask:    255.255.255.224
Block Size:     32
Network:        192.168.45.160
Broadcast:      192.168.45.191
First Usable:   192.168.45.161
Last Usable:    192.168.45.190
Usable Hosts:   30
```

## VLSM

For VLSM, I learned to:

1. List the host requirements.
2. Arrange requirements from largest to smallest.
3. Choose the smallest subnet that can support each requirement.
4. Allocate the largest subnet first.
5. Continue from the next available address.

Example:

```text
100 hosts → /25
50 hosts  → /26
25 hosts  → /27
10 hosts  → /28
```

## Challenge and How I Solved It

My main challenge was understanding subnet boundaries and correctly identifying the first usable address, especially when the changing octet was not the fourth octet. I solved this by consistently using the block-size method and writing out the subnet boundaries before calculating the network, broadcast, and usable ranges. I also practiced VLSM repeatedly until I could allocate different-sized subnets without overlapping them.
