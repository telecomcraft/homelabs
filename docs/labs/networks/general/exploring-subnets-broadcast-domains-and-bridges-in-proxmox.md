---
tags:
  - Proxmox
  - Linux
---

# Exploring Subnets, Broadcast Domains, and Bridges in Proxmox

Based on our progress in the last lab, [Connecting and Configuring Network Hosts in Proxmox](connecting-and-configuring-network-hosts-in-proxmox.md),
we know that our four hosts are somehow able to communicate within their subnet.
But how exactly are the hosts connecting?

Recall that each host is configured with a network device connected to the `vmbr1`
bridge and assigned an IPv4 address in the `10.0.1.0/24` subnet. These two factors are the key to understanding what is happening behind the scenes.

In this lab, we're going to begin exploring the concepts of subnets, broadcasting
domains, and bridges to allow our hosts to communicate (or not, in some cases).
Along the way, we'll see how these fundamental concepts lay the foundation for us
to build increasingly complex labs.

## Step 1: Examining the State of Our Network Hosts

The first thing we'll look at is the "state" of our network hosts. When `host1` sent
`ping` requests and an `nmap` scan to the other hosts, and received responses, it
created records of how to reach them. But how did it find them?

Remember the Address Resolution Protocol (ARP)? In IPv4, ARP is the first point of
contact between hosts on the same subnet. Because `host1` has already made contact
with each of the other hosts, it should have ARP records accessible that map each
host's IPv4 address with its MAC address, which is what is actually used to
communicate here on Layer 2.

When hosts are restarted, however, the ARP cache is flushed, so let's start out with
a clean ARP cache here, too, and populate it again so you can visibly see what's
happening. First flush `host1`'s ARP cache:

```
ip neighbor flush dev eth0
```

Then perform the `nmap` network scan again:

```
nmap -sn 10.0.1.0/24
```

which should produce something similar to the following output:

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

Now that we have a fresh network scan, let's look at `host1`'s ARP cache
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

Entries in a device's ARP cache are cached for a certain amount of time and
then cleared out, with the "soft" limit being once 512 entries are reached
and the "hard" limit being when there are 1,024 entries, at which point the
cache is cleared.

Check the ARP cache again:

```
ip neighbor
```

and you will see the status of the existing entries have changed, and now indicate
they may be flushed if new MAC addresses continue actively populating the ARP cache:

```
10.0.1.3 dev eth0 lladdr 00:50:56:16:30:c7 STALE
10.0.1.4 dev eth0 lladdr 00:50:56:ad:24:4a STALE
10.0.1.2 dev eth0 lladdr 00:50:56:ad:0e:33 STALE
```

We won't go much deeper into the innerworkings of the ARP cache at this point,
but we will study how the protocol works for neighbor discovery, so you can
see what goes on when hosts attempt to reach each other and how it relates to
the ARP cache.

To do this, let's operate from `host2`, and observe its interactions with `host1` at
the protocol level during a `ping` test. To begin, clear out `host2`'s ARP cache so
we're starting fresh again:

```
sudo ip neighbor flush dev eth0
```

Return to `host1`, and begin a packet capture session using `tcpdump`, another crucial
tool we'll be using constantly in our labs:

```
sudo tcpdump
```

Switch back to `host2`, where we're going to send `host1` a single `ping` request:

```
sudo ping -c 1 10.0.1.1
```

which should produce the following output:

```
PING 10.0.1.1 (10.0.1.1) 56(84) bytes of data.
64 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=0.148 ms

--- 10.0.1.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.148/0.148/0.148/0.000 ms
```

Here you can see that one request packet was sent from `host1` to `host2`, and a reply
packet from `host2` to `host1` was also successfully transferred, resulting in no packet
loss. The two hosts successfully communicated, which means each should now have a record
of the other in its ARP cache. Let's check `host2`'s ARP cache:

```
ip neighbor
```

which shows a fresh entry for `host2`:

```
10.0.1.1 dev eth0 lladdr 00:50:56:94:55:70 REACHABLE
```

This means that not only was `host2`'s `ping` request successful, but that `host1` was
properly captured in the ARP cache. The same is also true on the other side for `host1`.
How did all of this happen exchanging just two packets? It didn't—there were six
packets involved.

