# VLSM Design
## Lab Topology

![Topology](topology.png)

## Verification

PC1 configuration:

![PC1 ipconfig](pc1-ipconfig.png)

Router interface status:

![Interface status](ip-interface-brief.png)

Ping from PC1 to PC3:

![Ping test](ping-test.png)
## Method

1. List the host needs.
2. Sort from largest to smallest.
3. Pick the smallest prefix that fits each need.
4. Allocate the largest first, then continue from the next free address.

## Worked Example

Network: 192.168.10.0/24

| Segment | Hosts needed | Prefix | Network | Usable range | Broadcast |
|---------|-------------:|--------|---------|--------------|-----------|
| A | 100 | /25 | 192.168.10.0 | .1 – .126 | .127 |
| B | 50 | /26 | 192.168.10.128 | .129 – .190 | .191 |
| C | 25 | /27 | 192.168.10.192 | .193 – .222 | .223 |
| D | 10 | /28 | 192.168.10.224 | .225 – .238 | .239 |

Free space left: 192.168.10.240 – .255 (one /28 block).

Why largest first: each subnet must start on a multiple of its block size. If small subnets go first, they can leave gaps that a large subnet cannot use.

## My Lab Problem

(Fill this in from the VLSM problem you actually solved with 192.168.10.0/24.)

Requirements:

| Segment | Hosts needed |
|---------|-------------:|
| | |

My solution:

| Segment | Prefix | Network | Usable range | Broadcast |
|---------|--------|---------|--------------|-----------|
| | | | | |

What I got wrong at first and how I fixed it:
