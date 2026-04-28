# CCNA Lab Solutions and Walkthroughs

My solutions and walkthroughs for **Jeremy's IT Lab** CCNA Packet Tracer practice exercises. This repository documents my hands-on work with a focus on the configuration, verification, and troubleshooting workflow a network operations engineer uses day to day.

## About this repository

Each lab is in its own folder containing:

- A **README** describing the objective, topology, tasks, my approach, the verification commands I used, and what I learned (including any mistakes and how I caught them)
- **IOS configuration files** (`.txt`) - the running-config exported from each device after I completed the lab

## Why I built this

I'm building a career in IT infrastructure and security. Working through labs and writing up each one is how I'm developing hands-on CLI skills and the habit of documenting changes clearly — both essential in any operations or engineering role.

## Lab index

| # | Lab | Topics |
|---|-----|--------|
| 01 | [VLANs and Trunking](./lab-01-vlans-and-trunking) | VLAN creation, access ports, 802.1Q trunking, native VLAN behaviour |
| 02 | [SSH Remote Access Configuration](./lab-02-ssh-configuration) | RSA key generation, VTY hardening, SSHv2, disabling Telnet |
| 03 | [OSPF Single-Area](./lab-03-ospf-single-area) | Loopback router IDs, passive interfaces, reference bandwidth, interface cost manipulation |
| 04 | [DHCP Configuration](./lab-04-dhcp-configuration) | DHCP pools, excluded addresses, DHCP client on a router, relay agent with ip helper-address |
| 05 | [Extended ACLs](./lab-05-extended-acls) | Numbered extended ACLs, source/destination filtering, wildcard masks, ACL entry ordering, implicit deny |

## Tool

- Cisco Packet Tracer

## Credit

All lab scenarios are based on the free CCNA practice labs published by **Jeremy Cioara** [jeremysitlab.com](https://jeremysitlab.com) and his accompanying YouTube videos.
