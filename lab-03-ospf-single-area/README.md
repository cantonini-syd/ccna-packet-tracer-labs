# Lab 03 — OSPF Single-Area

## Source
Based on Jeremy's IT Lab CCNA Packet Tracer practice lab. 

## Objective
Configure single-area OSPF across a four-router topology, suppress unnecessary advertisements on loopback interfaces, tune the reference bandwidth so modern link speeds are distinguished in path selection, and then influence path selection by manipulating interface cost.

## Topology
Four routers (R1–R4) in OSPF Area 0. Physical links form a square: R1↔R2 (12.0.0.0/24), R2↔R3 (23.0.0.0/24), R1↔R4 (14.0.0.0/24), R4↔R3 (34.0.0.0/24). The R1↔R4 and R3↔R4 links are GigabitEthernet; the R1↔R2 and R2↔R3 links are FastEthernet.

Loopbacks (one per router, used as OSPF router IDs):
- R1 Lo0: 1.1.1.1/32
- R2 Lo0: 2.2.2.2/32
- R3 Lo0: 3.3.3.3/32
- R4 Lo0: 4.4.4.4/32

## Tasks completed
- Configured loopback interfaces on all four routers before enabling OSPF, so they would be picked up as router IDs
- Enabled OSPF on all routers
- Advertised all interfaces using the `network 0.0.0.0 255.255.255.255 area 0` shortcut
- Configured Lo0 as passive on every router to suppress unnecessary OSPF hellos
- Set `auto-cost reference-bandwidth 10000` on all four routers so a 10 Gbps interface has a cost of 1
- Adjusted the OSPF cost on R1 G0/0 and R4 G0/0 (the R1↔R4 link) to 10000, forcing R1 to reach R3's loopback via R2 instead of R4

## My approach
Built loopbacks first, because if OSPF is already running when you add a new loopback, the router won't re-elect its router ID without an OSPF process reset, and that's something you can't casually do in production. For the network statements I used a lab-only shortcut `network 0.0.0.0 255.255.255.255 area 0`, which activates OSPF on every interface, but in a real network I'd use specific network statements or the `ip ospf 1 area 0` interface-level command to be explicit about which interfaces participate. After verifying full reachability, I set the reference bandwidth to 10000 Mbps so that Gigabit interfaces have cost 10 and Fast Ethernet has cost 100, which made OSPF prefer the R1→R4→R3 path to R3's loopback over R1→R2→R3 (two Fast hops). Then reversed it by manually raising the cost on the R1↔R4 link to 10000, forcing traffic back through R2.

## Verification
- `show ip ospf neighbor` on R1 — confirmed FULL adjacencies with R2 (via F1/0) and R4 (via G0/0), and showed 2.2.2.2 and 4.4.4.4 as the neighbor router IDs (confirming the loopbacks were picked up)
- `show ip ospf interface brief` — confirmed OSPF active on the expected interfaces and Lo0 marked passive
- `show ip route ospf` on R1 — before reference-bandwidth change: equal-cost load-balance to 3.3.3.3 via both R2 and R4; after the change: single best path via R4 (gigabit); after the interface cost override: single best path via R2
- `show ip ospf interface g0/0` on R1 — confirmed cost of 10000 after the manual override

## Troubleshooting / learnings
The reference-bandwidth default of 100 Mbps means OSPF can't tell the difference between Fast Ethernet, Gigabit, 10 Gig, or 100 Gig links out of the box because the cost formula bottoms out at 1. If you don't bump it, OSPF will happily pick a slow path that has fewer FastEthernet hops over a fast path with more Gigabit hops, which is the opposite of what you want. If you change `auto-cost reference-bandwidth` on one router in an area, change it on all of them, inconsistent values cause mismatched costs and unpredictable path selection.

## Files
- `R1-config.txt` — running configuration from R1
- `R2-config.txt` — running configuration from R2
- `R3-config.txt` — running configuration from R3
- `R4-config.txt` — running configuration from R4
