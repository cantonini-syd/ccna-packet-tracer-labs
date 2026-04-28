# Lab 04 — DHCP Configuration

## Source
Based on Jeremy's IT Lab CCNA Packet Tracer practice lab. 

## Objective
Configure a Cisco router as a DHCP server to dynamically assign IP addresses, default gateways, and DNS server addresses to hosts. Also configure a second router as both a DHCP client and a DHCP relay agent for a remote subnet.

## Topology
- PC1 and PC2 → SW1 → R1 G0/1 (10.0.0.0/24 subnet)
- R1 G0/0 (.1) ↔ R2 G0/0 (.1) (192.168.12.0/24 point-to-point link)
- R2 G0/1 (.1) → SW2 → PC3 and PC4 (20.0.0.0/24 subnet)

R1 is the DHCP server for all three subnets. R2 acts as a relay agent for the 20.0.0.0/24 subnet since those hosts are not directly connected to R1.

## Tasks completed
- Created three DHCP pools on R1:
  - **10pool** — 10.0.0.0/24, gateway 10.0.0.1, DNS 10.0.0.1, excluded 10.0.0.1–10.0.0.10
  - **20pool** — 20.0.0.0/24, gateway 20.0.0.1, DNS 20.0.0.1, excluded 20.0.0.1–20.0.0.10
  - **12pool** — 192.168.12.0/24 (for the point-to-point link between R1 and R2)
- Configured R2 G0/0 as a DHCP client (`ip address dhcp`) and enabled the interface
- Configured R2 G0/1 as a DHCP relay agent using `ip helper-address 192.168.12.1`
- Verified PC1, PC2, PC3, and PC4 all received correct addresses, gateways, and DNS via DHCP

## My approach
Started on R1 and built the DHCP pools one at a time. The excluded-address ranges are configured in global config mode, not inside the pool — easy to forget. Excluded 10.0.0.1–10 and 20.0.0.1–10 to reserve the lower addresses for infrastructure (routers, switches, servers). Then moved to R2 and set G0/0 to `ip address dhcp` so R2 picks up an address from the 12pool on R1. The relay agent was a single command on R2's G0/1: `ip helper-address 192.168.12.1` pointing at R1's address, which forwards the DHCP broadcasts from the 20.0.0.0/24 subnet as unicast to R1.

## Verification
- On PC1: `ipconfig /release` then `ipconfig /renew` — received 10.0.0.11 (first address outside the excluded range), gateway 10.0.0.1, DNS 10.0.0.1 
- On PC3: same release/renew — received address in the 20.0.0.0/24 range, gateway 20.0.0.1, DNS 20.0.0.1 (confirming relay agent works)
- `show ip dhcp binding` on R1 — confirmed leased addresses for all clients
- `show ip dhcp pool` on R1 — confirmed pool utilisation
- `show ip interface brief` on R2 — confirmed G0/0 received a DHCP-assigned address in 192.168.12.0/24

## Troubleshooting / learnings
The relay agent concept clicked in this lab. DHCP discovery is a broadcast, and broadcasts don't cross router boundaries — so without the relay agent on R2, hosts on the 20.0.0.0/24 subnet would never reach R1 even though R1 has a pool configured for that range. The `ip helper-address` command converts the broadcast into a unicast aimed at the DHCP server, which is why the relay must be configured on the interface closest to the clients (R2 G0/1), not on R1. Also learned that the excluded-address command is global, not per-pool — it applies to any pool whose range overlaps with the exclusion. In production, you'd typically exclude the default gateway address plus a few extras for static infrastructure IPs.

## Files
- `R1-config.txt` — running configuration from R1
- `R2-config.txt` — running configuration from R2