Let's return to `host1`, and cancel out (`CTRL`+`C`
on Windows) of the packet capture. Your output should look very similar to this:

```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
19:21:42.310066 ARP, Request who-has 10.0.1.1 tell 10.0.1.2, length 28
19:21:42.310088 ARP, Reply 10.0.1.1 is-at 00:50:56:94:55:70 (oui Unknown), length 28
19:21:42.310132 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 21255, seq 1, length 64
19:21:42.310140 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 21255, seq 1, length 64
19:21:47.343328 ARP, Request who-has 10.0.1.2 tell 10.0.1.1, length 28
19:21:47.343360 ARP, Reply 10.0.1.2 is-at 00:50:56:ad:0e:33 (oui Unknown), length 28
^C
6 packets captured
6 packets received by filter
0 packets dropped by kernel
```

We'll dig deep into analyzing packets in future labs, but knowing the basics is an
invaluable skill from the beginning of your learning to enhance your ability to
observe how protocols work and troubleshoot network issues that inevitably arise. So
let's start with the packet capture from `host2`.

On line 2, you see that `tcpdump` was listening on `host1`'s `eth0` interface, which
is connected to `host2` via bridge `vmbr1` (again, think "switch"). That's obvious
for this lab, as there's only one interface, but always check on devices with
multiple ports.

The next two lines, 3 and 4, are ARP packets. The first, at 19:21:42.310066, is a
broadcast request from the bridge `vmbr1` to all hosts on the subnet asking who is
`10.0.1.1`, as `10.0.1.2` needs to know. The next packet, at 19:21:42.310088, is 
the broadcast reply from `10.0.1.1` letting the bridge and `10.0.1.2` that it has
that IP address, and its MAC address is `00:50:56:94:55:70` (we'll discuss OUIs
later).

The bridge, like any switch, then sends the 3rd packet, at 19:21:42.310132, which
is the ICMP echo request (the formal term for the protocol used by `ping`), to
`host1`, now knowing who it is. `host1` responds with the proper ICMP echo reply
in the packet at 19:21:42.310140, addressed to `10.0.1.2`, but not sure which host
that is.

As a result, `host1` also utilizes ARP in packet 5, at 19:21:47.343328, broadcasting
a request asking who has `10.0.1.2`, as `10.0.1.1` needs to know. The sixth packet,
at 19:21:47.343360, is the broadcast reply from `10.0.1.2` letting the bridge and
`10.0.1.2` that it has that IP address, and its MAC address is `00:50:56:ad:0e:33`.

When the packet capture was stopped, it completed its job by reporting the number
of packets captured, filtered, and dropped. We didn't have a filter defined, such
as filtering out ARP packets, and no packets were dropped because of other
transmission issues, so it was a successful job.

One last point about ARP before we move on. Once `host1` and `host2` know each other,
is ARP no longer necessary, since they can simply address each other directly during
communication? As with anything in our labs, let's test it out.

Just like before, put `host1` into a packet capture mode:

```
sudo tcpdump
```

and have `host2` ping `host1`. This time, however, have `host2` flood ping `host1` 25
times:

```
sudo ping -f -c 25 10.0.1.1
```

This should result in something like the following:

