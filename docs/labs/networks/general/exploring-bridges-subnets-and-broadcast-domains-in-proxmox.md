Based on our progress in the last lab, [Connecting and Configuring Network Hosts in Proxmox](connecting-and-configuring-network-hosts-in-proxmox.md),
we know that our four hosts are somehow able to communicate within their subnet.
But how exactly are the hosts connecting?

Recall that each host is configured with a network device connected to the `vmbr1`
bridge and assigned an IPv4 address in the `10.0.1.0/24` subnet. These two factors are the key to understanding what is happening behind the scenes.

In this lab, we're going to begin exploring what happens between the bridge, subnet,
and broadcasting domain to allow our hosts to communicate. Along the way, we'll see
how these fundamental concepts lay the foundation for us to build increasingly
complex labs.

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

As we saw with the ARP process, the bridge plays a central role in enabling hosts to
communicate. Let's move on and learn more about bridging, which involves many of
the same concepts in hardware switching.

## Step 2: Dipping Our Toes into Bridging Details



## Step 3: Digging into Subnets and Broadcast Domains

Must be on the same bridge
Must be on the same subnet