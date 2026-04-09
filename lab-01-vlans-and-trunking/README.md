# Lab 01 — VLANs and Trunking

## Source
Based on Jeremy's IT Lab CCNA Packet Tracer practice lab.

## Objective
Use VLANs to isolate hosts at Layer 2 even though they share the same Layer 3 subnet, and configure a trunk link between two switches so multiple VLANs can pass between them.

## Topology
Two Cisco 2960-24TT switches (SW1, SW2) connected via Fa0/1 ↔ Fa0/1. Four PCs all in the 10.0.0.0/24 subnet:
- PC1 (10.0.0.1/24) → SW1 Fa0/2
- PC2 (10.0.0.2/24) → SW1 Fa0/3
- PC3 (10.0.0.3/24) → SW2 Fa0/2
- PC4 (10.0.0.4/24) → SW2 Fa0/3

PC1 and PC3 are assigned to VLAN 1, PC2 and PC4 to VLAN 2.

## Tasks completed
- Verified initial Layer 3 connectivity between all four PCs
- Configured SW1 Fa0/2 and SW2 Fa0/2 as access ports in VLAN 1
- Configured SW1 Fa0/3 and SW2 Fa0/3 as access ports in VLAN 2
- Tested connectivity again to observe the native VLAN behaviour
- Configured Fa0/1 on both switches as 802.1Q trunk ports
- Verified VLAN isolation across the trunk

## My approach
Started by pinging between all PCs to confirm baseline reachability — all four were in the same /24, so everything worked. Then assigned access ports on both switches and explicitly set `switchport mode access` even though access is the default, since being explicit makes the config easier to read later. After the second round of pings I saw PC2 ↔ PC4 fail while PC1 ↔ PC3 still worked, which is the native VLAN behaviour — VLAN 1 traffic crosses the inter-switch link untagged by default. Fixed it by setting Fa0/1 on both switches to `switchport mode trunk` so tagged VLAN 2 frames could traverse the link.

## Verification
- `show running-config` — confirmed access port and VLAN assignments on both switches
- Pings from PC1: PC3 succeeded (same VLAN), PC2 and PC4 failed (different VLAN)
- Pings from PC2: PC4 succeeded (same VLAN, after trunk config), PC1 and PC3 failed

The expected isolation behaviour was achieved: hosts in the same VLAN can reach each other across the trunk, hosts in different VLANs cannot, even though all four share the same /24.

## Troubleshooting / learnings
The key gotcha here was the **native VLAN**. When I first assigned PCs to VLANs but hadn't yet configured the trunk, PC1 ↔ PC3 (VLAN 1) still worked while PC2 ↔ PC4 (VLAN 2) failed. That's because Fa0/1 on each switch was still a default access port in VLAN 1, which is also the default native VLAN — so VLAN 1 traffic passes untagged and "just works", while VLAN 2 frames have nowhere to go. Also noticed that explicitly assigning an interface to VLAN 1 doesn't show up in `show running-config` because VLAN 1 is the default. In production the native VLAN should be changed away from VLAN 1 for security (mitigates VLAN hopping), but this lab kept the default.

## Files
- `SW1-config.txt` — running configuration from SW1
- `SW2-config.txt` — running configuration from SW2
