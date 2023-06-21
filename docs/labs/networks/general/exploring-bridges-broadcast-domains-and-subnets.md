Based on our progress in the last lab, [Connecting and Configuring Netork Hosts](connecting-and-configuring-network-hosts-in-proxmox.md),
we know that our four hosts are somehow able to communicate within their subnet.
But how exactly are the hosts connecting?

Recall that each host is configured with a network device connected to the `vmbr1`
bridge and assigned an IPv4 address in the `10.0.1.0/24` subnet. These two factors are the key to understanding what is happening behind the scenes.

In this lab, we're going to begin exploring what happens between the bridge, subnet,
and broadcasting domain to allow our hosts to communicate. Along the way, we'll see
how these fundamental concepts lay the foundation for us to build increasingly
complex labs.

## Examining the State of Our Network Hosts

The first thing we'll look at is the "state" of our network hosts. When `host1` sent
`ping` requests and an `nmap` scan to the other hosts, and received received
responses, it created records of how to reach them. But how did it find them?

Remeber the Address Resolution Protocol (ARP)? In IPv4, ARP is the first point of
contact between hosts on the same subnet. Because `host1` has already made contact
with each of the other hosts, it should have ARP records accessible that map each
host's IPv4 address with its MAC address, which is what is actually used to
communicate here on Layer 2.

When hosts are restarted, however, the ARP table is flushed, so let's start out with
a clean ARP table here, too, and populate it again so you can visibly see what's
happening. First flush `host1`'s ARP table:

```
ip neighbor flush dev eth0
```

Then perform the `nmap` network scan again:

```
nmap -sn 10.0.1.0/24
```

which should produce similar output:

```
Starting Nmap 7.80 ( https://nmap.org ) at 2023-06-21 19:07 UTC
Nmap scan report for 10.0.1.1
Host is up (0.00048s latency).
Nmap scan report for 10.0.1.2
Host is up (0.00044s latency).
Nmap scan report for 10.0.1.3
Host is up (0.00022s latency).
Nmap scan report for 10.0.1.4
Host is up (0.00013s latency).
Nmap done: 256 IP addresses (4 hosts up) scanned in 16.61 seconds
```


Now that we have a fresh network scan, let's look at `host1`'s ARP table
using the `ip` tool, part of the `iproute2` suite of tools we'll be using
regularly in our labs:

```
ip neighbor
```

which should output the following:

```
10.0.1.3 dev eth0 lladdr 00:50:56:16:30:c7 REACHABLE
10.0.1.4 dev eth0 lladdr 00:50:56:ad:24:4a REACHABLE
10.0.1.2 dev eth0 lladdr 00:50:56:ad:0e:33 REACHABLE
```

## Dipping Our Toes into Bridging Details

## Digging into Subnets and Broadcast Domains

Must be on the same bridge
Must be on the same subnet