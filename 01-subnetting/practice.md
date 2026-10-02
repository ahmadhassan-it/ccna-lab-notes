# Subnetting Practice Set
[← Back to Subnetting notes](README.md)

Solve each problem by hand first (block size method). Answers are at the bottom.

## Part A: Find network, broadcast, and host range

1. 192.168.1.77/26
2. 10.0.0.200/27
3. 172.16.5.50/28
4. 192.168.20.130/25
5. 192.168.100.9/29
6. 10.10.10.5/30

## Part B: Quick questions

7. An office needs at least 40 hosts. What is the smallest prefix that fits?
8. Convert the mask 255.255.255.240 to CIDR.
9. Are 192.168.1.100/26 and 192.168.1.130/26 in the same subnet?

## Part C: VLSM

10. Split 192.168.30.0/24 for these needs: Sales 60 hosts, IT 25 hosts, Finance 10 hosts, one router link (2 hosts). Allocate the largest first.

---

## Answers

| # | Network | Broadcast | Usable range | Hosts |
|---|---------|-----------|--------------|-------|
| 1 | 192.168.1.64 | 192.168.1.127 | .65 to .126 | 62 |
| 2 | 10.0.0.192 | 10.0.0.223 | .193 to .222 | 30 |
| 3 | 172.16.5.48 | 172.16.5.63 | .49 to .62 | 14 |
| 4 | 192.168.20.128 | 192.168.20.255 | .129 to .254 | 126 |
| 5 | 192.168.100.8 | 192.168.100.15 | .9 to .14 | 6 |
| 6 | 10.10.10.4 | 10.10.10.7 | .5 to .6 | 2 |

7. /26 (62 usable hosts). /27 gives only 30, which is too small.
8. /28 (240 = 4 network bits in the last octet, 24 + 4 = 28).
9. No. .100 is in the .64 block (64 to 127). .130 is in the .128 block (128 to 191).

10. VLSM answer:

| Segment | Needs | Prefix | Network | Usable range | Broadcast |
|---------|-------|--------|---------|--------------|-----------|
| Sales | 60 | /26 | 192.168.30.0 | .1 to .62 | .63 |
| IT | 25 | /27 | 192.168.30.64 | .65 to .94 | .95 |
| Finance | 10 | /28 | 192.168.30.96 | .97 to .110 | .111 |
| Router link | 2 | /30 | 192.168.30.112 | .113 to .114 | .115 |

Why largest first: big blocks must start on a matching boundary. If you place small subnets first, they can block the space a big subnet needs.