```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
02:14:34.087556 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 1, length 64
02:14:34.087576 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 1, length 64
02:14:34.087644 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 2, length 64
02:14:34.087647 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 2, length 64
02:14:34.087702 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 3, length 64
02:14:34.087705 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 3, length 64
02:14:34.087740 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 4, length 64
02:14:34.087743 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 4, length 64
02:14:34.087769 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 5, length 64
02:14:34.087772 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 5, length 64
02:14:34.087797 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 6, length 64
02:14:34.087800 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 6, length 64
02:14:34.087845 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 7, length 64
02:14:34.087848 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 7, length 64
02:14:34.087897 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 8, length 64
02:14:34.087900 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 8, length 64
02:14:34.087933 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 9, length 64
02:14:34.087936 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 9, length 64
02:14:34.087962 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 10, length 64
02:14:34.087964 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 10, length 64
02:14:34.087997 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 11, length 64
02:14:34.088000 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 11, length 64
02:14:34.088033 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 12, length 64
02:14:34.088036 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 12, length 64
02:14:34.088078 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 13, length 64
02:14:34.088081 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 13, length 64
02:14:34.088105 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 14, length 64
02:14:34.088108 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 14, length 64
02:14:34.088141 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 15, length 64
02:14:34.088144 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 15, length 64
02:14:34.088178 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 16, length 64
02:14:34.088180 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 16, length 64
02:14:34.088221 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 17, length 64
02:14:34.088224 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 17, length 64
02:14:34.088255 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 18, length 64
02:14:34.088257 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 18, length 64
02:14:34.088295 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 19, length 64
02:14:34.088298 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 19, length 64
02:14:34.088329 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 20, length 64
02:14:34.088331 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 20, length 64
02:14:34.088373 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 21, length 64
02:14:34.088375 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 21, length 64
02:14:34.088414 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 22, length 64
02:14:34.088417 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 22, length 64
02:14:34.088448 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 23, length 64
02:14:34.088450 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 23, length 64
02:14:34.088475 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 24, length 64
02:14:34.088478 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 24, length 64
02:14:34.088502 IP 10.0.1.2 > 10.0.1.1: ICMP echo request, id 9783, seq 25, length 64
02:14:34.088504 IP 10.0.1.1 > 10.0.1.2: ICMP echo reply, id 9783, seq 25, length 64
02:14:39.183337 ARP, Request who-has 10.0.1.2 tell 10.0.1.1, length 28
02:14:39.183380 ARP, Reply 10.0.1.2 is-at 00:50:56:ad:0e:33 (oui Unknown), length 28
^C
52 packets captured
52 packets received by filter
0 packets dropped by kernel
```

So the basic answer is that, for the most part, once hosts have cached their addresses
in the ARP cache, they can communicate via unicast transmission. Every so often,
however, ARP checks are sent to verify host addressing, thus the two extra packets in
addition to the 25 pairs of ICMP request/reply pairs.

!!! question

    Wondering how often ARP is part of the transmissions between known hosts? Modify
    the `ping` command above to use higher count flood pings and see what happens. If
    it's too many, does the kernel drop packets?

As we saw with the ARP process, communication between hosts begins with broadcasts
across their subnet to determine which hosts have which MAC and IP addresses. Let's
move on and learn more about the role of subnets and broadcast domains.

## Step 2: Digging into IPv4 Subnets and Broadcast Domains

Currently our network consists of four hosts all assigned IPv4 addresses in the
same subnet of `10.0.1.0/24`, all contained in the same Layer 2 bridge `vmbr1`.
Because they are in the same subnet, and on the same bridge, they should always
be able to communicate. We've verifed this using both `ping` and `nmap` tests.

Performing a packet capture, we've also seen what happens during communication
at the protocol layer, with hosts broadcasting ARP requests/replies to all hosts
when identifying each other. How does broadcasting work within a subnet?

As we know, a subnet is a defined IP space assigned to hosts. So far, we have
assigned the IPs

* `10.0.1.1` to `host1`
* `10.0.1.2` to `host2`
* `10.0.1.3` to `host3`
* `10.0.1.4` to `host4`

!!! note

    You might have noticed that the last "octet" of each host matches the number
    in each host name. IP addressing is generally arbitrary, but we'll do things
    like this in some cases to help you follow along in these early labs.

How many hosts can we have in this address space? You probably have seen many ways
to calculate subnet sizes, but let's make it easy by using another handy tool on
our Linux hosts: `ipcalc`.

Ask `ipcalc` to calculate the details of our `10.0.1.0/24` subnet from `host1`:

```
ipcalc 10.0.1.0/24
```

which reports back the following:

```
Address:   10.0.1.0             00001010.00000000.00000001. 00000000
Netmask:   255.255.255.0 = 24   11111111.11111111.11111111. 00000000
Wildcard:  0.0.0.255            00000000.00000000.00000000. 11111111
=>
Network:   10.0.1.0/24          00001010.00000000.00000001. 00000000
HostMin:   10.0.1.1             00001010.00000000.00000001. 00000001
HostMax:   10.0.1.254           00001010.00000000.00000001. 11111110
Broadcast: 10.0.1.255           00001010.00000000.00000001. 11111111
Hosts/Net: 254                   Class A, Private Internet
```

