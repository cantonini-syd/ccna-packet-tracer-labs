# Lab 05 — Extended ACLs

## Source
Based on Jeremy's IT Lab CCNA Packet Tracer practice lab. Original lab and topology by Jeremy Cioara at jeremysitlab.com.

## Objective
Configure a single numbered extended ACL to control which hosts can reach which servers, filtering on both source and destination IP addresses. Extended ACLs provide more granular control than standard ACLs because they can match on protocol, source, destination, and port — not just source address.

## Topology
Three subnets connected by two routers (R1 and R2) via a serial link (12.0.0.0/24):

- **192.168.1.0/24** — PC1 (.11), PC2 (.12) → SW1 → R1 F0/0
- **192.168.2.0/24** — PC3 (.13), PC4 (.14) → SW2 → R1 F1/0
- **12.0.0.0/24** — R1 S2/0 (.1) ↔ R2 S2/0 (.2) serial WAN link
- **192.168.3.0/24** — SRV1 (.100), SRV2 (.101) → SW3 → R2 F0/0

## Requirements
- Only PC1 (192.168.1.11) can access SRV1 (192.168.3.100) — all other hosts blocked from SRV1
- Only hosts on the 192.168.2.0/24 network can access SRV2 (192.168.3.101) — all other hosts blocked from SRV2
- All other traffic not matching these rules should still be permitted (override the implicit deny)

## Tasks completed
- Created extended ACL 100 on R1 with five entries:
  1. `permit ip host 192.168.1.11 host 192.168.3.100` — allow PC1 → SRV1
  2. `deny ip any host 192.168.3.100` — block everyone else → SRV1
  3. `permit ip 192.168.2.0 0.0.0.255 host 192.168.3.101` — allow 192.168.2.0/24 → SRV2
  4. `deny ip any host 192.168.3.101` — block everyone else → SRV2
  5. `permit ip any any` — allow all other traffic (overrides implicit deny)
- Applied ACL 100 outbound on R1 S2/0 (`ip access-group 100 out`)
- Verified correct permit/deny behaviour from all four PCs

## My approach
Extended ACLs should be applied as close to the source as possible — that's R1's S2/0 (outbound toward R2 and the servers). Used a single ACL with carefully ordered entries: specific permits first, then the deny for that server, then the next server's permit/deny pair, and finally a `permit any any` at the bottom to avoid blocking unrelated traffic. Order matters because IOS evaluates ACL entries top-down and stops at the first match.

The wildcard mask `0.0.0.255` in entry 3 matches any host in the 192.168.2.0/24 subnet. The `host` keyword is shorthand for a wildcard mask of `0.0.0.0` (match exactly one IP).

## Verification
- **PC1 → SRV1** (ping 192.168.3.100): ✓ Success — PC1 is explicitly permitted
- **PC1 → SRV2** (ping 192.168.3.101): ✗ Denied — PC1 is in 192.168.1.0/24, not 192.168.2.0/24
- **PC2 → SRV1** (ping 192.168.3.100): ✗ Denied — only PC1 is permitted to SRV1
- **PC2 → SRV2** (ping 192.168.3.101): ✗ Denied — PC2 is in 192.168.1.0/24
- **PC3 → SRV1** (ping 192.168.3.100): ✗ Denied — not PC1
- **PC3 → SRV2** (ping 192.168.3.101): ✓ Success — PC3 is in 192.168.2.0/24
- **PC4 → SRV1** (ping 192.168.3.100): ✗ Denied — not PC1
- **PC4 → SRV2** (ping 192.168.3.101): ✓ Success — PC4 is in 192.168.2.0/24

All results match the requirements.

## Troubleshooting / learnings
The biggest takeaway is **ACL entry order**. If I had put `deny ip any host 192.168.3.100` before the permit for PC1, PC1 would also be blocked — IOS hits the first match and stops. Same reason the `permit ip any any` must go last as that overrides the implicit deny at the bottom of every ACL. 

Also reinforced the difference between standard and extended ACLs: standard ACLs (1–99) filter on source IP only and are applied close to the destination; extended ACLs (100–199) filter on source, destination, protocol, and port, and are applied close to the source. In a NOC context, extended ACLs are what you'd see in change requests — "allow this host to reach this server on this port" — so getting comfortable reading and writing them is essential.

`show access-lists` on R1 shows hit counts next to each entry, which is useful for confirming which rule matched and for troubleshooting unexpected blocks in production.

## Files
- `R1-config.txt` — running configuration from R1
