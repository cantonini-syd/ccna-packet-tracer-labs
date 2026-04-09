# CCNA Lab Solutions and Walkthroughs

My solutions and walkthroughs for **Jeremy's IT Lab** CCNA Packet Tracer practice exercises. This repository documents my hands-on work with a focus on the configuration, verification, and troubleshooting workflow a network operations engineer uses day to day.

## About this repository

Each lab is in its own folder containing:

- A **README** describing the objective, topology, tasks, my approach, the verification commands I used, and what I learned (including any mistakes and how I caught them)
- **IOS configuration files** (`.txt`) — the running-config exported from each device after I completed the lab

## Why I built this

I'm transitioning into network operations. Working through labs and writing up is how I'm building both the muscle memory for the CLI and the discipline of documenting changes the way I'd be expected to in a real NOC. 

## Lab index

| # | Lab | Topics |
|---|-----|--------|
| 01 | [VLANs and Trunking](./lab-01-vlans-and-trunking) | VLAN creation, access ports, 802.1Q trunking, native VLAN behaviour |
| 02 | [SSH Remote Access Configuration](./lab-02-ssh-configuration) | RSA key generation, VTY hardening, SSHv2, disabling Telnet |
| 03 | [OSPF Single-Area](./lab-03-ospf-single-area) | Loopback router IDs, passive interfaces, reference bandwidth, interface cost manipulation |

*More labs added as I work through them.*

## Tools

- Cisco Packet Tracer
- Cisco IOS 12.2 (switches) / 15.1 (routers)

## Credit

All lab scenarios are based on the free CCNA practice labs published by **Jeremy Cioara** [jeremysitlab.com](https://jeremysitlab.com) and his accompanying YouTube videos. Jeremy's content is an outstanding free resource for anyone studying networking — if you find these walkthroughs useful, go support him directly.

## License

The walkthroughs and configurations in this repository are released under the MIT License (see `LICENSE`). 