`ipcalc` has provided us everything we need to know about our subnet in a very
nice format. On the left is the name of each calculation, followed by the value
in IPv4 integer octets, then the value again formatted as binary octets (for all
IP-related data). The last line is a bit different, which we'll also cover
below.

* The Address line shows the address that we submitted in the command
* The Netmask line shows the netmask that we submitted in the command, both in
octet and CIDR format
* The Wildcard line shows the usable portion of the address within the subnet
while everything in the first three octets remains fixed
* The Network line shows the network ID for this subnet, which you'll recall
we used in our `nmap` network discovery scans
* The HostMin line shows the first address in this subnet available for host
assignment; and we've assigned this value to `host1`
* The HostMax line shows the last address in this subnet available for host
assignment, and we haven't assigned this value to any host yet
* The Broadcast line shows the broadcast address in this subnet
* The Hosts/Net line shows the total number of hosts available in this subnet,
which is 256 in our case

The Network address cannot be assigned to a host, thus it cannot respond to
`ping` requests, either. Let's verify this with a test from `host1`:

```
sudo ping -c 4 10.0.1.0
```

which results in no replies, as expected:

```
PING 10.0.1.0 (10.0.1.0) 56(84) bytes of data.
From 10.0.1.1 icmp_seq=1 Destination Host Unreachable
From 10.0.1.1 icmp_seq=2 Destination Host Unreachable
From 10.0.1.1 icmp_seq=3 Destination Host Unreachable
From 10.0.1.1 icmp_seq=4 Destination Host Unreachable

--- 10.0.1.0 ping statistics ---
4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3077ms
```

