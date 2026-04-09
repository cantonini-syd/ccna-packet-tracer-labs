# Lab 02 — SSH Remote Access Configuration

## Source
Based on Jeremy's IT Lab CCNA Packet Tracer practice lab.

## Objective
Configure secure remote management access to a Cisco router and switch using SSH instead of Telnet, so that management traffic between the admin PC and the devices is encrypted.

## Topology
- PC1 (192.168.1.11/24) → SW1 Fa0/1
- SW1 (Cisco 2960-24TT, VLAN1 SVI 192.168.1.2/24) → R1 G0/0 via SW1 G0/1
- R1 (Cisco 1941, G0/0 192.168.1.1/24)

All devices in the 192.168.1.0/24 management subnet.

## Tasks completed
- Set hostnames on SW1 and R1
- Configured R1 G0/0 and SW1 VLAN1 SVI with management IPs
- Created local user account `cisco` / password `CCNA` on both devices
- Configured DNS domain name `cisco.com` on both devices
- Generated 1024-bit RSA crypto keys on both devices
- Configured VTY lines 0–15 to use the local user database, allow SSH only, and time out after 5 minutes
- Enabled SSH version 2 on both devices
- Verified Telnet is rejected and SSH login from PC1 succeeds to both devices

## My approach
SSH on Cisco IOS has prerequisites that must be in place before the key generation will even work — hostname must not be the default, and a domain name must be configured, because the RSA key's fully-qualified name is built from `hostname.domain`. So I worked through it in dependency order: hostname → IPs → local user → domain name → `crypto key generate rsa` → VTY config → enable SSHv2. Set `transport input ssh` (not `telnet ssh`) on the VTY lines to explicitly disable Telnet, and `exec-timeout 5` to drop idle sessions after five minutes.

## Verification
- `show running-config` on both devices — confirmed hostname, domain name, username, VTY config, and `ip ssh version 2`
- `show ip ssh` — confirmed SSH enabled, version 2.0
- `show ssh` — viewed active sessions after connecting from PC1
- From PC1 command prompt: `telnet 192.168.1.2` → connection refused ✓ (expected, Telnet disabled)
- From PC1: `ssh -l cisco 192.168.1.2` → prompted for password, logged into SW1 
- From PC1: `ssh -l cisco 192.168.1.1` → prompted for password, logged into R1 

## Troubleshooting / learnings
The four prerequisites for SSH on IOS are easy to forget and the error messages aren't always obvious if you skip one — `crypto key generate rsa` will fail outright if the hostname is still `Switch` or there's no domain name set, because it can't build a key label. Got into the habit of doing hostname + domain name first, every time, before touching crypto. Also worth noting that `transport input ssh` is the security-relevant command — without it, Telnet would still be accepted on the VTY lines even after enabling SSH, which defeats the point. In production, the local user password should be hashed with `username cisco secret CCNA` rather than `password CCNA` (the latter stores it in clear text or weak Type 7 unless `service password-encryption` is enabled), and the keys should be at least 2048 bits — 1024 is fine for a lab but considered weak now.

## Files
- `SW1-config.txt` — running configuration from SW1
- `R1-config.txt` — running configuration from R1
