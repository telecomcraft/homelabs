---
tags:
- MikroTik
- RouterOS
- Linux
- Proxmox
---

# Static Routing Between Subnets with RouterOS [DRAFT]

## Lab Setup

For this lab, we'll need the following to be pre-configured:

- `host1` is on `vmbr1` and has an IPv4 address of `10.0.1.10/24`
- `host2` is on `vmbr1` and has an IPv4 address of `10.0.1.11/24`
- `host3` is on `vmbr1` and has an IPv4 address of `10.0.2.10/24`
- `host4` is on `vmbr1` and has an IPv4 address of `10.0.2.11/24`
- `routeros-r1` is on `vmbr1` and has an IPv4 address on `ether1` of `10.0.1.1/24`
- `routeros-r2` is on `vmbr1` and has an IPv4 address on `ether1` of `10.0.2.1/24`