!!! tip

    Before we move on, bring up a detached console window for each host in Proxmox
    by clicking the `Console` button near the top-right of each guest view, and
    arrange them on your screen so they're all visible. Depending on your desktop,
    there are "quarter tile" features built-in, such as in Windows or KDE, or
    extentions available, such as [Rectangle](https://rectangleapp.com/) for macOS
    or [WinTile](https://extensions.gnome.org/extension/1723/wintile-windows-10-window-tiling-for-gnome/)
    for GNOME.

We already know we can ping `10.0.1.1` through `10.0.1.4`, and anything up to
`10.0.1.254` if we had hosts assigned to those IP addresses. But what about the
broadcast address? Don't hosts use that for ARP request/replies? Let's try to
ping it from `host1`:

```
sudo ping 10.0.1.255
```

and we get an interesting response:

```
ping: Do you want to ping broadcast? Then -b. If not, check your local firewall rules
```

The `ping` command is giving us a helpful tip: if we want to ping the broadcast
address, we need to add the `-c` flag to the command. Let's try again from
`host1`:

```
sudo ping -b 10.0.1.255
```

and now we get the following output:

```
WARNING: pinging broadcast address
PING 10.0.1.255 (10.0.1.255) 56(84) bytes of data.
```

At this point `ping` is sending ICMP requests, but nothing is visible in the
output like it would a normal ping test. How can we observe what's happening?
Let's check our other hosts.

On `host2`, `host3`, and `host4`, run the following command:

```
sudo tcpdump
```

Upon doing this, you should see something similar to the following:

```
01:15:56.463377 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 59888, seq 29, length 64
01:15:57.487378 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 59888, seq 30, length 64
01:15:58.515390 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 59888, seq 31, length 64
01:15:59.535379 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 59888, seq 32, length 64
01:16:00.559384 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 59888, seq 33, length 64
```

In these packet captures, you're seeing broadcast requests from `host1` at `10.0.1.1` to
the broadcast address for this subnet at `10.0.1.255`. In this case, there are two key
things to notice:

1. Each packet is an ICMP echo request, with no ICMP echo reply being returned
2. Within each host's console, the `seq` count matches, showing that every host is
receiving the broadcasts simultaneously

Return to `host1` and cancel the broadcast pings with `Ctrl`+`C`, and look at the
results:

```
WARNING: pinging broadcast address
PING 10.0.1.255 (10.0.1.255) 56(84) bytes of data.
^C
--- 10.0.1.255 ping statistics ---
33 packets transmitted, 0 received, 100% packet loss, time 32757ms
```

In this case, 33 packets (here in my case, for example) were broadcasted, but
`host1` believes none were received resulting in 100% packet loss. However, as we
saw using `tcpdump`, the packets were delivered, but no protocol, such as ARP, was
targeted for a reply.

!!! note

    There are times where hosts may reply to broadcast pings using approaches such
    as `nmap --script broadcast-ping`, but it is not a reliable method depending on
    the host configuration.

Now that we've experimented with broadcasts a bit, let's consider the broadcast
domain itself. So far, we've observed that all hosts in the subnet can be reached
via unicast and broadcast communication. Does this mean the subnet and broadcast
domain are contiguous? Let's test this by making a change.

From Proxmox's UI, change `host2`'s IPv4 address to `10.0.2.2/24`. By doing this,
we create a second subnet within the same bridge `vmbr1`. Next, calculate
`host2`'s subnet details using the `ipcalc` tool:

```
ipcalc 10.0.2.2/24
```

and we can see from the results that `host2` is now in a completely different
subnet:

```
Address:   10.0.2.2             00001010.00000000.00000010. 00000010
Netmask:   255.255.255.0 = 24   11111111.11111111.11111111. 00000000
Wildcard:  0.0.0.255            00000000.00000000.00000000. 11111111
=>
Network:   10.0.2.0/24          00001010.00000000.00000010. 00000000
HostMin:   10.0.2.1             00001010.00000000.00000010. 00000001
HostMax:   10.0.2.254           00001010.00000000.00000010. 11111110
Broadcast: 10.0.2.255           00001010.00000000.00000010. 11111111
Hosts/Net: 254                   Class A, Private Internet
```

* The Network is now `10.0.2.0/24`, not `10.0.1.0/24`
* The HostMin is now `10.0.2.1` and the HostMax is now `10.0.2.254`
* The Broadcast is now `10.0.2.255`

Going back to `host1`, try to ping `host2`:

```
sudo ping 10.0.2.2
```

Doing this will issue an error:

```
ping: connect: Network is unreachable
```

Try the same from `host2`, and try to ping `host1`:

```
sudo ping 10.0.1.1
```

Again, the other subnet is unreachable:

```
ping: connect: Network is unreachable
```

It appears both subnets are isolated from each other, and for subnets on the
same switch, this is generally what you want. But are they? Let's try a
broadcast from `host1` to its subnet's broadcast address. First start the
broadcasting from `host1:`

```
sudo ping -b 10.0.1.255
```

Next, start a packet capture on `host2`:

```
sudo tcpdump
```

Did you end up with something similar to the output displayed below? What are you
now seeing on `host2`? ICMP requests from `host1` on the other network? Apparently
broadcast domains are larger in scope than subnets.

```
18:05:12.216806 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 2358, seq 1, length 64
18:05:13.231371 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 2358, seq 2, length 64
18:05:14.255380 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 2358, seq 3, length 64
18:05:15.279377 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 2358, seq 4, length 64
18:05:16.303372 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 2358, seq 5, length 64
18:05:17.327376 IP 10.0.1.1 > 10.0.1.255: ICMP echo request, id 2358, seq 6, length 64
```

It appears that hosts on different subnets cannot directly communicate with each
other at Layer 3, but communication is still possible at Layer 2. This is because
there is a difference between a Layer 3 broadcast address and a Layer 2 broadcast
domain.

These are the next questions we need to explore with our labs:

1. How would we prevent communication between subnets at Layer 2
2. How would we allow communication between subnets at Layer 3

Before we wrap up this lab, let's consider the role of bridging in this situation.

## Step 3: Dipping Our Toes into Bridging Details

Recall the differences between hubs and switches. Hubs broadcast all frames out
to all connected hosts, and not those destined for the broadcast address. All
connected hosts are also in the same collision domain, causing packets to collide
when hosts attempt to communicate at the same time.

Switches, however, can enable unicast communication between hosts while limiting
broadcasts to all hosts. Each port on a switch is a separate collision domain, as
well, preventing packet transmissions among hosts from interfering with each
other.

What about our Linux bridge? In an earlier lab I stated bridges are basically
switches, especially in eliminating the impact of collision domains, but they
share other characteristics, too. Let's make some more changes to our network
to experiment with the impact of bridges on subnets and broadcast domains.

The first thing we need to do is create another Linux bridge on our Proxmox
hypervisor:

1. At the top of the Proxmox resource tree, select the Proxmox node your lab is
running within and then select the `System` > `Network` view of the content panel.
2. At the top-left of the network device table, click the `Create` dropdown button
and select `Linux Bridge`.
3. In the dialog box, ensure the Name is `vmbr2` and Autostart is checked, then
click the Create button.
4. At the top of the network device table, click the Apply Configuration button
and the new bridge will be enabled.

Next, edit `host2`s network configuration in Proxmox by setting the `eth0`
network device's bridge to `vmbr2`. Be sure to keep the IPv4 address the
same, however.

Now, again start pinging from `host1` to its broadcast address:

```
sudo ping -b 10.0.1.255
```

Then start another packet capture on `host2`:

```
sudo tcpdump
```

What happened this time? Let's look at the results of `host1`'s and `host2`'s
output together:

```
WARNING: pinging broadcast address
PING 10.0.1.255 (10.0.1.255) 56(84) bytes of data.
^C
--- 10.0.1.255 ping statistics ---
145 packets transmitted, 0 received, 100% packet loss, time 147437ms
```

```
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
^C
0 packets captured
0 packets received by filter
0 packets dropped by kernel
```

In this case, `host1` transmitted 145 packets, but `host2` captured none of
them. Now Layer 2 communication is blocked by creating a second bridge, thus
creating two broadcast domains. Bridging seems key to answering the question
about preventing communication between subnets at Layer 2.

Before we wrap up, let's perform a larger test of both unicast and broadcast
communication. Like `host2`, change `host4`'s Bridge to `vmbr2`, but keep
its IPv4 address `10.0.1.4/24`. Then arrange the console windows for all four
hosts in a quarter tile layout so they're all visible.

Start a packet capture again on `host2`:

```
sudo tcpdump
```

and start a ping test on `host4` to that subnet's broadcast address:

```
sudo ping -b -c 4 10.0.1.255
```

Next, start a packet capture on `host1`:

```
sudo tcpdump
```

and start a ping test on `host3`:

```
sudo ping -b -c 4 10.0.1.255
```

Let's review the results for each host, grouped by bridge.

`host3` sent four ICMP echo requests, which were detected (but not replied to)
by `host1` within bridge `vmbr1`:

```
eron@host3:~$ sudo ping -b -c 4 10.0.1.255
WARNING: pinging broadcast address
PING 10.0.1.255 (10.0.1.255) 56(84) bytes of data.

--- 10.0.1.255 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3072ms
```

```
eron@host1:~$ sudo tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
11:23:16.591449 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 54008, seq 1, length 64
11:23:17.615374 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 54008, seq 2, length 64
11:23:18.639371 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 54008, seq 3, length 64
11:23:19.663375 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 54008, seq 4, length 64
^C
4 packets captured
4 packets received by filter
0 packets dropped by kernel
```

Just as above, `host4` sent four ICMP echo requests, which were detected (but not
replied to) by `host2` within bridge `vmbr2`:

```
eron@host4:~$ sudo ping -b -c 4 10.0.1.255
WARNING: pinging broadcast address
PING 10.0.1.255 (10.0.1.255) 56(84) bytes of data.

--- 10.0.1.255 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3064ms
```

```
eron@host2:~$ sudo tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
11:22:47.191030 IP 10.0.1.4 > 10.0.1.255: ICMP echo request, id 27961, seq 1, length 64
11:22:48.207371 IP 10.0.1.4 > 10.0.1.255: ICMP echo request, id 27961, seq 2, length 64
11:22:49.231376 IP 10.0.1.4 > 10.0.1.255: ICMP echo request, id 27961, seq 3, length 64
11:22:50.255377 IP 10.0.1.4 > 10.0.1.255: ICMP echo request, id 27961, seq 4, length 64
^C
4 packets captured
4 packets received by filter
0 packets dropped by kernel
```

Within bridge `vmbr1`, the two hosts were in the same subnet, and the broadcasts
were captured as expected. Within bridge `vmbr2`, the two hosts were not in the
same subnet, but the broadcasts were still captured.

Remember that this was also expected, because regardless of the subnet, broadcasts
are being transmitted across the entire Layer 2 broadcast domain, which right now
is the entire bridge.

So we see that bridges can isolate broadcast domains, but only when there is one
subnet per bridge. But that essentially means having to restrict an entire switch
to only one subnet, which is unrealistic in nearly any network with multiple
subnets. In a physical network, this would require a switch for each subnet.

As our final experiment, let's move `host2` and `host4` back to `vmbr1`, and
change `host4`'s IPv4 address to `10.0.2.4/24`, so it's in the same subnet as
`host2`. With those configuration changes made, we now how two subnets, each with
two hosts, on one bridge. What happens when they start communicating?

Start a packet capture on `host1`:

```
sudo tcpdump
```

and start a ping test to `10.0.1.0/24` network's broadcast address from `host3`:

```
sudo ping -b -c 4 10.0.1.255
```

Next start a packet capture on `host2`:

```
sudo tcpdump
```

and start a ping test to `10.0.2.0/24` network's broadcast address from `host4`:

```
sudo ping -b -c 4 10.0.2.255
```

Again we'll review the results for each host, grouped by bridge.

`host3` sent four ICMP echo requests, which were detected (but not replied to)
by `host1` within bridge `vmbr1`:

```
eron@host3:~$ sudo ping -b -c 4 10.0.1.255
WARNING: pinging broadcast address
PING 10.0.1.255 (10.0.1.255) 56(84) bytes of data.

--- 10.0.1.255 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3065ms
```

```
eron@host1:~$ sudo tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
15:07:12.790897 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 57207, seq 1, length 64
15:07:13.807380 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 57207, seq 2, length 64
15:07:14.831382 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 57207, seq 3, length 64
15:07:15.855384 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 57207, seq 4, length 64
15:07:44.485779 IP 10.0.2.4 > 10.0.2.255: ICMP echo request, id 7169, seq 1, length 64
15:07:45.487366 IP 10.0.2.4 > 10.0.2.255: ICMP echo request, id 7169, seq 2, length 64
15:07:46.511402 IP 10.0.2.4 > 10.0.2.255: ICMP echo request, id 7169, seq 3, length 64
15:07:47.535375 IP 10.0.2.4 > 10.0.2.255: ICMP echo request, id 7169, seq 4, length 64
^C
8 packets captured
8 packets received by filter
0 packets dropped by kernel
```

But wait, there are four broadcast packets from `host4` here as well! All `host4`
did was send four ICMP echo requests on its own subnet, which were detected (but
not replied to) by `host2` within bridge `vmbr1`:

```
eron@host4:~$ sudo ping -b -c 4 10.0.2.255
WARNING: pinging broadcast address
PING 10.0.2.255 (10.0.2.255) 56(84) bytes of data.

--- 10.0.2.255 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3050ms
```

```
eron@host2:~$ sudo tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
15:07:12.790892 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 57207, seq 1, length 64
15:07:13.807374 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 57207, seq 2, length 64
15:07:14.831376 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 57207, seq 3, length 64
15:07:15.855379 IP 10.0.1.3 > 10.0.1.255: ICMP echo request, id 57207, seq 4, length 64
15:07:44.485776 IP 10.0.2.4 > 10.0.2.255: ICMP echo request, id 7169, seq 1, length 64
15:07:45.487364 IP 10.0.2.4 > 10.0.2.255: ICMP echo request, id 7169, seq 2, length 64
15:07:46.511399 IP 10.0.2.4 > 10.0.2.255: ICMP echo request, id 7169, seq 3, length 64
15:07:47.535372 IP 10.0.2.4 > 10.0.2.255: ICMP echo request, id 7169, seq 4, length 64
^C
8 packets captured
8 packets received by filter
0 packets dropped by kernel
```

The packet capture from `host2` shows packets from `host4` along with those from
`host3` as well! So now we really see the problem: how can we restrict broadcast
domains within the same bridge or switch containing multiple subnets?

We'll figure that out in the next lab, [Exploring Subnets and VLANs in Proxmox](exploring-subnets-and-vlans-in-proxmox.md).
Great work today; I know this was a long lab, but hopefully these concepts are
starting to make sense.
