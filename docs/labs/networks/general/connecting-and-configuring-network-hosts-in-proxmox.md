---
tags:
  - Proxmox
  - Linux
---

# Connecting and Configuring Network Hosts in Proxmox

Imagine having two physical Linux servers, each with a single network adapter.
If you interconnect the two servers with a patch cord, and configure the network
adapters so they can communicate, what will you have?

A simple, but fully-functional network. And that's where you'll begin your
learning on building networks, which is all about connecting hosts together.

In this lab, we're going to directly connect our network hosts together virtually
by adding them to a Linux bridge and configuring them with basic network settings.
No additional routers, switches, or firewalls will be necessary at this point;
just the built-in functionality of Linux and Proxmox.

## Step 1: Create a Linux Bridge for Lab Connectivity

A [Linux bridge](https://developers.redhat.com/articles/2022/04/06/introduction-linux-bridging-commands-and-features)
is like a basic switch integrated into the operating system that
supports things like STP and VLAN trunking, and allows virtual machines and containers to communicate within hypervisors, like Proxmox.

When installing Proxmox, a Linux bridge named `vmbr0` will be created to connect the
hypervisor to the outside world. In my own lab setup, I use `vmbr0` to connect the
Proxmox cluster and all VMs and CTs to my management subnet. By default, your CTs
and Proxmox itself are running on that bridge.

For our labs at this point, however, we have no need for our CTs to reach outside
the lab subnet we'll be creating, so we'll create a second bridge just for our hosts
to connect and communicate across. This provides a self-contained Layer 2 broadcast
domain we can completely control, just like having a dedicated hardware switch with
only your lab links connected.

1. At the top of the Proxmox resource tree, select the Proxmox node your lab is
running within and then select the `System` > `Network` view of the content panel.
2. At the top-left of the network device table, click the `Create` dropdown button
and select `Linux Bridge`.
3. In the dialog box, ensure the Name is `vmbr1` and Autostart is checked, then
click the Create button.
4. At the top of the network device table, click the Apply Configuration button
and the new bridge will be enabled.

## Step 2: Connecting Hosts to the Lab Bridge

With our lab bridge created, we'll need to now change the bridge setting on each
host's interface from `vmbr0` to `vmbr1`. Think of this like moving a host's patch
cord from one hardware switch to another.

We'll also be changing each host's MAC address on its interface to match up with
the interfaces used in our labs. This helps to ensure your results are consistent
with the lab instructions. Use the table below for the MAC address assignments:

| Host | MAC Address |
| ---- | ----------- |
| host1 | 00:50:56:94:55:70 |
| host2 | 00:50:56:AD:0E:33 |
| host3 | 00:50:56:16:30:C7 |
| host4 | 00:50:56:AD:24:4A |

For each host, perform the following:

1. From the Proxmox Web UI, select the CT in the resource tree and select the
Network content panel.
2. Select the network device

## Step 3: Configuring Hosts to Communicate Across the Bridge

With our hosts all added to the same bridge, which by default is also the same
broadcast domain, they will be able to communicate once they obtain IP address
configurations.

For this lab, we will be using the `10.0.1.0/24` IPv4 subnet for the lab network.
Because our network devices are set to static addresses, we'll have to manually
assign the right address to the right host. Use the table below for the IP address assignments:

| Host | IPv4 Address |
| ---- | ------------ |
| host1 | 10.0.1.1/24 |
| host2 | 10.0.1.2/24 |
| host3 | 10.0.1.3/24 |
| host4 | 10.0.1.4/24 |

Within each host's `Network` settings in Proxmox, configure the `eth0` network
device's IPv4 setting to the respective address in the table above and verify
the correct assigned address has been configured:

```
sudo ip addr show eth0
```

## Step 3: Confirm Host Communication Across the Bridge

At this point, all hosts should be connected and reachable, or "up." As a best
practice, always verify this.

The most common method is to use the `ping` command, so let's use `host1` to test
connectivity to the other three hosts. From `host1`, check to see if `host2` is
available:

```
sudo ping -c4 10.0.1.2
```

Our `ping` response should be:

```
PING 10.0.1.2 (10.0.1.2) 56(84) bytes of data.
64 bytes from 10.0.1.2: icmp_seq=1 ttl=64 time=0.069 ms
64 bytes from 10.0.1.2: icmp_seq=2 ttl=64 time=0.035 ms
64 bytes from 10.0.1.2: icmp_seq=3 ttl=64 time=0.035 ms
64 bytes from 10.0.1.2: icmp_seq=4 ttl=64 time=0.036 ms

--- 10.0.1.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3071ms
rtt min/avg/max/mdev = 0.035/0.043/0.069/0.014 ms
```

Again from `host1`, check to see if `host3` is available:

```
sudo ping -c4 10.0.1.3
```

Our `ping` response should be:

```
PING 10.0.1.3 (10.0.1.3) 56(84) bytes of data.
64 bytes from 10.0.1.3: icmp_seq=1 ttl=64 time=0.118 ms
64 bytes from 10.0.1.3: icmp_seq=2 ttl=64 time=0.038 ms
64 bytes from 10.0.1.3: icmp_seq=3 ttl=64 time=0.037 ms
64 bytes from 10.0.1.3: icmp_seq=4 ttl=64 time=0.036 ms

--- 10.0.1.3 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3077ms
rtt min/avg/max/mdev = 0.036/0.057/0.118/0.035 ms
```

Again from `host1`, check to see if `host4` is available:

```
sudo ping -c4 10.0.1.4
```

Our `ping` response should be:

```
PING 10.0.1.4 (10.0.1.4) 56(84) bytes of data.
64 bytes from 10.0.1.4: icmp_seq=1 ttl=64 time=0.099 ms
64 bytes from 10.0.1.4: icmp_seq=2 ttl=64 time=0.029 ms
64 bytes from 10.0.1.4: icmp_seq=3 ttl=64 time=0.034 ms
64 bytes from 10.0.1.4: icmp_seq=4 ttl=64 time=0.026 ms

--- 10.0.1.4 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3068ms
rtt min/avg/max/mdev = 0.026/0.047/0.099/0.030 ms
```

While `ping` is an essential tool for network testing, when you are scanning
subnets or looking for more information from hosts (such as what ports are
open), use `nmap`. Let's test host connectivity again from `host1`, using
`nmap` this time:

```
nmap -sn 10.0.1.0/24
```

Our network scan should be:

```
Starting Nmap 7.80 ( https://nmap.org ) at 2023-06-21 11:15 UTC
Nmap scan report for 10.0.1.1
Host is up (0.00034s latency).
Nmap scan report for 10.0.1.2
Host is up (0.00031s latency).
Nmap scan report for 10.0.1.3
Host is up (0.00021s latency).
Nmap scan report for 10.0.1.4
Host is up (0.00016s latency).
Nmap done: 256 IP addresses (4 hosts up) scanned in 15.91 seconds
```

With one command, we can tell `nmap` to scan the entire `/24` subnet and report back
the status of every host, including `host1`.

!!! question

    Practice performing `ping` and `nmap` tests between the four hosts to get
    comfortable with these essential tools. Which approach for network discovery
    do you think is better?

Well done! We have built an entire network of Linux hosts that can communicate with
each other. In the next lab,
[Exploring Bridges, Subnets, and Broadcast Domains in Proxmox](exploring-subnets-broadcast-domains-and-bridges-in-proxmox.md),
we will begin exploring our network to better understand how everything works
together.
