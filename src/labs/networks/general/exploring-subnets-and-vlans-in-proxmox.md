---
tags:
  - Proxmox
  - Linux
---

# Exploring Subnets and VLANs in Proxmox

## Introduction

In this lab we're going to continue exploring subnets and begin to look at the
role of VLANs in addressing the broadcast domain issue we observed in the
previous lab. Recall from that lab that we ended up with two subnets on the same
bridge and shared the same broadcast domain.

VLANs are a way to logically separate broadcast domains within the same Layer 2
network, such as a bridge or switch. This allows us to reduce broadcasts and
flooding, which improves network performance. VLANs also:

* **increase network security** by segmenting Layer 2 networks and enabling
fine-grained control of inter-VLAN communication between hosts via routing and
firewalls,
* **provide management flexibility** in organizing and controlling traffic based on
each host or subnet's role, functions, or priority (quality of service [QoS]) and
* **maximize hardware utilization** by enabling multiple VLANs to either exist on
the same bridge or switch as well as connecting across multiple bridges or
switches.

In this lab we're going to concentrate on segmenting broadcast domains using VLANs,
but we'll be addressing the other applications of VLANs in future labs.

## Step 1: Configure the Bridge to Be VLAN Aware

Before we begin experimenting with VLANS, we have to make a change to bridge
`vmbr1` in order to allow VLANs on the hosts. On the `Networks` view of the
Proxmox node, edit `vmbr1` and check "VLAN aware." Then click the `Apply
Configuration` button and you're ready to use VLANs on the bridge.

!!! warning

    Ensure this configuration is performed before attempting to apply VLAN
    settings to hosts on a bridge. Otherwise, you'll get errors.

    The same is true for attempting to disable VLAN settings on a bridge. In
    this case, if hosts are configured with VLANs, and you try to uncheck "VLAN
    aware" on the bridge, you will get errors here, as well.

There are other VLAN configuration options within Proxmox, but this is simplest
and most consistent way to configure them at this point. We'll dig into advanced
Proxmox networking in future labs.

## Step 2: Assign Subnets to VLANs

With the bridge now VLAN aware, it's time to assign VLANS to each subnet in order
to separate each subnet's broadcast domain. To do this, we'll use a common VLAN
to subnet numbering convention that is helpful while learning these concepts.

!!! tip

    Using one VLAN per subnet is the recommended best practice.

We're going to create two new subnets: `10.0.10.0/24` and `10.0.20.0/24`, and
assign VLAN `10` to the first subnet and VLAN `20` to the second subnet. Using this
convention, the third octet of the subnet matches the VLAN ID.

!!! warning

    This numbering convention is helpful for learning purposes, but will often
    break down or not scale in production networks, and also should not be used
    as a substitute for good documentation.

    We'll cover both network address planning and documentation in future labs.

`host1` and `host2` are going to be assigned to the `10.0.10.0/24` subnet, and
`host3` and `host4` are going to be assigned to the `10.0.20.0/24` subnet. Here's
the new addressing plan for our four hosts:

| Host | IPv4 Address | VLAN |
| ---- | ------------ | ---- |
| host1 | 10.0.10.1/24 | 10 |
| host2 | 10.0.10.2/24 | 10 |
| host3 | 10.0.20.3/24 | 20 |
| host4 | 10.0.20.4/24 | 20 |

Make these changes by updating the IPv4 address and VLAN ID settings within the
network configurations for each host in Proxmox. Verify each host has the right
IPv4 address by running an `ip` command that displays host addressing and
reviewing the output:

```
ip a
```

## Step 3: Verify Broadcast Domain Isolation for Each Subnet

Once all configuration changes are made, we'll be ready to verify that the VLANs
now isolate the two broadcast domains, as expected. To test, we're going to use
`nmap` again, but this time, we'll also see how it works behind the scenes so you
understand why it's a good choice for this test.

Open up all four hosts in the quarter tile layout so they're all visible, and
begin packet captures on `host2` and `host4`:

```
sudo tcpdump
```

Next, we're going to perform a quick scan of each subnet and then examine the
results. Starting on `host1`, run the following on `host1`:

```
nmap -sn 10.0.10.0/24
```

Then check the output on both `host1` and `host2`

```
eron@host1:~$ nmap -sn 10.0.10.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2023-06-27 13:54 UTC
Nmap scan report for 10.0.10.1
Host is up (0.00033s latency).
Nmap scan report for 10.0.10.2
Host is up (0.00023s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 15.91 seconds
```

```
eron@host2:~$ sudo tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
13:54:02.238931 IP 10.0.10.1.54440 > 10.0.10.2.http: Flags [S], seq 3772918725, win 64240, options [mss 1460,sackOK,TS val 3851871978 ecr 0,nop,wscale 7], length 0
13:54:02.238938 IP 10.0.10.2.http > 10.0.10.1.54440: Flags [R.], seq 0, ack 3772918726, win 0, length 0
13:54:02.238976 ARP, Request who-has 10.0.10.3 tell 10.0.10.1, length 28
13:54:02.238996 ARP, Request who-has 10.0.10.4 tell 10.0.10.1, length 28
13:54:02.239034 ARP, Request who-has 10.0.10.5 tell 10.0.10.1, length 28
13:54:02.239061 ARP, Request who-has 10.0.10.6 tell 10.0.10.1, length 28
13:54:02.239079 ARP, Request who-has 10.0.10.7 tell 10.0.10.1, length 28
13:54:02.239114 ARP, Request who-has 10.0.10.8 tell 10.0.10.1, length 28
13:54:02.239140 ARP, Request who-has 10.0.10.9 tell 10.0.10.1, length 28
13:54:02.239174 ARP, Request who-has 10.0.10.10 tell 10.0.10.1, length 28
13:54:02.239278 ARP, Request who-has 10.0.10.13 tell 10.0.10.1, length 28
13:54:02.239317 ARP, Request who-has 10.0.10.14 tell 10.0.10.1, length 28
13:54:02.239343 ARP, Request who-has 10.0.10.15 tell 10.0.10.1, length 28
13:54:02.239359 ARP, Request who-has 10.0.10.16 tell 10.0.10.1, length 28
13:54:02.339041 ARP, Request who-has 10.0.10.65 tell 10.0.10.1, length 28
13:54:02.339115 ARP, Request who-has 10.0.10.68 tell 10.0.10.1, length 28
13:54:02.339142 ARP, Request who-has 10.0.10.69 tell 10.0.10.1, length 28
13:54:02.339160 ARP, Request who-has 10.0.10.70 tell 10.0.10.1, length 28
13:54:02.339230 ARP, Request who-has 10.0.10.73 tell 10.0.10.1, length 28
13:54:02.339249 ARP, Request who-has 10.0.10.74 tell 10.0.10.1, length 28
13:54:02.339267 ARP, Request who-has 10.0.10.75 tell 10.0.10.1, length 28
13:54:02.339284 ARP, Request who-has 10.0.10.76 tell 10.0.10.1, length 28
13:54:02.339345 ARP, Request who-has 10.0.10.79 tell 10.0.10.1, length 28
13:54:02.339362 ARP, Request who-has 10.0.10.80 tell 10.0.10.1, length 28
13:54:02.339424 ARP, Request who-has 10.0.10.83 tell 10.0.10.1, length 28
13:54:02.339441 ARP, Request who-has 10.0.10.84 tell 10.0.10.1, length 28
13:54:02.439092 ARP, Request who-has 10.0.10.131 tell 10.0.10.1, length 28
13:54:02.439157 ARP, Request who-has 10.0.10.134 tell 10.0.10.1, length 28
13:54:02.439220 ARP, Request who-has 10.0.10.137 tell 10.0.10.1, length 28
13:54:02.439238 ARP, Request who-has 10.0.10.138 tell 10.0.10.1, length 28
13:54:02.439299 ARP, Request who-has 10.0.10.141 tell 10.0.10.1, length 28
13:54:02.439317 ARP, Request who-has 10.0.10.142 tell 10.0.10.1, length 28
13:54:02.439378 ARP, Request who-has 10.0.10.145 tell 10.0.10.1, length 28
13:54:02.439400 ARP, Request who-has 10.0.10.146 tell 10.0.10.1, length 28
13:54:02.439464 ARP, Request who-has 10.0.10.149 tell 10.0.10.1, length 28
13:54:02.439483 ARP, Request who-has 10.0.10.150 tell 10.0.10.1, length 28
13:54:02.439500 ARP, Request who-has 10.0.10.151 tell 10.0.10.1, length 28
13:54:02.439558 ARP, Request who-has 10.0.10.154 tell 10.0.10.1, length 28
13:54:02.539133 ARP, Request who-has 10.0.10.193 tell 10.0.10.1, length 28
13:54:02.539197 ARP, Request who-has 10.0.10.196 tell 10.0.10.1, length 28
13:54:02.539257 ARP, Request who-has 10.0.10.199 tell 10.0.10.1, length 28
13:54:02.539315 ARP, Request who-has 10.0.10.202 tell 10.0.10.1, length 28
13:54:02.539376 ARP, Request who-has 10.0.10.205 tell 10.0.10.1, length 28
13:54:02.539395 ARP, Request who-has 10.0.10.206 tell 10.0.10.1, length 28
13:54:02.539455 ARP, Request who-has 10.0.10.209 tell 10.0.10.1, length 28
13:54:02.539474 ARP, Request who-has 10.0.10.210 tell 10.0.10.1, length 28
13:54:02.539547 ARP, Request who-has 10.0.10.213 tell 10.0.10.1, length 28
13:54:02.539565 ARP, Request who-has 10.0.10.214 tell 10.0.10.1, length 28
13:54:02.539631 ARP, Request who-has 10.0.10.217 tell 10.0.10.1, length 28
13:54:02.539651 ARP, Request who-has 10.0.10.218 tell 10.0.10.1, length 28
13:54:02.639187 ARP, Request who-has 10.0.10.253 tell 10.0.10.1, length 28
13:54:02.639252 ARP, Request who-has 10.0.10.0 tell 10.0.10.1, length 28
13:54:02.639311 ARP, Request who-has 10.0.10.11 tell 10.0.10.1, length 28
13:54:02.639369 ARP, Request who-has 10.0.10.17 tell 10.0.10.1, length 28
13:54:02.639426 ARP, Request who-has 10.0.10.20 tell 10.0.10.1, length 28
13:54:02.639483 ARP, Request who-has 10.0.10.23 tell 10.0.10.1, length 28
13:54:02.639544 ARP, Request who-has 10.0.10.26 tell 10.0.10.1, length 28
13:54:02.639562 ARP, Request who-has 10.0.10.27 tell 10.0.10.1, length 28
13:54:02.639622 ARP, Request who-has 10.0.10.30 tell 10.0.10.1, length 28
13:54:02.639640 ARP, Request who-has 10.0.10.31 tell 10.0.10.1, length 28
13:54:02.639700 ARP, Request who-has 10.0.10.34 tell 10.0.10.1, length 28
13:54:02.639718 ARP, Request who-has 10.0.10.35 tell 10.0.10.1, length 28
13:54:02.739241 ARP, Request who-has 10.0.10.71 tell 10.0.10.1, length 28
13:54:02.739306 ARP, Request who-has 10.0.10.77 tell 10.0.10.1, length 28
13:54:02.739367 ARP, Request who-has 10.0.10.81 tell 10.0.10.1, length 28
13:54:02.739429 ARP, Request who-has 10.0.10.85 tell 10.0.10.1, length 28
13:54:02.739490 ARP, Request who-has 10.0.10.88 tell 10.0.10.1, length 28
13:54:02.739549 ARP, Request who-has 10.0.10.91 tell 10.0.10.1, length 28
13:54:02.739609 ARP, Request who-has 10.0.10.94 tell 10.0.10.1, length 28
13:54:02.739626 ARP, Request who-has 10.0.10.95 tell 10.0.10.1, length 28
13:54:02.739688 ARP, Request who-has 10.0.10.98 tell 10.0.10.1, length 28
13:54:02.739705 ARP, Request who-has 10.0.10.99 tell 10.0.10.1, length 28
13:54:02.739771 ARP, Request who-has 10.0.10.102 tell 10.0.10.1, length 28
13:54:02.739793 ARP, Request who-has 10.0.10.103 tell 10.0.10.1, length 28
13:54:02.839291 ARP, Request who-has 10.0.10.132 tell 10.0.10.1, length 28
13:54:02.839359 ARP, Request who-has 10.0.10.135 tell 10.0.10.1, length 28
13:54:02.839420 ARP, Request who-has 10.0.10.139 tell 10.0.10.1, length 28
13:54:02.839481 ARP, Request who-has 10.0.10.143 tell 10.0.10.1, length 28
13:54:02.839540 ARP, Request who-has 10.0.10.147 tell 10.0.10.1, length 28
13:54:02.839599 ARP, Request who-has 10.0.10.152 tell 10.0.10.1, length 28
13:54:02.839660 ARP, Request who-has 10.0.10.155 tell 10.0.10.1, length 28
13:54:02.839724 ARP, Request who-has 10.0.10.158 tell 10.0.10.1, length 28
13:54:02.839743 ARP, Request who-has 10.0.10.159 tell 10.0.10.1, length 28
13:54:02.839802 ARP, Request who-has 10.0.10.162 tell 10.0.10.1, length 28
13:54:02.839862 ARP, Request who-has 10.0.10.165 tell 10.0.10.1, length 28
13:54:02.839880 ARP, Request who-has 10.0.10.166 tell 10.0.10.1, length 28
13:54:02.939327 ARP, Request who-has 10.0.10.191 tell 10.0.10.1, length 28
13:54:02.939393 ARP, Request who-has 10.0.10.194 tell 10.0.10.1, length 28
13:54:02.939454 ARP, Request who-has 10.0.10.197 tell 10.0.10.1, length 28
13:54:02.939513 ARP, Request who-has 10.0.10.200 tell 10.0.10.1, length 28
13:54:02.939591 ARP, Request who-has 10.0.10.207 tell 10.0.10.1, length 28
13:54:02.939649 ARP, Request who-has 10.0.10.211 tell 10.0.10.1, length 28
13:54:02.939708 ARP, Request who-has 10.0.10.215 tell 10.0.10.1, length 28
13:54:02.939767 ARP, Request who-has 10.0.10.219 tell 10.0.10.1, length 28
13:54:02.939825 ARP, Request who-has 10.0.10.222 tell 10.0.10.1, length 28
13:54:02.939884 ARP, Request who-has 10.0.10.225 tell 10.0.10.1, length 28
13:54:02.939946 ARP, Request who-has 10.0.10.228 tell 10.0.10.1, length 28
13:54:02.939966 ARP, Request who-has 10.0.10.229 tell 10.0.10.1, length 28
13:54:03.040226 ARP, Request who-has 10.0.10.232 tell 10.0.10.1, length 28
13:54:03.040248 ARP, Request who-has 10.0.10.233 tell 10.0.10.1, length 28
13:54:03.040266 ARP, Request who-has 10.0.10.234 tell 10.0.10.1, length 28
13:54:03.040292 ARP, Request who-has 10.0.10.235 tell 10.0.10.1, length 28
13:54:03.040309 ARP, Request who-has 10.0.10.236 tell 10.0.10.1, length 28
13:54:03.040327 ARP, Request who-has 10.0.10.237 tell 10.0.10.1, length 28
13:54:03.040345 ARP, Request who-has 10.0.10.238 tell 10.0.10.1, length 28
13:54:03.040362 ARP, Request who-has 10.0.10.239 tell 10.0.10.1, length 28
13:54:03.040378 ARP, Request who-has 10.0.10.240 tell 10.0.10.1, length 28
13:54:03.040395 ARP, Request who-has 10.0.10.241 tell 10.0.10.1, length 28
13:54:03.040413 ARP, Request who-has 10.0.10.242 tell 10.0.10.1, length 28
13:54:03.040430 ARP, Request who-has 10.0.10.243 tell 10.0.10.1, length 28
13:54:03.140269 ARP, Request who-has 10.0.10.60 tell 10.0.10.1, length 28
13:54:03.140341 ARP, Request who-has 10.0.10.63 tell 10.0.10.1, length 28
13:54:03.140361 ARP, Request who-has 10.0.10.64 tell 10.0.10.1, length 28
13:54:03.140382 ARP, Request who-has 10.0.10.66 tell 10.0.10.1, length 28
13:54:03.140456 ARP, Request who-has 10.0.10.72 tell 10.0.10.1, length 28
13:54:03.140475 ARP, Request who-has 10.0.10.78 tell 10.0.10.1, length 28
13:54:03.140492 ARP, Request who-has 10.0.10.82 tell 10.0.10.1, length 28
13:54:03.140509 ARP, Request who-has 10.0.10.86 tell 10.0.10.1, length 28
13:54:03.140526 ARP, Request who-has 10.0.10.87 tell 10.0.10.1, length 28
13:54:03.140543 ARP, Request who-has 10.0.10.89 tell 10.0.10.1, length 28
13:54:03.140607 ARP, Request who-has 10.0.10.92 tell 10.0.10.1, length 28
13:54:03.140630 ARP, Request who-has 10.0.10.93 tell 10.0.10.1, length 28
13:54:03.240320 ARP, Request who-has 10.0.10.148 tell 10.0.10.1, length 28
13:54:03.240387 ARP, Request who-has 10.0.10.153 tell 10.0.10.1, length 28
13:54:03.240455 ARP, Request who-has 10.0.10.156 tell 10.0.10.1, length 28
13:54:03.240474 ARP, Request who-has 10.0.10.157 tell 10.0.10.1, length 28
13:54:03.240538 ARP, Request who-has 10.0.10.160 tell 10.0.10.1, length 28
13:54:03.240556 ARP, Request who-has 10.0.10.161 tell 10.0.10.1, length 28
13:54:03.240573 ARP, Request who-has 10.0.10.163 tell 10.0.10.1, length 28
13:54:03.240638 ARP, Request who-has 10.0.10.167 tell 10.0.10.1, length 28
13:54:03.240663 ARP, Request who-has 10.0.10.168 tell 10.0.10.1, length 28
13:54:03.240681 ARP, Request who-has 10.0.10.169 tell 10.0.10.1, length 28
13:54:03.240744 ARP, Request who-has 10.0.10.172 tell 10.0.10.1, length 28
13:54:03.240764 ARP, Request who-has 10.0.10.173 tell 10.0.10.1, length 28
13:54:03.240880 ARP, Request who-has 10.0.10.16 tell 10.0.10.1, length 28
13:54:03.240882 ARP, Request who-has 10.0.10.15 tell 10.0.10.1, length 28
13:54:03.240882 ARP, Request who-has 10.0.10.14 tell 10.0.10.1, length 28
13:54:03.240883 ARP, Request who-has 10.0.10.13 tell 10.0.10.1, length 28
13:54:03.240884 ARP, Request who-has 10.0.10.10 tell 10.0.10.1, length 28
13:54:03.240885 ARP, Request who-has 10.0.10.9 tell 10.0.10.1, length 28
13:54:03.240885 ARP, Request who-has 10.0.10.8 tell 10.0.10.1, length 28
13:54:03.240886 ARP, Request who-has 10.0.10.7 tell 10.0.10.1, length 28
13:54:03.240887 ARP, Request who-has 10.0.10.6 tell 10.0.10.1, length 28
13:54:03.240887 ARP, Request who-has 10.0.10.5 tell 10.0.10.1, length 28
13:54:03.240888 ARP, Request who-has 10.0.10.4 tell 10.0.10.1, length 28
13:54:03.240889 ARP, Request who-has 10.0.10.3 tell 10.0.10.1, length 28
13:54:03.368936 ARP, Request who-has 10.0.10.84 tell 10.0.10.1, length 28
13:54:03.368938 ARP, Request who-has 10.0.10.83 tell 10.0.10.1, length 28
13:54:03.368939 ARP, Request who-has 10.0.10.80 tell 10.0.10.1, length 28
13:54:03.368939 ARP, Request who-has 10.0.10.79 tell 10.0.10.1, length 28
13:54:03.368940 ARP, Request who-has 10.0.10.76 tell 10.0.10.1, length 28
13:54:03.368940 ARP, Request who-has 10.0.10.75 tell 10.0.10.1, length 28
13:54:03.368941 ARP, Request who-has 10.0.10.74 tell 10.0.10.1, length 28
13:54:03.368942 ARP, Request who-has 10.0.10.73 tell 10.0.10.1, length 28
13:54:03.368942 ARP, Request who-has 10.0.10.70 tell 10.0.10.1, length 28
13:54:03.368943 ARP, Request who-has 10.0.10.69 tell 10.0.10.1, length 28
13:54:03.368943 ARP, Request who-has 10.0.10.68 tell 10.0.10.1, length 28
13:54:03.368944 ARP, Request who-has 10.0.10.65 tell 10.0.10.1, length 28
13:54:03.464947 ARP, Request who-has 10.0.10.154 tell 10.0.10.1, length 28
13:54:03.464950 ARP, Request who-has 10.0.10.151 tell 10.0.10.1, length 28
13:54:03.464951 ARP, Request who-has 10.0.10.150 tell 10.0.10.1, length 28
13:54:03.464952 ARP, Request who-has 10.0.10.149 tell 10.0.10.1, length 28
13:54:03.464952 ARP, Request who-has 10.0.10.146 tell 10.0.10.1, length 28
13:54:03.464953 ARP, Request who-has 10.0.10.145 tell 10.0.10.1, length 28
13:54:03.464954 ARP, Request who-has 10.0.10.142 tell 10.0.10.1, length 28
13:54:03.464954 ARP, Request who-has 10.0.10.141 tell 10.0.10.1, length 28
13:54:03.464955 ARP, Request who-has 10.0.10.138 tell 10.0.10.1, length 28
13:54:03.464955 ARP, Request who-has 10.0.10.137 tell 10.0.10.1, length 28
13:54:03.464956 ARP, Request who-has 10.0.10.134 tell 10.0.10.1, length 28
13:54:03.464957 ARP, Request who-has 10.0.10.131 tell 10.0.10.1, length 28
13:54:03.540432 IP 10.0.10.1.54454 > 10.0.10.2.http: Flags [S], seq 1946917605, win 64240, options [mss 1460,sackOK,TS val 3851873279 ecr 0,nop,wscale 7], length 0
13:54:03.540440 IP 10.0.10.2.http > 10.0.10.1.54454: Flags [R.], seq 0, ack 1946917606, win 0, length 0
13:54:03.540670 ARP, Request who-has 10.0.10.96 tell 10.0.10.1, length 28
13:54:03.540691 ARP, Request who-has 10.0.10.97 tell 10.0.10.1, length 28
13:54:03.540709 ARP, Request who-has 10.0.10.100 tell 10.0.10.1, length 28
13:54:03.540727 ARP, Request who-has 10.0.10.101 tell 10.0.10.1, length 28
13:54:03.540747 ARP, Request who-has 10.0.10.104 tell 10.0.10.1, length 28
13:54:03.540765 ARP, Request who-has 10.0.10.105 tell 10.0.10.1, length 28
13:54:03.540782 ARP, Request who-has 10.0.10.106 tell 10.0.10.1, length 28
13:54:03.540808 ARP, Request who-has 10.0.10.107 tell 10.0.10.1, length 28
13:54:03.540842 ARP, Request who-has 10.0.10.108 tell 10.0.10.1, length 28
13:54:03.540860 ARP, Request who-has 10.0.10.109 tell 10.0.10.1, length 28
13:54:03.540877 ARP, Request who-has 10.0.10.110 tell 10.0.10.1, length 28
13:54:03.540896 ARP, Request who-has 10.0.10.111 tell 10.0.10.1, length 28
13:54:03.540913 ARP, Request who-has 10.0.10.112 tell 10.0.10.1, length 28
13:54:03.540932 ARP, Request who-has 10.0.10.113 tell 10.0.10.1, length 28
13:54:03.540950 ARP, Request who-has 10.0.10.114 tell 10.0.10.1, length 28
13:54:03.540970 ARP, Request who-has 10.0.10.115 tell 10.0.10.1, length 28
13:54:03.540988 ARP, Request who-has 10.0.10.116 tell 10.0.10.1, length 28
13:54:03.541005 ARP, Request who-has 10.0.10.117 tell 10.0.10.1, length 28
13:54:03.541023 ARP, Request who-has 10.0.10.118 tell 10.0.10.1, length 28
13:54:03.541040 ARP, Request who-has 10.0.10.119 tell 10.0.10.1, length 28
13:54:03.541058 ARP, Request who-has 10.0.10.120 tell 10.0.10.1, length 28
13:54:03.541079 ARP, Request who-has 10.0.10.121 tell 10.0.10.1, length 28
13:54:03.541096 ARP, Request who-has 10.0.10.122 tell 10.0.10.1, length 28
13:54:03.541114 ARP, Request who-has 10.0.10.123 tell 10.0.10.1, length 28
13:54:03.541133 ARP, Request who-has 10.0.10.124 tell 10.0.10.1, length 28
13:54:03.541153 ARP, Request who-has 10.0.10.125 tell 10.0.10.1, length 28
13:54:03.541170 ARP, Request who-has 10.0.10.126 tell 10.0.10.1, length 28
13:54:03.541189 ARP, Request who-has 10.0.10.127 tell 10.0.10.1, length 28
13:54:03.541207 ARP, Request who-has 10.0.10.128 tell 10.0.10.1, length 28
13:54:03.541229 ARP, Request who-has 10.0.10.129 tell 10.0.10.1, length 28
13:54:03.541248 ARP, Request who-has 10.0.10.130 tell 10.0.10.1, length 28
13:54:03.541273 ARP, Request who-has 10.0.10.133 tell 10.0.10.1, length 28
13:54:03.541298 ARP, Request who-has 10.0.10.136 tell 10.0.10.1, length 28
13:54:03.541337 ARP, Request who-has 10.0.10.140 tell 10.0.10.1, length 28
13:54:03.541441 ARP, Request who-has 10.0.10.144 tell 10.0.10.1, length 28
13:54:03.541514 ARP, Request who-has 10.0.10.164 tell 10.0.10.1, length 28
13:54:03.541530 ARP, Request who-has 10.0.10.170 tell 10.0.10.1, length 28
13:54:03.541546 ARP, Request who-has 10.0.10.171 tell 10.0.10.1, length 28
13:54:03.541561 ARP, Request who-has 10.0.10.174 tell 10.0.10.1, length 28
13:54:03.541581 ARP, Request who-has 10.0.10.175 tell 10.0.10.1, length 28
13:54:03.541597 ARP, Request who-has 10.0.10.176 tell 10.0.10.1, length 28
13:54:03.541613 ARP, Request who-has 10.0.10.177 tell 10.0.10.1, length 28
13:54:03.541630 ARP, Request who-has 10.0.10.178 tell 10.0.10.1, length 28
13:54:03.541645 ARP, Request who-has 10.0.10.179 tell 10.0.10.1, length 28
13:54:03.541661 ARP, Request who-has 10.0.10.180 tell 10.0.10.1, length 28
13:54:03.541676 ARP, Request who-has 10.0.10.181 tell 10.0.10.1, length 28
13:54:03.541695 ARP, Request who-has 10.0.10.182 tell 10.0.10.1, length 28
13:54:03.541711 ARP, Request who-has 10.0.10.183 tell 10.0.10.1, length 28
13:54:03.541731 ARP, Request who-has 10.0.10.184 tell 10.0.10.1, length 28
13:54:03.541750 ARP, Request who-has 10.0.10.185 tell 10.0.10.1, length 28
13:54:03.541766 ARP, Request who-has 10.0.10.186 tell 10.0.10.1, length 28
13:54:03.541782 ARP, Request who-has 10.0.10.187 tell 10.0.10.1, length 28
13:54:03.541799 ARP, Request who-has 10.0.10.188 tell 10.0.10.1, length 28
13:54:03.560932 ARP, Request who-has 10.0.10.218 tell 10.0.10.1, length 28
13:54:03.560934 ARP, Request who-has 10.0.10.217 tell 10.0.10.1, length 28
13:54:03.560934 ARP, Request who-has 10.0.10.214 tell 10.0.10.1, length 28
13:54:03.560935 ARP, Request who-has 10.0.10.213 tell 10.0.10.1, length 28
13:54:03.560936 ARP, Request who-has 10.0.10.210 tell 10.0.10.1, length 28
13:54:03.560936 ARP, Request who-has 10.0.10.209 tell 10.0.10.1, length 28
13:54:03.560937 ARP, Request who-has 10.0.10.206 tell 10.0.10.1, length 28
13:54:03.560938 ARP, Request who-has 10.0.10.205 tell 10.0.10.1, length 28
13:54:03.560938 ARP, Request who-has 10.0.10.202 tell 10.0.10.1, length 28
13:54:03.560939 ARP, Request who-has 10.0.10.199 tell 10.0.10.1, length 28
13:54:03.560940 ARP, Request who-has 10.0.10.196 tell 10.0.10.1, length 28
13:54:03.560940 ARP, Request who-has 10.0.10.193 tell 10.0.10.1, length 28
13:54:03.640866 ARP, Request who-has 10.0.10.244 tell 10.0.10.1, length 28
13:54:03.640890 ARP, Request who-has 10.0.10.245 tell 10.0.10.1, length 28
13:54:03.640910 ARP, Request who-has 10.0.10.246 tell 10.0.10.1, length 28
13:54:03.641026 ARP, Request who-has 10.0.10.249 tell 10.0.10.1, length 28
13:54:03.641043 ARP, Request who-has 10.0.10.250 tell 10.0.10.1, length 28
13:54:03.641060 ARP, Request who-has 10.0.10.251 tell 10.0.10.1, length 28
13:54:03.641077 ARP, Request who-has 10.0.10.252 tell 10.0.10.1, length 28
13:54:03.641094 ARP, Request who-has 10.0.10.254 tell 10.0.10.1, length 28
13:54:03.641285 ARP, Request who-has 10.0.10.12 tell 10.0.10.1, length 28
13:54:03.641340 ARP, Request who-has 10.0.10.18 tell 10.0.10.1, length 28
13:54:03.641357 ARP, Request who-has 10.0.10.19 tell 10.0.10.1, length 28
13:54:03.641374 ARP, Request who-has 10.0.10.21 tell 10.0.10.1, length 28
13:54:03.641391 ARP, Request who-has 10.0.10.22 tell 10.0.10.1, length 28
13:54:03.641409 ARP, Request who-has 10.0.10.24 tell 10.0.10.1, length 28
13:54:03.641426 ARP, Request who-has 10.0.10.25 tell 10.0.10.1, length 28
13:54:03.641443 ARP, Request who-has 10.0.10.28 tell 10.0.10.1, length 28
13:54:03.641467 ARP, Request who-has 10.0.10.29 tell 10.0.10.1, length 28
13:54:03.641483 ARP, Request who-has 10.0.10.32 tell 10.0.10.1, length 28
13:54:03.641701 ARP, Request who-has 10.0.10.36 tell 10.0.10.1, length 28
13:54:03.641717 ARP, Request who-has 10.0.10.37 tell 10.0.10.1, length 28
13:54:03.641734 ARP, Request who-has 10.0.10.38 tell 10.0.10.1, length 28
13:54:03.641749 ARP, Request who-has 10.0.10.39 tell 10.0.10.1, length 28
13:54:03.641764 ARP, Request who-has 10.0.10.40 tell 10.0.10.1, length 28
13:54:03.641783 ARP, Request who-has 10.0.10.41 tell 10.0.10.1, length 28
13:54:03.641799 ARP, Request who-has 10.0.10.42 tell 10.0.10.1, length 28
13:54:03.641919 ARP, Request who-has 10.0.10.45 tell 10.0.10.1, length 28
13:54:03.641936 ARP, Request who-has 10.0.10.46 tell 10.0.10.1, length 28
13:54:03.641953 ARP, Request who-has 10.0.10.47 tell 10.0.10.1, length 28
13:54:03.641969 ARP, Request who-has 10.0.10.48 tell 10.0.10.1, length 28
13:54:03.641984 ARP, Request who-has 10.0.10.49 tell 10.0.10.1, length 28
13:54:03.642000 ARP, Request who-has 10.0.10.50 tell 10.0.10.1, length 28
13:54:03.642015 ARP, Request who-has 10.0.10.51 tell 10.0.10.1, length 28
13:54:03.642030 ARP, Request who-has 10.0.10.52 tell 10.0.10.1, length 28
13:54:03.642046 ARP, Request who-has 10.0.10.53 tell 10.0.10.1, length 28
13:54:03.642062 ARP, Request who-has 10.0.10.54 tell 10.0.10.1, length 28
13:54:03.642078 ARP, Request who-has 10.0.10.55 tell 10.0.10.1, length 28
13:54:03.642094 ARP, Request who-has 10.0.10.56 tell 10.0.10.1, length 28
13:54:03.642112 ARP, Request who-has 10.0.10.57 tell 10.0.10.1, length 28
13:54:03.642128 ARP, Request who-has 10.0.10.58 tell 10.0.10.1, length 28
13:54:03.642143 ARP, Request who-has 10.0.10.59 tell 10.0.10.1, length 28
13:54:03.642159 ARP, Request who-has 10.0.10.61 tell 10.0.10.1, length 28
13:54:03.642175 ARP, Request who-has 10.0.10.62 tell 10.0.10.1, length 28
13:54:03.656893 ARP, Request who-has 10.0.10.35 tell 10.0.10.1, length 28
13:54:03.656895 ARP, Request who-has 10.0.10.34 tell 10.0.10.1, length 28
13:54:03.656895 ARP, Request who-has 10.0.10.31 tell 10.0.10.1, length 28
13:54:03.656896 ARP, Request who-has 10.0.10.30 tell 10.0.10.1, length 28
13:54:03.656897 ARP, Request who-has 10.0.10.27 tell 10.0.10.1, length 28
13:54:03.656898 ARP, Request who-has 10.0.10.26 tell 10.0.10.1, length 28
13:54:03.656898 ARP, Request who-has 10.0.10.23 tell 10.0.10.1, length 28
13:54:03.656899 ARP, Request who-has 10.0.10.20 tell 10.0.10.1, length 28
13:54:03.656899 ARP, Request who-has 10.0.10.17 tell 10.0.10.1, length 28
13:54:03.656900 ARP, Request who-has 10.0.10.11 tell 10.0.10.1, length 28
13:54:03.656901 ARP, Request who-has 10.0.10.0 tell 10.0.10.1, length 28
13:54:03.656901 ARP, Request who-has 10.0.10.253 tell 10.0.10.1, length 28
13:54:03.741047 ARP, Request who-has 10.0.10.189 tell 10.0.10.1, length 28
13:54:03.741067 ARP, Request who-has 10.0.10.190 tell 10.0.10.1, length 28
13:54:03.741163 ARP, Request who-has 10.0.10.195 tell 10.0.10.1, length 28
13:54:03.741190 ARP, Request who-has 10.0.10.198 tell 10.0.10.1, length 28
13:54:03.741441 ARP, Request who-has 10.0.10.204 tell 10.0.10.1, length 28
13:54:03.741537 ARP, Request who-has 10.0.10.208 tell 10.0.10.1, length 28
13:54:03.741572 ARP, Request who-has 10.0.10.212 tell 10.0.10.1, length 28
13:54:03.741608 ARP, Request who-has 10.0.10.216 tell 10.0.10.1, length 28
13:54:03.741717 ARP, Request who-has 10.0.10.220 tell 10.0.10.1, length 28
13:54:03.741733 ARP, Request who-has 10.0.10.221 tell 10.0.10.1, length 28
13:54:03.741761 ARP, Request who-has 10.0.10.223 tell 10.0.10.1, length 28
13:54:03.741795 ARP, Request who-has 10.0.10.224 tell 10.0.10.1, length 28
13:54:03.741810 ARP, Request who-has 10.0.10.226 tell 10.0.10.1, length 28
13:54:03.741827 ARP, Request who-has 10.0.10.227 tell 10.0.10.1, length 28
13:54:03.741843 ARP, Request who-has 10.0.10.230 tell 10.0.10.1, length 28
13:54:03.741877 ARP, Request who-has 10.0.10.231 tell 10.0.10.1, length 28
13:54:03.741895 ARP, Request who-has 10.0.10.247 tell 10.0.10.1, length 28
13:54:03.742130 ARP, Request who-has 10.0.10.33 tell 10.0.10.1, length 28
13:54:03.742162 ARP, Request who-has 10.0.10.43 tell 10.0.10.1, length 28
13:54:03.742178 ARP, Request who-has 10.0.10.44 tell 10.0.10.1, length 28
13:54:03.742313 ARP, Request who-has 10.0.10.67 tell 10.0.10.1, length 28
13:54:03.752936 ARP, Request who-has 10.0.10.103 tell 10.0.10.1, length 28
13:54:03.752938 ARP, Request who-has 10.0.10.102 tell 10.0.10.1, length 28
13:54:03.752938 ARP, Request who-has 10.0.10.99 tell 10.0.10.1, length 28
13:54:03.752939 ARP, Request who-has 10.0.10.98 tell 10.0.10.1, length 28
13:54:03.752940 ARP, Request who-has 10.0.10.95 tell 10.0.10.1, length 28
13:54:03.752940 ARP, Request who-has 10.0.10.94 tell 10.0.10.1, length 28
13:54:03.752941 ARP, Request who-has 10.0.10.91 tell 10.0.10.1, length 28
13:54:03.752942 ARP, Request who-has 10.0.10.88 tell 10.0.10.1, length 28
13:54:03.752942 ARP, Request who-has 10.0.10.85 tell 10.0.10.1, length 28
13:54:03.752943 ARP, Request who-has 10.0.10.81 tell 10.0.10.1, length 28
13:54:03.752943 ARP, Request who-has 10.0.10.77 tell 10.0.10.1, length 28
13:54:03.752944 ARP, Request who-has 10.0.10.71 tell 10.0.10.1, length 28
13:54:03.841344 ARP, Request who-has 10.0.10.192 tell 10.0.10.1, length 28
13:54:03.841437 ARP, Request who-has 10.0.10.201 tell 10.0.10.1, length 28
13:54:03.841464 ARP, Request who-has 10.0.10.203 tell 10.0.10.1, length 28
13:54:03.842322 ARP, Request who-has 10.0.10.90 tell 10.0.10.1, length 28
13:54:03.848953 ARP, Request who-has 10.0.10.166 tell 10.0.10.1, length 28
13:54:03.848954 ARP, Request who-has 10.0.10.165 tell 10.0.10.1, length 28
13:54:03.848955 ARP, Request who-has 10.0.10.162 tell 10.0.10.1, length 28
13:54:03.848956 ARP, Request who-has 10.0.10.159 tell 10.0.10.1, length 28
13:54:03.848956 ARP, Request who-has 10.0.10.158 tell 10.0.10.1, length 28
13:54:03.848957 ARP, Request who-has 10.0.10.155 tell 10.0.10.1, length 28
13:54:03.848958 ARP, Request who-has 10.0.10.152 tell 10.0.10.1, length 28
13:54:03.848958 ARP, Request who-has 10.0.10.147 tell 10.0.10.1, length 28
13:54:03.848959 ARP, Request who-has 10.0.10.143 tell 10.0.10.1, length 28
13:54:03.848960 ARP, Request who-has 10.0.10.139 tell 10.0.10.1, length 28
13:54:03.848960 ARP, Request who-has 10.0.10.135 tell 10.0.10.1, length 28
13:54:03.848961 ARP, Request who-has 10.0.10.132 tell 10.0.10.1, length 28
13:54:03.941560 ARP, Request who-has 10.0.10.248 tell 10.0.10.1, length 28
13:54:03.944939 ARP, Request who-has 10.0.10.229 tell 10.0.10.1, length 28
13:54:03.944941 ARP, Request who-has 10.0.10.228 tell 10.0.10.1, length 28
13:54:03.944942 ARP, Request who-has 10.0.10.225 tell 10.0.10.1, length 28
13:54:03.944943 ARP, Request who-has 10.0.10.222 tell 10.0.10.1, length 28
13:54:03.944943 ARP, Request who-has 10.0.10.219 tell 10.0.10.1, length 28
13:54:03.944944 ARP, Request who-has 10.0.10.215 tell 10.0.10.1, length 28
13:54:03.944945 ARP, Request who-has 10.0.10.211 tell 10.0.10.1, length 28
13:54:03.944945 ARP, Request who-has 10.0.10.207 tell 10.0.10.1, length 28
13:54:03.944946 ARP, Request who-has 10.0.10.200 tell 10.0.10.1, length 28
13:54:03.944947 ARP, Request who-has 10.0.10.197 tell 10.0.10.1, length 28
13:54:03.944947 ARP, Request who-has 10.0.10.194 tell 10.0.10.1, length 28
13:54:03.944948 ARP, Request who-has 10.0.10.191 tell 10.0.10.1, length 28
13:54:04.044932 ARP, Request who-has 10.0.10.243 tell 10.0.10.1, length 28
13:54:04.044934 ARP, Request who-has 10.0.10.242 tell 10.0.10.1, length 28
13:54:04.044934 ARP, Request who-has 10.0.10.241 tell 10.0.10.1, length 28
13:54:04.044935 ARP, Request who-has 10.0.10.240 tell 10.0.10.1, length 28
13:54:04.044935 ARP, Request who-has 10.0.10.239 tell 10.0.10.1, length 28
13:54:04.044936 ARP, Request who-has 10.0.10.238 tell 10.0.10.1, length 28
13:54:04.044937 ARP, Request who-has 10.0.10.237 tell 10.0.10.1, length 28
13:54:04.044937 ARP, Request who-has 10.0.10.236 tell 10.0.10.1, length 28
13:54:04.044938 ARP, Request who-has 10.0.10.235 tell 10.0.10.1, length 28
13:54:04.044939 ARP, Request who-has 10.0.10.234 tell 10.0.10.1, length 28
13:54:04.044939 ARP, Request who-has 10.0.10.233 tell 10.0.10.1, length 28
13:54:04.044940 ARP, Request who-has 10.0.10.232 tell 10.0.10.1, length 28
13:54:04.168927 ARP, Request who-has 10.0.10.93 tell 10.0.10.1, length 28
13:54:04.168929 ARP, Request who-has 10.0.10.92 tell 10.0.10.1, length 28
13:54:04.168930 ARP, Request who-has 10.0.10.89 tell 10.0.10.1, length 28
13:54:04.168930 ARP, Request who-has 10.0.10.87 tell 10.0.10.1, length 28
13:54:04.168931 ARP, Request who-has 10.0.10.86 tell 10.0.10.1, length 28
13:54:04.168932 ARP, Request who-has 10.0.10.82 tell 10.0.10.1, length 28
13:54:04.168932 ARP, Request who-has 10.0.10.78 tell 10.0.10.1, length 28
13:54:04.168933 ARP, Request who-has 10.0.10.72 tell 10.0.10.1, length 28
13:54:04.168934 ARP, Request who-has 10.0.10.66 tell 10.0.10.1, length 28
13:54:04.168934 ARP, Request who-has 10.0.10.64 tell 10.0.10.1, length 28
13:54:04.168935 ARP, Request who-has 10.0.10.63 tell 10.0.10.1, length 28
13:54:04.168936 ARP, Request who-has 10.0.10.60 tell 10.0.10.1, length 28
13:54:04.265000 ARP, Request who-has 10.0.10.3 tell 10.0.10.1, length 28
13:54:04.265002 ARP, Request who-has 10.0.10.4 tell 10.0.10.1, length 28
13:54:04.265002 ARP, Request who-has 10.0.10.5 tell 10.0.10.1, length 28
13:54:04.265003 ARP, Request who-has 10.0.10.6 tell 10.0.10.1, length 28
13:54:04.265004 ARP, Request who-has 10.0.10.7 tell 10.0.10.1, length 28
13:54:04.265004 ARP, Request who-has 10.0.10.8 tell 10.0.10.1, length 28
13:54:04.265005 ARP, Request who-has 10.0.10.9 tell 10.0.10.1, length 28
13:54:04.265006 ARP, Request who-has 10.0.10.10 tell 10.0.10.1, length 28
13:54:04.265006 ARP, Request who-has 10.0.10.13 tell 10.0.10.1, length 28
13:54:04.265007 ARP, Request who-has 10.0.10.14 tell 10.0.10.1, length 28
13:54:04.265007 ARP, Request who-has 10.0.10.15 tell 10.0.10.1, length 28
13:54:04.265008 ARP, Request who-has 10.0.10.16 tell 10.0.10.1, length 28
13:54:04.265009 ARP, Request who-has 10.0.10.173 tell 10.0.10.1, length 28
13:54:04.265009 ARP, Request who-has 10.0.10.172 tell 10.0.10.1, length 28
13:54:04.265010 ARP, Request who-has 10.0.10.169 tell 10.0.10.1, length 28
13:54:04.265011 ARP, Request who-has 10.0.10.168 tell 10.0.10.1, length 28
13:54:04.265011 ARP, Request who-has 10.0.10.167 tell 10.0.10.1, length 28
13:54:04.265012 ARP, Request who-has 10.0.10.163 tell 10.0.10.1, length 28
13:54:04.265013 ARP, Request who-has 10.0.10.161 tell 10.0.10.1, length 28
13:54:04.265013 ARP, Request who-has 10.0.10.160 tell 10.0.10.1, length 28
13:54:04.265014 ARP, Request who-has 10.0.10.157 tell 10.0.10.1, length 28
13:54:04.265015 ARP, Request who-has 10.0.10.156 tell 10.0.10.1, length 28
13:54:04.265015 ARP, Request who-has 10.0.10.153 tell 10.0.10.1, length 28
13:54:04.265016 ARP, Request who-has 10.0.10.148 tell 10.0.10.1, length 28
13:54:04.392915 ARP, Request who-has 10.0.10.65 tell 10.0.10.1, length 28
13:54:04.392917 ARP, Request who-has 10.0.10.68 tell 10.0.10.1, length 28
13:54:04.392918 ARP, Request who-has 10.0.10.69 tell 10.0.10.1, length 28
13:54:04.392919 ARP, Request who-has 10.0.10.70 tell 10.0.10.1, length 28
13:54:04.392919 ARP, Request who-has 10.0.10.73 tell 10.0.10.1, length 28
13:54:04.392920 ARP, Request who-has 10.0.10.74 tell 10.0.10.1, length 28
13:54:04.392921 ARP, Request who-has 10.0.10.75 tell 10.0.10.1, length 28
13:54:04.392921 ARP, Request who-has 10.0.10.76 tell 10.0.10.1, length 28
13:54:04.392922 ARP, Request who-has 10.0.10.79 tell 10.0.10.1, length 28
13:54:04.392922 ARP, Request who-has 10.0.10.80 tell 10.0.10.1, length 28
13:54:04.392923 ARP, Request who-has 10.0.10.83 tell 10.0.10.1, length 28
13:54:04.392924 ARP, Request who-has 10.0.10.84 tell 10.0.10.1, length 28
13:54:04.488924 ARP, Request who-has 10.0.10.131 tell 10.0.10.1, length 28
13:54:04.488925 ARP, Request who-has 10.0.10.134 tell 10.0.10.1, length 28
13:54:04.488926 ARP, Request who-has 10.0.10.137 tell 10.0.10.1, length 28
13:54:04.488927 ARP, Request who-has 10.0.10.138 tell 10.0.10.1, length 28
13:54:04.488927 ARP, Request who-has 10.0.10.141 tell 10.0.10.1, length 28
13:54:04.488928 ARP, Request who-has 10.0.10.142 tell 10.0.10.1, length 28
13:54:04.488929 ARP, Request who-has 10.0.10.145 tell 10.0.10.1, length 28
13:54:04.488929 ARP, Request who-has 10.0.10.146 tell 10.0.10.1, length 28
13:54:04.488930 ARP, Request who-has 10.0.10.149 tell 10.0.10.1, length 28
13:54:04.488930 ARP, Request who-has 10.0.10.150 tell 10.0.10.1, length 28
13:54:04.488931 ARP, Request who-has 10.0.10.151 tell 10.0.10.1, length 28
13:54:04.488932 ARP, Request who-has 10.0.10.154 tell 10.0.10.1, length 28
13:54:04.557150 ARP, Request who-has 10.0.10.188 tell 10.0.10.1, length 28
13:54:04.557152 ARP, Request who-has 10.0.10.187 tell 10.0.10.1, length 28
13:54:04.557153 ARP, Request who-has 10.0.10.186 tell 10.0.10.1, length 28
13:54:04.557154 ARP, Request who-has 10.0.10.185 tell 10.0.10.1, length 28
13:54:04.557154 ARP, Request who-has 10.0.10.184 tell 10.0.10.1, length 28
13:54:04.557155 ARP, Request who-has 10.0.10.183 tell 10.0.10.1, length 28
13:54:04.557156 ARP, Request who-has 10.0.10.182 tell 10.0.10.1, length 28
13:54:04.557156 ARP, Request who-has 10.0.10.181 tell 10.0.10.1, length 28
13:54:04.557157 ARP, Request who-has 10.0.10.180 tell 10.0.10.1, length 28
13:54:04.557158 ARP, Request who-has 10.0.10.179 tell 10.0.10.1, length 28
13:54:04.557158 ARP, Request who-has 10.0.10.178 tell 10.0.10.1, length 28
13:54:04.557159 ARP, Request who-has 10.0.10.177 tell 10.0.10.1, length 28
13:54:04.557160 ARP, Request who-has 10.0.10.176 tell 10.0.10.1, length 28
13:54:04.557160 ARP, Request who-has 10.0.10.175 tell 10.0.10.1, length 28
13:54:04.557161 ARP, Request who-has 10.0.10.174 tell 10.0.10.1, length 28
13:54:04.557162 ARP, Request who-has 10.0.10.171 tell 10.0.10.1, length 28
13:54:04.557162 ARP, Request who-has 10.0.10.170 tell 10.0.10.1, length 28
13:54:04.557163 ARP, Request who-has 10.0.10.164 tell 10.0.10.1, length 28
13:54:04.557164 ARP, Request who-has 10.0.10.144 tell 10.0.10.1, length 28
13:54:04.557165 ARP, Request who-has 10.0.10.140 tell 10.0.10.1, length 28
13:54:04.557165 ARP, Request who-has 10.0.10.136 tell 10.0.10.1, length 28
13:54:04.557166 ARP, Request who-has 10.0.10.133 tell 10.0.10.1, length 28
13:54:04.557167 ARP, Request who-has 10.0.10.130 tell 10.0.10.1, length 28
13:54:04.557167 ARP, Request who-has 10.0.10.129 tell 10.0.10.1, length 28
13:54:04.557168 ARP, Request who-has 10.0.10.128 tell 10.0.10.1, length 28
13:54:04.557168 ARP, Request who-has 10.0.10.127 tell 10.0.10.1, length 28
13:54:04.557169 ARP, Request who-has 10.0.10.126 tell 10.0.10.1, length 28
13:54:04.557170 ARP, Request who-has 10.0.10.125 tell 10.0.10.1, length 28
13:54:04.557170 ARP, Request who-has 10.0.10.124 tell 10.0.10.1, length 28
13:54:04.557171 ARP, Request who-has 10.0.10.123 tell 10.0.10.1, length 28
13:54:04.557172 ARP, Request who-has 10.0.10.122 tell 10.0.10.1, length 28
13:54:04.557172 ARP, Request who-has 10.0.10.121 tell 10.0.10.1, length 28
13:54:04.557173 ARP, Request who-has 10.0.10.120 tell 10.0.10.1, length 28
13:54:04.557174 ARP, Request who-has 10.0.10.119 tell 10.0.10.1, length 28
13:54:04.557174 ARP, Request who-has 10.0.10.118 tell 10.0.10.1, length 28
13:54:04.557175 ARP, Request who-has 10.0.10.117 tell 10.0.10.1, length 28
13:54:04.557175 ARP, Request who-has 10.0.10.116 tell 10.0.10.1, length 28
13:54:04.557176 ARP, Request who-has 10.0.10.115 tell 10.0.10.1, length 28
13:54:04.557177 ARP, Request who-has 10.0.10.114 tell 10.0.10.1, length 28
13:54:04.557178 ARP, Request who-has 10.0.10.113 tell 10.0.10.1, length 28
13:54:04.557178 ARP, Request who-has 10.0.10.112 tell 10.0.10.1, length 28
13:54:04.557179 ARP, Request who-has 10.0.10.111 tell 10.0.10.1, length 28
13:54:04.557180 ARP, Request who-has 10.0.10.110 tell 10.0.10.1, length 28
13:54:04.557180 ARP, Request who-has 10.0.10.109 tell 10.0.10.1, length 28
13:54:04.557181 ARP, Request who-has 10.0.10.108 tell 10.0.10.1, length 28
13:54:04.557182 ARP, Request who-has 10.0.10.107 tell 10.0.10.1, length 28
13:54:04.557182 ARP, Request who-has 10.0.10.106 tell 10.0.10.1, length 28
13:54:04.557183 ARP, Request who-has 10.0.10.105 tell 10.0.10.1, length 28
13:54:04.557184 ARP, Request who-has 10.0.10.104 tell 10.0.10.1, length 28
13:54:04.557184 ARP, Request who-has 10.0.10.101 tell 10.0.10.1, length 28
13:54:04.557185 ARP, Request who-has 10.0.10.100 tell 10.0.10.1, length 28
13:54:04.557185 ARP, Request who-has 10.0.10.97 tell 10.0.10.1, length 28
13:54:04.557186 ARP, Request who-has 10.0.10.96 tell 10.0.10.1, length 28
13:54:04.588884 ARP, Request who-has 10.0.10.193 tell 10.0.10.1, length 28
13:54:04.588885 ARP, Request who-has 10.0.10.196 tell 10.0.10.1, length 28
13:54:04.588886 ARP, Request who-has 10.0.10.199 tell 10.0.10.1, length 28
13:54:04.588887 ARP, Request who-has 10.0.10.202 tell 10.0.10.1, length 28
13:54:04.588887 ARP, Request who-has 10.0.10.205 tell 10.0.10.1, length 28
13:54:04.588888 ARP, Request who-has 10.0.10.206 tell 10.0.10.1, length 28
13:54:04.588889 ARP, Request who-has 10.0.10.209 tell 10.0.10.1, length 28
13:54:04.588889 ARP, Request who-has 10.0.10.210 tell 10.0.10.1, length 28
13:54:04.588890 ARP, Request who-has 10.0.10.213 tell 10.0.10.1, length 28
13:54:04.588890 ARP, Request who-has 10.0.10.214 tell 10.0.10.1, length 28
13:54:04.588891 ARP, Request who-has 10.0.10.217 tell 10.0.10.1, length 28
13:54:04.588892 ARP, Request who-has 10.0.10.218 tell 10.0.10.1, length 28
13:54:04.649103 ARP, Request who-has 10.0.10.62 tell 10.0.10.1, length 28
13:54:04.649105 ARP, Request who-has 10.0.10.61 tell 10.0.10.1, length 28
13:54:04.649105 ARP, Request who-has 10.0.10.59 tell 10.0.10.1, length 28
13:54:04.649106 ARP, Request who-has 10.0.10.58 tell 10.0.10.1, length 28
13:54:04.649107 ARP, Request who-has 10.0.10.57 tell 10.0.10.1, length 28
13:54:04.649107 ARP, Request who-has 10.0.10.56 tell 10.0.10.1, length 28
13:54:04.649108 ARP, Request who-has 10.0.10.55 tell 10.0.10.1, length 28
13:54:04.649109 ARP, Request who-has 10.0.10.54 tell 10.0.10.1, length 28
13:54:04.649109 ARP, Request who-has 10.0.10.53 tell 10.0.10.1, length 28
13:54:04.649110 ARP, Request who-has 10.0.10.52 tell 10.0.10.1, length 28
13:54:04.649111 ARP, Request who-has 10.0.10.51 tell 10.0.10.1, length 28
13:54:04.649111 ARP, Request who-has 10.0.10.50 tell 10.0.10.1, length 28
13:54:04.649112 ARP, Request who-has 10.0.10.49 tell 10.0.10.1, length 28
13:54:04.649113 ARP, Request who-has 10.0.10.48 tell 10.0.10.1, length 28
13:54:04.649113 ARP, Request who-has 10.0.10.47 tell 10.0.10.1, length 28
13:54:04.649114 ARP, Request who-has 10.0.10.46 tell 10.0.10.1, length 28
13:54:04.649114 ARP, Request who-has 10.0.10.45 tell 10.0.10.1, length 28
13:54:04.649115 ARP, Request who-has 10.0.10.42 tell 10.0.10.1, length 28
13:54:04.649116 ARP, Request who-has 10.0.10.41 tell 10.0.10.1, length 28
13:54:04.649117 ARP, Request who-has 10.0.10.40 tell 10.0.10.1, length 28
13:54:04.649117 ARP, Request who-has 10.0.10.39 tell 10.0.10.1, length 28
13:54:04.649118 ARP, Request who-has 10.0.10.38 tell 10.0.10.1, length 28
13:54:04.649119 ARP, Request who-has 10.0.10.37 tell 10.0.10.1, length 28
13:54:04.649119 ARP, Request who-has 10.0.10.36 tell 10.0.10.1, length 28
13:54:04.649120 ARP, Request who-has 10.0.10.32 tell 10.0.10.1, length 28
13:54:04.649121 ARP, Request who-has 10.0.10.29 tell 10.0.10.1, length 28
13:54:04.649121 ARP, Request who-has 10.0.10.28 tell 10.0.10.1, length 28
13:54:04.649122 ARP, Request who-has 10.0.10.25 tell 10.0.10.1, length 28
13:54:04.649123 ARP, Request who-has 10.0.10.24 tell 10.0.10.1, length 28
13:54:04.649123 ARP, Request who-has 10.0.10.22 tell 10.0.10.1, length 28
13:54:04.649124 ARP, Request who-has 10.0.10.21 tell 10.0.10.1, length 28
13:54:04.649125 ARP, Request who-has 10.0.10.19 tell 10.0.10.1, length 28
13:54:04.649125 ARP, Request who-has 10.0.10.18 tell 10.0.10.1, length 28
13:54:04.649126 ARP, Request who-has 10.0.10.12 tell 10.0.10.1, length 28
13:54:04.649127 ARP, Request who-has 10.0.10.254 tell 10.0.10.1, length 28
13:54:04.649127 ARP, Request who-has 10.0.10.252 tell 10.0.10.1, length 28
13:54:04.649128 ARP, Request who-has 10.0.10.251 tell 10.0.10.1, length 28
13:54:04.649129 ARP, Request who-has 10.0.10.250 tell 10.0.10.1, length 28
13:54:04.649129 ARP, Request who-has 10.0.10.249 tell 10.0.10.1, length 28
13:54:04.649130 ARP, Request who-has 10.0.10.246 tell 10.0.10.1, length 28
13:54:04.649131 ARP, Request who-has 10.0.10.245 tell 10.0.10.1, length 28
13:54:04.649131 ARP, Request who-has 10.0.10.244 tell 10.0.10.1, length 28
13:54:04.680926 ARP, Request who-has 10.0.10.253 tell 10.0.10.1, length 28
13:54:04.680928 ARP, Request who-has 10.0.10.0 tell 10.0.10.1, length 28
13:54:04.680928 ARP, Request who-has 10.0.10.11 tell 10.0.10.1, length 28
13:54:04.680929 ARP, Request who-has 10.0.10.17 tell 10.0.10.1, length 28
13:54:04.680930 ARP, Request who-has 10.0.10.20 tell 10.0.10.1, length 28
13:54:04.680930 ARP, Request who-has 10.0.10.23 tell 10.0.10.1, length 28
13:54:04.680931 ARP, Request who-has 10.0.10.26 tell 10.0.10.1, length 28
13:54:04.680932 ARP, Request who-has 10.0.10.27 tell 10.0.10.1, length 28
13:54:04.680932 ARP, Request who-has 10.0.10.30 tell 10.0.10.1, length 28
13:54:04.680933 ARP, Request who-has 10.0.10.31 tell 10.0.10.1, length 28
13:54:04.680934 ARP, Request who-has 10.0.10.34 tell 10.0.10.1, length 28
13:54:04.680934 ARP, Request who-has 10.0.10.35 tell 10.0.10.1, length 28
13:54:04.744975 ARP, Request who-has 10.0.10.67 tell 10.0.10.1, length 28
13:54:04.744977 ARP, Request who-has 10.0.10.44 tell 10.0.10.1, length 28
13:54:04.744978 ARP, Request who-has 10.0.10.43 tell 10.0.10.1, length 28
13:54:04.744978 ARP, Request who-has 10.0.10.33 tell 10.0.10.1, length 28
13:54:04.744979 ARP, Request who-has 10.0.10.247 tell 10.0.10.1, length 28
13:54:04.744980 ARP, Request who-has 10.0.10.231 tell 10.0.10.1, length 28
13:54:04.744981 ARP, Request who-has 10.0.10.230 tell 10.0.10.1, length 28
13:54:04.744981 ARP, Request who-has 10.0.10.227 tell 10.0.10.1, length 28
13:54:04.744982 ARP, Request who-has 10.0.10.226 tell 10.0.10.1, length 28
13:54:04.744982 ARP, Request who-has 10.0.10.224 tell 10.0.10.1, length 28
13:54:04.744983 ARP, Request who-has 10.0.10.223 tell 10.0.10.1, length 28
13:54:04.744984 ARP, Request who-has 10.0.10.221 tell 10.0.10.1, length 28
13:54:04.744984 ARP, Request who-has 10.0.10.220 tell 10.0.10.1, length 28
13:54:04.744985 ARP, Request who-has 10.0.10.216 tell 10.0.10.1, length 28
13:54:04.744986 ARP, Request who-has 10.0.10.212 tell 10.0.10.1, length 28
13:54:04.744986 ARP, Request who-has 10.0.10.208 tell 10.0.10.1, length 28
13:54:04.744987 ARP, Request who-has 10.0.10.204 tell 10.0.10.1, length 28
13:54:04.744988 ARP, Request who-has 10.0.10.198 tell 10.0.10.1, length 28
13:54:04.744988 ARP, Request who-has 10.0.10.195 tell 10.0.10.1, length 28
13:54:04.744989 ARP, Request who-has 10.0.10.190 tell 10.0.10.1, length 28
13:54:04.744989 ARP, Request who-has 10.0.10.189 tell 10.0.10.1, length 28
13:54:04.776896 ARP, Request who-has 10.0.10.71 tell 10.0.10.1, length 28
13:54:04.776898 ARP, Request who-has 10.0.10.77 tell 10.0.10.1, length 28
13:54:04.776898 ARP, Request who-has 10.0.10.81 tell 10.0.10.1, length 28
13:54:04.776899 ARP, Request who-has 10.0.10.85 tell 10.0.10.1, length 28
13:54:04.776900 ARP, Request who-has 10.0.10.88 tell 10.0.10.1, length 28
13:54:04.776900 ARP, Request who-has 10.0.10.91 tell 10.0.10.1, length 28
13:54:04.776901 ARP, Request who-has 10.0.10.94 tell 10.0.10.1, length 28
13:54:04.776902 ARP, Request who-has 10.0.10.95 tell 10.0.10.1, length 28
13:54:04.776902 ARP, Request who-has 10.0.10.98 tell 10.0.10.1, length 28
13:54:04.776903 ARP, Request who-has 10.0.10.99 tell 10.0.10.1, length 28
13:54:04.776903 ARP, Request who-has 10.0.10.102 tell 10.0.10.1, length 28
13:54:04.776904 ARP, Request who-has 10.0.10.103 tell 10.0.10.1, length 28
13:54:04.872958 ARP, Request who-has 10.0.10.132 tell 10.0.10.1, length 28
13:54:04.872959 ARP, Request who-has 10.0.10.135 tell 10.0.10.1, length 28
13:54:04.872960 ARP, Request who-has 10.0.10.139 tell 10.0.10.1, length 28
13:54:04.872961 ARP, Request who-has 10.0.10.143 tell 10.0.10.1, length 28
13:54:04.872961 ARP, Request who-has 10.0.10.147 tell 10.0.10.1, length 28
13:54:04.872962 ARP, Request who-has 10.0.10.152 tell 10.0.10.1, length 28
13:54:04.872963 ARP, Request who-has 10.0.10.155 tell 10.0.10.1, length 28
13:54:04.872963 ARP, Request who-has 10.0.10.158 tell 10.0.10.1, length 28
13:54:04.872964 ARP, Request who-has 10.0.10.159 tell 10.0.10.1, length 28
13:54:04.872965 ARP, Request who-has 10.0.10.162 tell 10.0.10.1, length 28
13:54:04.872965 ARP, Request who-has 10.0.10.165 tell 10.0.10.1, length 28
13:54:04.872966 ARP, Request who-has 10.0.10.166 tell 10.0.10.1, length 28
13:54:04.872967 ARP, Request who-has 10.0.10.90 tell 10.0.10.1, length 28
13:54:04.872967 ARP, Request who-has 10.0.10.203 tell 10.0.10.1, length 28
13:54:04.872968 ARP, Request who-has 10.0.10.201 tell 10.0.10.1, length 28
13:54:04.872969 ARP, Request who-has 10.0.10.192 tell 10.0.10.1, length 28
13:54:04.940875 IP 10.0.10.1.57006 > 10.0.10.2.http: Flags [S], seq 2246678905, win 64240, options [mss 1460,sackOK,TS val 3851874680 ecr 0,nop,wscale 7], length 0
13:54:04.940882 IP 10.0.10.2.http > 10.0.10.1.57006: Flags [R.], seq 0, ack 2246678906, win 0, length 0
13:54:04.968925 ARP, Request who-has 10.0.10.191 tell 10.0.10.1, length 28
13:54:04.968927 ARP, Request who-has 10.0.10.194 tell 10.0.10.1, length 28
13:54:04.968927 ARP, Request who-has 10.0.10.197 tell 10.0.10.1, length 28
13:54:04.968928 ARP, Request who-has 10.0.10.200 tell 10.0.10.1, length 28
13:54:04.968929 ARP, Request who-has 10.0.10.207 tell 10.0.10.1, length 28
13:54:04.968929 ARP, Request who-has 10.0.10.211 tell 10.0.10.1, length 28
13:54:04.968930 ARP, Request who-has 10.0.10.215 tell 10.0.10.1, length 28
13:54:04.968930 ARP, Request who-has 10.0.10.219 tell 10.0.10.1, length 28
13:54:04.968931 ARP, Request who-has 10.0.10.222 tell 10.0.10.1, length 28
13:54:04.968932 ARP, Request who-has 10.0.10.225 tell 10.0.10.1, length 28
13:54:04.968932 ARP, Request who-has 10.0.10.228 tell 10.0.10.1, length 28
13:54:04.968933 ARP, Request who-has 10.0.10.229 tell 10.0.10.1, length 28
13:54:04.968934 ARP, Request who-has 10.0.10.248 tell 10.0.10.1, length 28
13:54:05.064931 ARP, Request who-has 10.0.10.232 tell 10.0.10.1, length 28
13:54:05.064933 ARP, Request who-has 10.0.10.233 tell 10.0.10.1, length 28
13:54:05.064934 ARP, Request who-has 10.0.10.234 tell 10.0.10.1, length 28
13:54:05.064934 ARP, Request who-has 10.0.10.235 tell 10.0.10.1, length 28
13:54:05.064935 ARP, Request who-has 10.0.10.236 tell 10.0.10.1, length 28
13:54:05.064936 ARP, Request who-has 10.0.10.237 tell 10.0.10.1, length 28
13:54:05.064936 ARP, Request who-has 10.0.10.238 tell 10.0.10.1, length 28
13:54:05.064937 ARP, Request who-has 10.0.10.239 tell 10.0.10.1, length 28
13:54:05.064938 ARP, Request who-has 10.0.10.240 tell 10.0.10.1, length 28
13:54:05.064938 ARP, Request who-has 10.0.10.241 tell 10.0.10.1, length 28
13:54:05.064939 ARP, Request who-has 10.0.10.242 tell 10.0.10.1, length 28
13:54:05.064939 ARP, Request who-has 10.0.10.243 tell 10.0.10.1, length 28
13:54:05.192927 ARP, Request who-has 10.0.10.60 tell 10.0.10.1, length 28
13:54:05.192929 ARP, Request who-has 10.0.10.63 tell 10.0.10.1, length 28
13:54:05.192929 ARP, Request who-has 10.0.10.64 tell 10.0.10.1, length 28
13:54:05.192930 ARP, Request who-has 10.0.10.66 tell 10.0.10.1, length 28
13:54:05.192931 ARP, Request who-has 10.0.10.72 tell 10.0.10.1, length 28
13:54:05.192932 ARP, Request who-has 10.0.10.78 tell 10.0.10.1, length 28
13:54:05.192932 ARP, Request who-has 10.0.10.82 tell 10.0.10.1, length 28
13:54:05.192933 ARP, Request who-has 10.0.10.86 tell 10.0.10.1, length 28
13:54:05.192933 ARP, Request who-has 10.0.10.87 tell 10.0.10.1, length 28
13:54:05.192934 ARP, Request who-has 10.0.10.89 tell 10.0.10.1, length 28
13:54:05.192935 ARP, Request who-has 10.0.10.92 tell 10.0.10.1, length 28
13:54:05.192935 ARP, Request who-has 10.0.10.93 tell 10.0.10.1, length 28
13:54:05.289141 ARP, Request who-has 10.0.10.148 tell 10.0.10.1, length 28
13:54:05.289143 ARP, Request who-has 10.0.10.153 tell 10.0.10.1, length 28
13:54:05.289143 ARP, Request who-has 10.0.10.156 tell 10.0.10.1, length 28
13:54:05.289144 ARP, Request who-has 10.0.10.157 tell 10.0.10.1, length 28
13:54:05.289145 ARP, Request who-has 10.0.10.160 tell 10.0.10.1, length 28
13:54:05.289145 ARP, Request who-has 10.0.10.161 tell 10.0.10.1, length 28
13:54:05.289146 ARP, Request who-has 10.0.10.163 tell 10.0.10.1, length 28
13:54:05.289147 ARP, Request who-has 10.0.10.167 tell 10.0.10.1, length 28
13:54:05.289147 ARP, Request who-has 10.0.10.168 tell 10.0.10.1, length 28
13:54:05.289148 ARP, Request who-has 10.0.10.169 tell 10.0.10.1, length 28
13:54:05.289149 ARP, Request who-has 10.0.10.172 tell 10.0.10.1, length 28
13:54:05.289149 ARP, Request who-has 10.0.10.173 tell 10.0.10.1, length 28
13:54:05.577198 ARP, Request who-has 10.0.10.96 tell 10.0.10.1, length 28
13:54:05.577202 ARP, Request who-has 10.0.10.97 tell 10.0.10.1, length 28
13:54:05.577203 ARP, Request who-has 10.0.10.100 tell 10.0.10.1, length 28
13:54:05.577203 ARP, Request who-has 10.0.10.101 tell 10.0.10.1, length 28
13:54:05.577204 ARP, Request who-has 10.0.10.104 tell 10.0.10.1, length 28
13:54:05.577205 ARP, Request who-has 10.0.10.105 tell 10.0.10.1, length 28
13:54:05.577205 ARP, Request who-has 10.0.10.106 tell 10.0.10.1, length 28
13:54:05.577206 ARP, Request who-has 10.0.10.107 tell 10.0.10.1, length 28
13:54:05.577207 ARP, Request who-has 10.0.10.108 tell 10.0.10.1, length 28
13:54:05.577207 ARP, Request who-has 10.0.10.109 tell 10.0.10.1, length 28
13:54:05.577208 ARP, Request who-has 10.0.10.110 tell 10.0.10.1, length 28
13:54:05.577209 ARP, Request who-has 10.0.10.111 tell 10.0.10.1, length 28
13:54:05.577209 ARP, Request who-has 10.0.10.112 tell 10.0.10.1, length 28
13:54:05.577210 ARP, Request who-has 10.0.10.113 tell 10.0.10.1, length 28
13:54:05.577210 ARP, Request who-has 10.0.10.114 tell 10.0.10.1, length 28
13:54:05.577211 ARP, Request who-has 10.0.10.115 tell 10.0.10.1, length 28
13:54:05.577212 ARP, Request who-has 10.0.10.116 tell 10.0.10.1, length 28
13:54:05.577212 ARP, Request who-has 10.0.10.117 tell 10.0.10.1, length 28
13:54:05.577213 ARP, Request who-has 10.0.10.118 tell 10.0.10.1, length 28
13:54:05.577214 ARP, Request who-has 10.0.10.119 tell 10.0.10.1, length 28
13:54:05.577215 ARP, Request who-has 10.0.10.120 tell 10.0.10.1, length 28
13:54:05.577216 ARP, Request who-has 10.0.10.121 tell 10.0.10.1, length 28
13:54:05.577216 ARP, Request who-has 10.0.10.122 tell 10.0.10.1, length 28
13:54:05.577217 ARP, Request who-has 10.0.10.123 tell 10.0.10.1, length 28
13:54:05.577218 ARP, Request who-has 10.0.10.124 tell 10.0.10.1, length 28
13:54:05.577218 ARP, Request who-has 10.0.10.125 tell 10.0.10.1, length 28
13:54:05.577219 ARP, Request who-has 10.0.10.126 tell 10.0.10.1, length 28
13:54:05.577220 ARP, Request who-has 10.0.10.127 tell 10.0.10.1, length 28
13:54:05.577220 ARP, Request who-has 10.0.10.128 tell 10.0.10.1, length 28
13:54:05.577221 ARP, Request who-has 10.0.10.129 tell 10.0.10.1, length 28
13:54:05.577222 ARP, Request who-has 10.0.10.130 tell 10.0.10.1, length 28
13:54:05.577222 ARP, Request who-has 10.0.10.133 tell 10.0.10.1, length 28
13:54:05.577223 ARP, Request who-has 10.0.10.136 tell 10.0.10.1, length 28
13:54:05.577224 ARP, Request who-has 10.0.10.140 tell 10.0.10.1, length 28
13:54:05.577224 ARP, Request who-has 10.0.10.144 tell 10.0.10.1, length 28
13:54:05.577225 ARP, Request who-has 10.0.10.164 tell 10.0.10.1, length 28
13:54:05.577226 ARP, Request who-has 10.0.10.170 tell 10.0.10.1, length 28
13:54:05.577226 ARP, Request who-has 10.0.10.171 tell 10.0.10.1, length 28
13:54:05.577227 ARP, Request who-has 10.0.10.174 tell 10.0.10.1, length 28
13:54:05.577228 ARP, Request who-has 10.0.10.175 tell 10.0.10.1, length 28
13:54:05.577229 ARP, Request who-has 10.0.10.176 tell 10.0.10.1, length 28
13:54:05.577229 ARP, Request who-has 10.0.10.177 tell 10.0.10.1, length 28
13:54:05.577230 ARP, Request who-has 10.0.10.178 tell 10.0.10.1, length 28
13:54:05.577231 ARP, Request who-has 10.0.10.179 tell 10.0.10.1, length 28
13:54:05.577231 ARP, Request who-has 10.0.10.180 tell 10.0.10.1, length 28
13:54:05.577232 ARP, Request who-has 10.0.10.181 tell 10.0.10.1, length 28
13:54:05.577233 ARP, Request who-has 10.0.10.182 tell 10.0.10.1, length 28
13:54:05.577233 ARP, Request who-has 10.0.10.183 tell 10.0.10.1, length 28
13:54:05.577234 ARP, Request who-has 10.0.10.184 tell 10.0.10.1, length 28
13:54:05.577234 ARP, Request who-has 10.0.10.185 tell 10.0.10.1, length 28
13:54:05.577235 ARP, Request who-has 10.0.10.186 tell 10.0.10.1, length 28
13:54:05.577236 ARP, Request who-has 10.0.10.187 tell 10.0.10.1, length 28
13:54:05.577236 ARP, Request who-has 10.0.10.188 tell 10.0.10.1, length 28
13:54:05.673096 ARP, Request who-has 10.0.10.244 tell 10.0.10.1, length 28
13:54:05.673099 ARP, Request who-has 10.0.10.245 tell 10.0.10.1, length 28
13:54:05.673100 ARP, Request who-has 10.0.10.246 tell 10.0.10.1, length 28
13:54:05.673101 ARP, Request who-has 10.0.10.249 tell 10.0.10.1, length 28
13:54:05.673101 ARP, Request who-has 10.0.10.250 tell 10.0.10.1, length 28
13:54:05.673102 ARP, Request who-has 10.0.10.251 tell 10.0.10.1, length 28
13:54:05.673103 ARP, Request who-has 10.0.10.252 tell 10.0.10.1, length 28
13:54:05.673103 ARP, Request who-has 10.0.10.254 tell 10.0.10.1, length 28
13:54:05.673104 ARP, Request who-has 10.0.10.12 tell 10.0.10.1, length 28
13:54:05.673105 ARP, Request who-has 10.0.10.18 tell 10.0.10.1, length 28
13:54:05.673105 ARP, Request who-has 10.0.10.19 tell 10.0.10.1, length 28
13:54:05.673106 ARP, Request who-has 10.0.10.21 tell 10.0.10.1, length 28
13:54:05.673107 ARP, Request who-has 10.0.10.22 tell 10.0.10.1, length 28
13:54:05.673107 ARP, Request who-has 10.0.10.24 tell 10.0.10.1, length 28
13:54:05.673108 ARP, Request who-has 10.0.10.25 tell 10.0.10.1, length 28
13:54:05.673108 ARP, Request who-has 10.0.10.28 tell 10.0.10.1, length 28
13:54:05.673109 ARP, Request who-has 10.0.10.29 tell 10.0.10.1, length 28
13:54:05.673110 ARP, Request who-has 10.0.10.32 tell 10.0.10.1, length 28
13:54:05.673110 ARP, Request who-has 10.0.10.36 tell 10.0.10.1, length 28
13:54:05.673111 ARP, Request who-has 10.0.10.37 tell 10.0.10.1, length 28
13:54:05.673112 ARP, Request who-has 10.0.10.38 tell 10.0.10.1, length 28
13:54:05.673113 ARP, Request who-has 10.0.10.39 tell 10.0.10.1, length 28
13:54:05.673113 ARP, Request who-has 10.0.10.40 tell 10.0.10.1, length 28
13:54:05.673114 ARP, Request who-has 10.0.10.41 tell 10.0.10.1, length 28
13:54:05.673115 ARP, Request who-has 10.0.10.42 tell 10.0.10.1, length 28
13:54:05.673115 ARP, Request who-has 10.0.10.45 tell 10.0.10.1, length 28
13:54:05.673116 ARP, Request who-has 10.0.10.46 tell 10.0.10.1, length 28
13:54:05.673116 ARP, Request who-has 10.0.10.47 tell 10.0.10.1, length 28
13:54:05.673117 ARP, Request who-has 10.0.10.48 tell 10.0.10.1, length 28
13:54:05.673118 ARP, Request who-has 10.0.10.49 tell 10.0.10.1, length 28
13:54:05.673118 ARP, Request who-has 10.0.10.50 tell 10.0.10.1, length 28
13:54:05.673119 ARP, Request who-has 10.0.10.51 tell 10.0.10.1, length 28
13:54:05.673120 ARP, Request who-has 10.0.10.52 tell 10.0.10.1, length 28
13:54:05.673120 ARP, Request who-has 10.0.10.53 tell 10.0.10.1, length 28
13:54:05.673121 ARP, Request who-has 10.0.10.54 tell 10.0.10.1, length 28
13:54:05.673121 ARP, Request who-has 10.0.10.55 tell 10.0.10.1, length 28
13:54:05.673122 ARP, Request who-has 10.0.10.56 tell 10.0.10.1, length 28
13:54:05.673123 ARP, Request who-has 10.0.10.57 tell 10.0.10.1, length 28
13:54:05.673123 ARP, Request who-has 10.0.10.58 tell 10.0.10.1, length 28
13:54:05.673124 ARP, Request who-has 10.0.10.59 tell 10.0.10.1, length 28
13:54:05.673125 ARP, Request who-has 10.0.10.61 tell 10.0.10.1, length 28
13:54:05.673125 ARP, Request who-has 10.0.10.62 tell 10.0.10.1, length 28
13:54:05.768982 ARP, Request who-has 10.0.10.189 tell 10.0.10.1, length 28
13:54:05.768985 ARP, Request who-has 10.0.10.190 tell 10.0.10.1, length 28
13:54:05.768986 ARP, Request who-has 10.0.10.195 tell 10.0.10.1, length 28
13:54:05.768986 ARP, Request who-has 10.0.10.198 tell 10.0.10.1, length 28
13:54:05.768987 ARP, Request who-has 10.0.10.204 tell 10.0.10.1, length 28
13:54:05.768988 ARP, Request who-has 10.0.10.208 tell 10.0.10.1, length 28
13:54:05.768988 ARP, Request who-has 10.0.10.212 tell 10.0.10.1, length 28
13:54:05.768989 ARP, Request who-has 10.0.10.216 tell 10.0.10.1, length 28
13:54:05.768990 ARP, Request who-has 10.0.10.220 tell 10.0.10.1, length 28
13:54:05.768990 ARP, Request who-has 10.0.10.221 tell 10.0.10.1, length 28
13:54:05.768991 ARP, Request who-has 10.0.10.223 tell 10.0.10.1, length 28
13:54:05.768991 ARP, Request who-has 10.0.10.224 tell 10.0.10.1, length 28
13:54:05.768992 ARP, Request who-has 10.0.10.226 tell 10.0.10.1, length 28
13:54:05.768993 ARP, Request who-has 10.0.10.227 tell 10.0.10.1, length 28
13:54:05.768993 ARP, Request who-has 10.0.10.230 tell 10.0.10.1, length 28
13:54:05.768994 ARP, Request who-has 10.0.10.231 tell 10.0.10.1, length 28
13:54:05.768995 ARP, Request who-has 10.0.10.247 tell 10.0.10.1, length 28
13:54:05.768995 ARP, Request who-has 10.0.10.33 tell 10.0.10.1, length 28
13:54:05.768996 ARP, Request who-has 10.0.10.43 tell 10.0.10.1, length 28
13:54:05.768997 ARP, Request who-has 10.0.10.44 tell 10.0.10.1, length 28
13:54:05.768997 ARP, Request who-has 10.0.10.67 tell 10.0.10.1, length 28
13:54:05.897167 ARP, Request who-has 10.0.10.192 tell 10.0.10.1, length 28
13:54:05.897170 ARP, Request who-has 10.0.10.201 tell 10.0.10.1, length 28
13:54:05.897171 ARP, Request who-has 10.0.10.203 tell 10.0.10.1, length 28
13:54:05.897171 ARP, Request who-has 10.0.10.90 tell 10.0.10.1, length 28
13:54:05.993145 ARP, Request who-has 10.0.10.248 tell 10.0.10.1, length 28
13:54:07.432845 ARP, Request who-has 10.0.10.1 tell 10.0.10.2, length 28
13:54:07.432890 ARP, Request who-has 10.0.10.2 tell 10.0.10.1, length 28
13:54:07.432893 ARP, Reply 10.0.10.2 is-at 00:50:56:ad:0e:33 (oui Unknown), length 28
13:54:07.432898 ARP, Reply 10.0.10.1 is-at 00:50:56:94:55:70 (oui Unknown), length 28
```

This generated a lot of broadcast traffic! For practice, see if you can find
the ARP request and response packets for `host1` and `host2` in the packet
capture output. We can tell that they were properly discovered in the output from
`host1`.

Now check the output on `host4`, which should pick up the above packets if it
was still on the same broadcasting domain:

```
eron@host4:~$ sudo tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

Still nothing? Awesome, there should be no packets yet if everything has been set
up right.

Now we're going to perform a quick scan of the other subnet and examine the results.
Starting on `host3`, run the following:

```
nmap -sn 10.0.10.0/24
```

When `nmap` is finished, cancel the packet capture on `host4`, and then check the
output on both `host3` and `host4`:

```
eron@host3:~$ nmap -sn 10.0.20.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2023-06-27 13:54 UTC
Nmap scan report for 10.0.20.3
Host is up (0.00052s latency).
Nmap scan report for 10.0.20.4
Host is up (0.00040s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 15.91 seconds
```

```
eron@host4:~$ sudo tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
13:54:49.924495 ARP, Request who-has 10.0.20.1 tell 10.0.20.3, length 28
13:54:49.924534 ARP, Request who-has 10.0.20.2 tell 10.0.20.3, length 28
13:54:49.924579 ARP, Request who-has 10.0.20.4 tell 10.0.20.3, length 28
13:54:49.924593 ARP, Reply 10.0.20.4 is-at 00:50:56:ad:24:4a (oui Unknown), length 28
13:54:49.924613 IP 10.0.20.3.57422 > 10.0.20.4.http: Flags [S], seq 2960543896, win 64240, options [mss 1460,sackOK,TS val 230221782 ecr 0,nop,wscale 7], length 0
13:54:49.924619 IP 10.0.20.4.http > 10.0.20.3.57422: Flags [R.], seq 0, ack 2960543897, win 0, length 0
13:54:49.924956 ARP, Request who-has 10.0.20.5 tell 10.0.20.3, length 28
13:54:49.924977 ARP, Request who-has 10.0.20.6 tell 10.0.20.3, length 28
13:54:49.924995 ARP, Request who-has 10.0.20.7 tell 10.0.20.3, length 28
13:54:49.925013 ARP, Request who-has 10.0.20.8 tell 10.0.20.3, length 28
13:54:49.925030 ARP, Request who-has 10.0.20.9 tell 10.0.20.3, length 28
13:54:49.925047 ARP, Request who-has 10.0.20.10 tell 10.0.20.3, length 28
13:54:49.925135 ARP, Request who-has 10.0.20.13 tell 10.0.20.3, length 28
13:54:49.925151 ARP, Request who-has 10.0.20.14 tell 10.0.20.3, length 28
13:54:49.925182 ARP, Request who-has 10.0.20.15 tell 10.0.20.3, length 28
13:54:49.925199 ARP, Request who-has 10.0.20.16 tell 10.0.20.3, length 28
13:54:50.025457 ARP, Request who-has 10.0.20.19 tell 10.0.20.3, length 28
13:54:50.025481 ARP, Request who-has 10.0.20.20 tell 10.0.20.3, length 28
13:54:50.025504 ARP, Request who-has 10.0.20.21 tell 10.0.20.3, length 28
13:54:50.025522 ARP, Request who-has 10.0.20.22 tell 10.0.20.3, length 28
13:54:50.025540 ARP, Request who-has 10.0.20.23 tell 10.0.20.3, length 28
13:54:50.025557 ARP, Request who-has 10.0.20.24 tell 10.0.20.3, length 28
13:54:50.025576 ARP, Request who-has 10.0.20.25 tell 10.0.20.3, length 28
13:54:50.025594 ARP, Request who-has 10.0.20.26 tell 10.0.20.3, length 28
13:54:50.025611 ARP, Request who-has 10.0.20.27 tell 10.0.20.3, length 28
13:54:50.025629 ARP, Request who-has 10.0.20.28 tell 10.0.20.3, length 28
13:54:50.025646 ARP, Request who-has 10.0.20.29 tell 10.0.20.3, length 28
13:54:50.025664 ARP, Request who-has 10.0.20.30 tell 10.0.20.3, length 28
13:54:50.125512 ARP, Request who-has 10.0.20.93 tell 10.0.20.3, length 28
13:54:50.125582 ARP, Request who-has 10.0.20.96 tell 10.0.20.3, length 28
13:54:50.125602 ARP, Request who-has 10.0.20.97 tell 10.0.20.3, length 28
13:54:50.125619 ARP, Request who-has 10.0.20.98 tell 10.0.20.3, length 28
13:54:50.125690 ARP, Request who-has 10.0.20.101 tell 10.0.20.3, length 28
13:54:50.125708 ARP, Request who-has 10.0.20.102 tell 10.0.20.3, length 28
13:54:50.125725 ARP, Request who-has 10.0.20.103 tell 10.0.20.3, length 28
13:54:50.125743 ARP, Request who-has 10.0.20.104 tell 10.0.20.3, length 28
13:54:50.125760 ARP, Request who-has 10.0.20.105 tell 10.0.20.3, length 28
13:54:50.125778 ARP, Request who-has 10.0.20.106 tell 10.0.20.3, length 28
13:54:50.125839 ARP, Request who-has 10.0.20.109 tell 10.0.20.3, length 28
13:54:50.125858 ARP, Request who-has 10.0.20.110 tell 10.0.20.3, length 28
13:54:50.225570 ARP, Request who-has 10.0.20.163 tell 10.0.20.3, length 28
13:54:50.225637 ARP, Request who-has 10.0.20.166 tell 10.0.20.3, length 28
13:54:50.225701 ARP, Request who-has 10.0.20.169 tell 10.0.20.3, length 28
13:54:50.225719 ARP, Request who-has 10.0.20.170 tell 10.0.20.3, length 28
13:54:50.225783 ARP, Request who-has 10.0.20.173 tell 10.0.20.3, length 28
13:54:50.225801 ARP, Request who-has 10.0.20.174 tell 10.0.20.3, length 28
13:54:50.225818 ARP, Request who-has 10.0.20.175 tell 10.0.20.3, length 28
13:54:50.225900 ARP, Request who-has 10.0.20.178 tell 10.0.20.3, length 28
13:54:50.225918 ARP, Request who-has 10.0.20.179 tell 10.0.20.3, length 28
13:54:50.225936 ARP, Request who-has 10.0.20.180 tell 10.0.20.3, length 28
13:54:50.225952 ARP, Request who-has 10.0.20.181 tell 10.0.20.3, length 28
13:54:50.226013 ARP, Request who-has 10.0.20.184 tell 10.0.20.3, length 28
13:54:50.325608 ARP, Request who-has 10.0.20.223 tell 10.0.20.3, length 28
13:54:50.325675 ARP, Request who-has 10.0.20.226 tell 10.0.20.3, length 28
13:54:50.325737 ARP, Request who-has 10.0.20.229 tell 10.0.20.3, length 28
13:54:50.325797 ARP, Request who-has 10.0.20.232 tell 10.0.20.3, length 28
13:54:50.325861 ARP, Request who-has 10.0.20.235 tell 10.0.20.3, length 28
13:54:50.325880 ARP, Request who-has 10.0.20.236 tell 10.0.20.3, length 28
13:54:50.325897 ARP, Request who-has 10.0.20.237 tell 10.0.20.3, length 28
13:54:50.325960 ARP, Request who-has 10.0.20.240 tell 10.0.20.3, length 28
13:54:50.325979 ARP, Request who-has 10.0.20.241 tell 10.0.20.3, length 28
13:54:50.326040 ARP, Request who-has 10.0.20.244 tell 10.0.20.3, length 28
13:54:50.326059 ARP, Request who-has 10.0.20.245 tell 10.0.20.3, length 28
13:54:50.326118 ARP, Request who-has 10.0.20.248 tell 10.0.20.3, length 28
13:54:50.425662 ARP, Request who-has 10.0.20.31 tell 10.0.20.3, length 28
13:54:50.425729 ARP, Request who-has 10.0.20.34 tell 10.0.20.3, length 28
13:54:50.425791 ARP, Request who-has 10.0.20.37 tell 10.0.20.3, length 28
13:54:50.425851 ARP, Request who-has 10.0.20.40 tell 10.0.20.3, length 28
13:54:50.425911 ARP, Request who-has 10.0.20.43 tell 10.0.20.3, length 28
13:54:50.425972 ARP, Request who-has 10.0.20.46 tell 10.0.20.3, length 28
13:54:50.425990 ARP, Request who-has 10.0.20.47 tell 10.0.20.3, length 28
13:54:50.426052 ARP, Request who-has 10.0.20.50 tell 10.0.20.3, length 28
13:54:50.426070 ARP, Request who-has 10.0.20.51 tell 10.0.20.3, length 28
13:54:50.426132 ARP, Request who-has 10.0.20.54 tell 10.0.20.3, length 28
13:54:50.426151 ARP, Request who-has 10.0.20.55 tell 10.0.20.3, length 28
13:54:50.426216 ARP, Request who-has 10.0.20.58 tell 10.0.20.3, length 28
13:54:50.525719 ARP, Request who-has 10.0.20.87 tell 10.0.20.3, length 28
13:54:50.525786 ARP, Request who-has 10.0.20.90 tell 10.0.20.3, length 28
13:54:50.525849 ARP, Request who-has 10.0.20.94 tell 10.0.20.3, length 28
13:54:50.525910 ARP, Request who-has 10.0.20.99 tell 10.0.20.3, length 28
13:54:50.525970 ARP, Request who-has 10.0.20.107 tell 10.0.20.3, length 28
13:54:50.526032 ARP, Request who-has 10.0.20.111 tell 10.0.20.3, length 28
13:54:50.526050 ARP, Request who-has 10.0.20.112 tell 10.0.20.3, length 28
13:54:50.526111 ARP, Request who-has 10.0.20.115 tell 10.0.20.3, length 28
13:54:50.526129 ARP, Request who-has 10.0.20.116 tell 10.0.20.3, length 28
13:54:50.526191 ARP, Request who-has 10.0.20.119 tell 10.0.20.3, length 28
13:54:50.526209 ARP, Request who-has 10.0.20.120 tell 10.0.20.3, length 28
13:54:50.526269 ARP, Request who-has 10.0.20.123 tell 10.0.20.3, length 28
13:54:50.625771 ARP, Request who-has 10.0.20.152 tell 10.0.20.3, length 28
13:54:50.625838 ARP, Request who-has 10.0.20.155 tell 10.0.20.3, length 28
13:54:50.625900 ARP, Request who-has 10.0.20.158 tell 10.0.20.3, length 28
13:54:50.625965 ARP, Request who-has 10.0.20.161 tell 10.0.20.3, length 28
13:54:50.626026 ARP, Request who-has 10.0.20.164 tell 10.0.20.3, length 28
13:54:50.626086 ARP, Request who-has 10.0.20.167 tell 10.0.20.3, length 28
13:54:50.626148 ARP, Request who-has 10.0.20.171 tell 10.0.20.3, length 28
13:54:50.626166 ARP, Request who-has 10.0.20.172 tell 10.0.20.3, length 28
13:54:50.626227 ARP, Request who-has 10.0.20.176 tell 10.0.20.3, length 28
13:54:50.626245 ARP, Request who-has 10.0.20.177 tell 10.0.20.3, length 28
13:54:50.626306 ARP, Request who-has 10.0.20.182 tell 10.0.20.3, length 28
13:54:50.626324 ARP, Request who-has 10.0.20.183 tell 10.0.20.3, length 28
13:54:50.725827 ARP, Request who-has 10.0.20.212 tell 10.0.20.3, length 28
13:54:50.725894 ARP, Request who-has 10.0.20.215 tell 10.0.20.3, length 28
13:54:50.725955 ARP, Request who-has 10.0.20.218 tell 10.0.20.3, length 28
13:54:50.726022 ARP, Request who-has 10.0.20.221 tell 10.0.20.3, length 28
13:54:50.726083 ARP, Request who-has 10.0.20.224 tell 10.0.20.3, length 28
13:54:50.726143 ARP, Request who-has 10.0.20.227 tell 10.0.20.3, length 28
13:54:50.726205 ARP, Request who-has 10.0.20.230 tell 10.0.20.3, length 28
13:54:50.726226 ARP, Request who-has 10.0.20.231 tell 10.0.20.3, length 28
13:54:50.726289 ARP, Request who-has 10.0.20.234 tell 10.0.20.3, length 28
13:54:50.726307 ARP, Request who-has 10.0.20.238 tell 10.0.20.3, length 28
13:54:50.726369 ARP, Request who-has 10.0.20.242 tell 10.0.20.3, length 28
13:54:50.726388 ARP, Request who-has 10.0.20.243 tell 10.0.20.3, length 28
13:54:50.825885 ARP, Request who-has 10.0.20.18 tell 10.0.20.3, length 28
13:54:50.825956 ARP, Request who-has 10.0.20.32 tell 10.0.20.3, length 28
13:54:50.826018 ARP, Request who-has 10.0.20.35 tell 10.0.20.3, length 28
13:54:50.826079 ARP, Request who-has 10.0.20.38 tell 10.0.20.3, length 28
13:54:50.826141 ARP, Request who-has 10.0.20.41 tell 10.0.20.3, length 28
13:54:50.826203 ARP, Request who-has 10.0.20.44 tell 10.0.20.3, length 28
13:54:50.826265 ARP, Request who-has 10.0.20.48 tell 10.0.20.3, length 28
13:54:50.826283 ARP, Request who-has 10.0.20.49 tell 10.0.20.3, length 28
13:54:50.826342 ARP, Request who-has 10.0.20.52 tell 10.0.20.3, length 28
13:54:50.826406 ARP, Request who-has 10.0.20.56 tell 10.0.20.3, length 28
13:54:50.826469 ARP, Request who-has 10.0.20.59 tell 10.0.20.3, length 28
13:54:50.826489 ARP, Request who-has 10.0.20.60 tell 10.0.20.3, length 28
13:54:50.925934 ARP, Request who-has 10.0.20.83 tell 10.0.20.3, length 28
13:54:50.926002 ARP, Request who-has 10.0.20.86 tell 10.0.20.3, length 28
13:54:50.926065 ARP, Request who-has 10.0.20.89 tell 10.0.20.3, length 28
13:54:50.926126 ARP, Request who-has 10.0.20.92 tell 10.0.20.3, length 28
13:54:50.926189 ARP, Request who-has 10.0.20.95 tell 10.0.20.3, length 28
13:54:50.926249 ARP, Request who-has 10.0.20.100 tell 10.0.20.3, length 28
13:54:50.926309 ARP, Request who-has 10.0.20.108 tell 10.0.20.3, length 28
13:54:50.926370 ARP, Request who-has 10.0.20.113 tell 10.0.20.3, length 28
13:54:50.926437 ARP, Request who-has 10.0.20.117 tell 10.0.20.3, length 28
13:54:50.926498 ARP, Request who-has 10.0.20.121 tell 10.0.20.3, length 28
13:54:50.926561 ARP, Request who-has 10.0.20.124 tell 10.0.20.3, length 28
13:54:50.926582 ARP, Request who-has 10.0.20.125 tell 10.0.20.3, length 28
13:54:50.952936 ARP, Request who-has 10.0.20.16 tell 10.0.20.3, length 28
13:54:50.952937 ARP, Request who-has 10.0.20.15 tell 10.0.20.3, length 28
13:54:50.952938 ARP, Request who-has 10.0.20.14 tell 10.0.20.3, length 28
13:54:50.952938 ARP, Request who-has 10.0.20.13 tell 10.0.20.3, length 28
13:54:50.952939 ARP, Request who-has 10.0.20.10 tell 10.0.20.3, length 28
13:54:50.952940 ARP, Request who-has 10.0.20.9 tell 10.0.20.3, length 28
13:54:50.952940 ARP, Request who-has 10.0.20.8 tell 10.0.20.3, length 28
13:54:50.952941 ARP, Request who-has 10.0.20.7 tell 10.0.20.3, length 28
13:54:50.952942 ARP, Request who-has 10.0.20.6 tell 10.0.20.3, length 28
13:54:50.952942 ARP, Request who-has 10.0.20.5 tell 10.0.20.3, length 28
13:54:50.952943 ARP, Request who-has 10.0.20.2 tell 10.0.20.3, length 28
13:54:50.952943 ARP, Request who-has 10.0.20.1 tell 10.0.20.3, length 28
13:54:51.048924 ARP, Request who-has 10.0.20.30 tell 10.0.20.3, length 28
13:54:51.048926 ARP, Request who-has 10.0.20.29 tell 10.0.20.3, length 28
13:54:51.048927 ARP, Request who-has 10.0.20.28 tell 10.0.20.3, length 28
13:54:51.048928 ARP, Request who-has 10.0.20.27 tell 10.0.20.3, length 28
13:54:51.048928 ARP, Request who-has 10.0.20.26 tell 10.0.20.3, length 28
13:54:51.048929 ARP, Request who-has 10.0.20.25 tell 10.0.20.3, length 28
13:54:51.048930 ARP, Request who-has 10.0.20.24 tell 10.0.20.3, length 28
13:54:51.048930 ARP, Request who-has 10.0.20.23 tell 10.0.20.3, length 28
13:54:51.048931 ARP, Request who-has 10.0.20.22 tell 10.0.20.3, length 28
13:54:51.048931 ARP, Request who-has 10.0.20.21 tell 10.0.20.3, length 28
13:54:51.048932 ARP, Request who-has 10.0.20.20 tell 10.0.20.3, length 28
13:54:51.048933 ARP, Request who-has 10.0.20.19 tell 10.0.20.3, length 28
13:54:51.144945 ARP, Request who-has 10.0.20.110 tell 10.0.20.3, length 28
13:54:51.144949 ARP, Request who-has 10.0.20.109 tell 10.0.20.3, length 28
13:54:51.144949 ARP, Request who-has 10.0.20.106 tell 10.0.20.3, length 28
13:54:51.144950 ARP, Request who-has 10.0.20.105 tell 10.0.20.3, length 28
13:54:51.144950 ARP, Request who-has 10.0.20.104 tell 10.0.20.3, length 28
13:54:51.144951 ARP, Request who-has 10.0.20.103 tell 10.0.20.3, length 28
13:54:51.144951 ARP, Request who-has 10.0.20.102 tell 10.0.20.3, length 28
13:54:51.144952 ARP, Request who-has 10.0.20.101 tell 10.0.20.3, length 28
13:54:51.144953 ARP, Request who-has 10.0.20.98 tell 10.0.20.3, length 28
13:54:51.144953 ARP, Request who-has 10.0.20.97 tell 10.0.20.3, length 28
13:54:51.144954 ARP, Request who-has 10.0.20.96 tell 10.0.20.3, length 28
13:54:51.144954 ARP, Request who-has 10.0.20.93 tell 10.0.20.3, length 28
13:54:51.226935 IP 10.0.20.3.57436 > 10.0.20.4.http: Flags [S], seq 2574521844, win 64240, options [mss 1460,sackOK,TS val 230223085 ecr 0,nop,wscale 7], length 0
13:54:51.226947 IP 10.0.20.4.http > 10.0.20.3.57436: Flags [R.], seq 0, ack 2574521845, win 0, length 0
13:54:51.227247 ARP, Request who-has 10.0.20.11 tell 10.0.20.3, length 28
13:54:51.227269 ARP, Request who-has 10.0.20.12 tell 10.0.20.3, length 28
13:54:51.227322 ARP, Request who-has 10.0.20.17 tell 10.0.20.3, length 28
13:54:51.227454 ARP, Request who-has 10.0.20.33 tell 10.0.20.3, length 28
13:54:51.227471 ARP, Request who-has 10.0.20.36 tell 10.0.20.3, length 28
13:54:51.227488 ARP, Request who-has 10.0.20.39 tell 10.0.20.3, length 28
13:54:51.227504 ARP, Request who-has 10.0.20.42 tell 10.0.20.3, length 28
13:54:51.227521 ARP, Request who-has 10.0.20.45 tell 10.0.20.3, length 28
13:54:51.227542 ARP, Request who-has 10.0.20.53 tell 10.0.20.3, length 28
13:54:51.227559 ARP, Request who-has 10.0.20.57 tell 10.0.20.3, length 28
13:54:51.227578 ARP, Request who-has 10.0.20.61 tell 10.0.20.3, length 28
13:54:51.227594 ARP, Request who-has 10.0.20.62 tell 10.0.20.3, length 28
13:54:51.227614 ARP, Request who-has 10.0.20.63 tell 10.0.20.3, length 28
13:54:51.227640 ARP, Request who-has 10.0.20.64 tell 10.0.20.3, length 28
13:54:51.227667 ARP, Request who-has 10.0.20.65 tell 10.0.20.3, length 28
13:54:51.227682 ARP, Request who-has 10.0.20.66 tell 10.0.20.3, length 28
13:54:51.227697 ARP, Request who-has 10.0.20.67 tell 10.0.20.3, length 28
13:54:51.227817 ARP, Request who-has 10.0.20.70 tell 10.0.20.3, length 28
13:54:51.227848 ARP, Request who-has 10.0.20.71 tell 10.0.20.3, length 28
13:54:51.227870 ARP, Request who-has 10.0.20.72 tell 10.0.20.3, length 28
13:54:51.227888 ARP, Request who-has 10.0.20.73 tell 10.0.20.3, length 28
13:54:51.227908 ARP, Request who-has 10.0.20.74 tell 10.0.20.3, length 28
13:54:51.227927 ARP, Request who-has 10.0.20.75 tell 10.0.20.3, length 28
13:54:51.227945 ARP, Request who-has 10.0.20.76 tell 10.0.20.3, length 28
13:54:51.227963 ARP, Request who-has 10.0.20.77 tell 10.0.20.3, length 28
13:54:51.227981 ARP, Request who-has 10.0.20.78 tell 10.0.20.3, length 28
13:54:51.228003 ARP, Request who-has 10.0.20.79 tell 10.0.20.3, length 28
13:54:51.228037 ARP, Request who-has 10.0.20.80 tell 10.0.20.3, length 28
13:54:51.228055 ARP, Request who-has 10.0.20.81 tell 10.0.20.3, length 28
13:54:51.228081 ARP, Request who-has 10.0.20.82 tell 10.0.20.3, length 28
13:54:51.228097 ARP, Request who-has 10.0.20.84 tell 10.0.20.3, length 28
13:54:51.228111 ARP, Request who-has 10.0.20.85 tell 10.0.20.3, length 28
13:54:51.228126 ARP, Request who-has 10.0.20.88 tell 10.0.20.3, length 28
13:54:51.228141 ARP, Request who-has 10.0.20.91 tell 10.0.20.3, length 28
13:54:51.240943 ARP, Request who-has 10.0.20.184 tell 10.0.20.3, length 28
13:54:51.240946 ARP, Request who-has 10.0.20.181 tell 10.0.20.3, length 28
13:54:51.240956 ARP, Request who-has 10.0.20.180 tell 10.0.20.3, length 28
13:54:51.240957 ARP, Request who-has 10.0.20.179 tell 10.0.20.3, length 28
13:54:51.240957 ARP, Request who-has 10.0.20.178 tell 10.0.20.3, length 28
13:54:51.240958 ARP, Request who-has 10.0.20.175 tell 10.0.20.3, length 28
13:54:51.240959 ARP, Request who-has 10.0.20.174 tell 10.0.20.3, length 28
13:54:51.240959 ARP, Request who-has 10.0.20.173 tell 10.0.20.3, length 28
13:54:51.240960 ARP, Request who-has 10.0.20.170 tell 10.0.20.3, length 28
13:54:51.240960 ARP, Request who-has 10.0.20.169 tell 10.0.20.3, length 28
13:54:51.240961 ARP, Request who-has 10.0.20.166 tell 10.0.20.3, length 28
13:54:51.240962 ARP, Request who-has 10.0.20.163 tell 10.0.20.3, length 28
13:54:51.327457 ARP, Request who-has 10.0.20.126 tell 10.0.20.3, length 28
13:54:51.327490 ARP, Request who-has 10.0.20.127 tell 10.0.20.3, length 28
13:54:51.327781 ARP, Request who-has 10.0.20.130 tell 10.0.20.3, length 28
13:54:51.327809 ARP, Request who-has 10.0.20.131 tell 10.0.20.3, length 28
13:54:51.327827 ARP, Request who-has 10.0.20.132 tell 10.0.20.3, length 28
13:54:51.327843 ARP, Request who-has 10.0.20.133 tell 10.0.20.3, length 28
13:54:51.327864 ARP, Request who-has 10.0.20.134 tell 10.0.20.3, length 28
13:54:51.328013 ARP, Request who-has 10.0.20.137 tell 10.0.20.3, length 28
13:54:51.328053 ARP, Request who-has 10.0.20.138 tell 10.0.20.3, length 28
13:54:51.328079 ARP, Request who-has 10.0.20.139 tell 10.0.20.3, length 28
13:54:51.328095 ARP, Request who-has 10.0.20.140 tell 10.0.20.3, length 28
13:54:51.328111 ARP, Request who-has 10.0.20.141 tell 10.0.20.3, length 28
13:54:51.328127 ARP, Request who-has 10.0.20.142 tell 10.0.20.3, length 28
13:54:51.328143 ARP, Request who-has 10.0.20.143 tell 10.0.20.3, length 28
13:54:51.328160 ARP, Request who-has 10.0.20.144 tell 10.0.20.3, length 28
13:54:51.328178 ARP, Request who-has 10.0.20.145 tell 10.0.20.3, length 28
13:54:51.328194 ARP, Request who-has 10.0.20.146 tell 10.0.20.3, length 28
13:54:51.328211 ARP, Request who-has 10.0.20.147 tell 10.0.20.3, length 28
13:54:51.328226 ARP, Request who-has 10.0.20.148 tell 10.0.20.3, length 28
13:54:51.328242 ARP, Request who-has 10.0.20.149 tell 10.0.20.3, length 28
13:54:51.328258 ARP, Request who-has 10.0.20.150 tell 10.0.20.3, length 28
13:54:51.328279 ARP, Request who-has 10.0.20.151 tell 10.0.20.3, length 28
13:54:51.328490 ARP, Request who-has 10.0.20.154 tell 10.0.20.3, length 28
13:54:51.328509 ARP, Request who-has 10.0.20.156 tell 10.0.20.3, length 28
13:54:51.328525 ARP, Request who-has 10.0.20.157 tell 10.0.20.3, length 28
13:54:51.328541 ARP, Request who-has 10.0.20.159 tell 10.0.20.3, length 28
13:54:51.328557 ARP, Request who-has 10.0.20.160 tell 10.0.20.3, length 28
13:54:51.328581 ARP, Request who-has 10.0.20.162 tell 10.0.20.3, length 28
13:54:51.328606 ARP, Request who-has 10.0.20.165 tell 10.0.20.3, length 28
13:54:51.328630 ARP, Request who-has 10.0.20.168 tell 10.0.20.3, length 28
13:54:51.336932 ARP, Request who-has 10.0.20.248 tell 10.0.20.3, length 28
13:54:51.336934 ARP, Request who-has 10.0.20.245 tell 10.0.20.3, length 28
13:54:51.336935 ARP, Request who-has 10.0.20.244 tell 10.0.20.3, length 28
13:54:51.336935 ARP, Request who-has 10.0.20.241 tell 10.0.20.3, length 28
13:54:51.336936 ARP, Request who-has 10.0.20.240 tell 10.0.20.3, length 28
13:54:51.336946 ARP, Request who-has 10.0.20.237 tell 10.0.20.3, length 28
13:54:51.336947 ARP, Request who-has 10.0.20.236 tell 10.0.20.3, length 28
13:54:51.336948 ARP, Request who-has 10.0.20.235 tell 10.0.20.3, length 28
13:54:51.336948 ARP, Request who-has 10.0.20.232 tell 10.0.20.3, length 28
13:54:51.336949 ARP, Request who-has 10.0.20.229 tell 10.0.20.3, length 28
13:54:51.336949 ARP, Request who-has 10.0.20.226 tell 10.0.20.3, length 28
13:54:51.336950 ARP, Request who-has 10.0.20.223 tell 10.0.20.3, length 28
13:54:51.427504 ARP, Request who-has 10.0.20.199 tell 10.0.20.3, length 28
13:54:51.427547 ARP, Request who-has 10.0.20.200 tell 10.0.20.3, length 28
13:54:51.427661 ARP, Request who-has 10.0.20.203 tell 10.0.20.3, length 28
13:54:51.427699 ARP, Request who-has 10.0.20.204 tell 10.0.20.3, length 28
13:54:51.427734 ARP, Request who-has 10.0.20.205 tell 10.0.20.3, length 28
13:54:51.427763 ARP, Request who-has 10.0.20.206 tell 10.0.20.3, length 28
13:54:51.427793 ARP, Request who-has 10.0.20.207 tell 10.0.20.3, length 28
13:54:51.427945 ARP, Request who-has 10.0.20.210 tell 10.0.20.3, length 28
13:54:51.427981 ARP, Request who-has 10.0.20.211 tell 10.0.20.3, length 28
13:54:51.428003 ARP, Request who-has 10.0.20.213 tell 10.0.20.3, length 28
13:54:51.428019 ARP, Request who-has 10.0.20.214 tell 10.0.20.3, length 28
13:54:51.428036 ARP, Request who-has 10.0.20.216 tell 10.0.20.3, length 28
13:54:51.428071 ARP, Request who-has 10.0.20.217 tell 10.0.20.3, length 28
13:54:51.428088 ARP, Request who-has 10.0.20.219 tell 10.0.20.3, length 28
13:54:51.428105 ARP, Request who-has 10.0.20.220 tell 10.0.20.3, length 28
13:54:51.428124 ARP, Request who-has 10.0.20.222 tell 10.0.20.3, length 28
13:54:51.428149 ARP, Request who-has 10.0.20.225 tell 10.0.20.3, length 28
13:54:51.428174 ARP, Request who-has 10.0.20.228 tell 10.0.20.3, length 28
13:54:51.428205 ARP, Request who-has 10.0.20.233 tell 10.0.20.3, length 28
13:54:51.428384 ARP, Request who-has 10.0.20.246 tell 10.0.20.3, length 28
13:54:51.428401 ARP, Request who-has 10.0.20.247 tell 10.0.20.3, length 28
13:54:51.428425 ARP, Request who-has 10.0.20.249 tell 10.0.20.3, length 28
13:54:51.428443 ARP, Request who-has 10.0.20.250 tell 10.0.20.3, length 28
13:54:51.428460 ARP, Request who-has 10.0.20.251 tell 10.0.20.3, length 28
13:54:51.428476 ARP, Request who-has 10.0.20.252 tell 10.0.20.3, length 28
13:54:51.428496 ARP, Request who-has 10.0.20.253 tell 10.0.20.3, length 28
13:54:51.428513 ARP, Request who-has 10.0.20.254 tell 10.0.20.3, length 28
13:54:51.428542 ARP, Request who-has 10.0.20.0 tell 10.0.20.3, length 28
13:54:51.428687 ARP, Request who-has 10.0.20.68 tell 10.0.20.3, length 28
13:54:51.428706 ARP, Request who-has 10.0.20.69 tell 10.0.20.3, length 28
13:54:51.428757 ARP, Request who-has 10.0.20.114 tell 10.0.20.3, length 28
13:54:51.428775 ARP, Request who-has 10.0.20.118 tell 10.0.20.3, length 28
13:54:51.428795 ARP, Request who-has 10.0.20.122 tell 10.0.20.3, length 28
13:54:51.432294 ARP, Request who-has 10.0.20.128 tell 10.0.20.3, length 28
13:54:51.432327 ARP, Request who-has 10.0.20.129 tell 10.0.20.3, length 28
13:54:51.432353 ARP, Request who-has 10.0.20.135 tell 10.0.20.3, length 28
13:54:51.432384 ARP, Request who-has 10.0.20.136 tell 10.0.20.3, length 28
13:54:51.432402 ARP, Request who-has 10.0.20.153 tell 10.0.20.3, length 28
13:54:51.432897 ARP, Request who-has 10.0.20.58 tell 10.0.20.3, length 28
13:54:51.432899 ARP, Request who-has 10.0.20.55 tell 10.0.20.3, length 28
13:54:51.432900 ARP, Request who-has 10.0.20.54 tell 10.0.20.3, length 28
13:54:51.432900 ARP, Request who-has 10.0.20.51 tell 10.0.20.3, length 28
13:54:51.432901 ARP, Request who-has 10.0.20.50 tell 10.0.20.3, length 28
13:54:51.432902 ARP, Request who-has 10.0.20.47 tell 10.0.20.3, length 28
13:54:51.432902 ARP, Request who-has 10.0.20.46 tell 10.0.20.3, length 28
13:54:51.432903 ARP, Request who-has 10.0.20.43 tell 10.0.20.3, length 28
13:54:51.432903 ARP, Request who-has 10.0.20.40 tell 10.0.20.3, length 28
13:54:51.432904 ARP, Request who-has 10.0.20.37 tell 10.0.20.3, length 28
13:54:51.432905 ARP, Request who-has 10.0.20.34 tell 10.0.20.3, length 28
13:54:51.432905 ARP, Request who-has 10.0.20.31 tell 10.0.20.3, length 28
13:54:51.528083 ARP, Request who-has 10.0.20.239 tell 10.0.20.3, length 28
13:54:51.532392 ARP, Request who-has 10.0.20.194 tell 10.0.20.3, length 28
13:54:51.532479 ARP, Request who-has 10.0.20.197 tell 10.0.20.3, length 28
13:54:51.532500 ARP, Request who-has 10.0.20.198 tell 10.0.20.3, length 28
13:54:51.532518 ARP, Request who-has 10.0.20.201 tell 10.0.20.3, length 28
13:54:51.532537 ARP, Request who-has 10.0.20.202 tell 10.0.20.3, length 28
13:54:51.532614 ARP, Request who-has 10.0.20.208 tell 10.0.20.3, length 28
13:54:51.532637 ARP, Request who-has 10.0.20.209 tell 10.0.20.3, length 28
13:54:51.532893 ARP, Request who-has 10.0.20.123 tell 10.0.20.3, length 28
13:54:51.532894 ARP, Request who-has 10.0.20.120 tell 10.0.20.3, length 28
13:54:51.532895 ARP, Request who-has 10.0.20.119 tell 10.0.20.3, length 28
13:54:51.532895 ARP, Request who-has 10.0.20.116 tell 10.0.20.3, length 28
13:54:51.532896 ARP, Request who-has 10.0.20.115 tell 10.0.20.3, length 28
13:54:51.532897 ARP, Request who-has 10.0.20.112 tell 10.0.20.3, length 28
13:54:51.532897 ARP, Request who-has 10.0.20.111 tell 10.0.20.3, length 28
13:54:51.532898 ARP, Request who-has 10.0.20.107 tell 10.0.20.3, length 28
13:54:51.532899 ARP, Request who-has 10.0.20.99 tell 10.0.20.3, length 28
13:54:51.532899 ARP, Request who-has 10.0.20.94 tell 10.0.20.3, length 28
13:54:51.532900 ARP, Request who-has 10.0.20.90 tell 10.0.20.3, length 28
13:54:51.532900 ARP, Request who-has 10.0.20.87 tell 10.0.20.3, length 28
13:54:51.628025 ARP, Request who-has 10.0.20.186 tell 10.0.20.3, length 28
13:54:51.628094 ARP, Request who-has 10.0.20.189 tell 10.0.20.3, length 28
13:54:51.628160 ARP, Request who-has 10.0.20.192 tell 10.0.20.3, length 28
13:54:51.628228 ARP, Request who-has 10.0.20.195 tell 10.0.20.3, length 28
13:54:51.628610 ARP, Request who-has 10.0.20.185 tell 10.0.20.3, length 28
13:54:51.628751 ARP, Request who-has 10.0.20.188 tell 10.0.20.3, length 28
13:54:51.628924 ARP, Request who-has 10.0.20.191 tell 10.0.20.3, length 28
13:54:51.628958 ARP, Request who-has 10.0.20.193 tell 10.0.20.3, length 28
13:54:51.629091 ARP, Request who-has 10.0.20.187 tell 10.0.20.3, length 28
13:54:51.629131 ARP, Request who-has 10.0.20.190 tell 10.0.20.3, length 28
13:54:51.632603 ARP, Request who-has 10.0.20.196 tell 10.0.20.3, length 28
13:54:51.656931 ARP, Request who-has 10.0.20.183 tell 10.0.20.3, length 28
13:54:51.656932 ARP, Request who-has 10.0.20.182 tell 10.0.20.3, length 28
13:54:51.656933 ARP, Request who-has 10.0.20.177 tell 10.0.20.3, length 28
13:54:51.656934 ARP, Request who-has 10.0.20.176 tell 10.0.20.3, length 28
13:54:51.656934 ARP, Request who-has 10.0.20.172 tell 10.0.20.3, length 28
13:54:51.656935 ARP, Request who-has 10.0.20.171 tell 10.0.20.3, length 28
13:54:51.656936 ARP, Request who-has 10.0.20.167 tell 10.0.20.3, length 28
13:54:51.656936 ARP, Request who-has 10.0.20.164 tell 10.0.20.3, length 28
13:54:51.656937 ARP, Request who-has 10.0.20.161 tell 10.0.20.3, length 28
13:54:51.656938 ARP, Request who-has 10.0.20.158 tell 10.0.20.3, length 28
13:54:51.656938 ARP, Request who-has 10.0.20.155 tell 10.0.20.3, length 28
13:54:51.656939 ARP, Request who-has 10.0.20.152 tell 10.0.20.3, length 28
13:54:51.752909 ARP, Request who-has 10.0.20.243 tell 10.0.20.3, length 28
13:54:51.752911 ARP, Request who-has 10.0.20.242 tell 10.0.20.3, length 28
13:54:51.752912 ARP, Request who-has 10.0.20.238 tell 10.0.20.3, length 28
13:54:51.752912 ARP, Request who-has 10.0.20.234 tell 10.0.20.3, length 28
13:54:51.752913 ARP, Request who-has 10.0.20.231 tell 10.0.20.3, length 28
13:54:51.752913 ARP, Request who-has 10.0.20.230 tell 10.0.20.3, length 28
13:54:51.752914 ARP, Request who-has 10.0.20.227 tell 10.0.20.3, length 28
13:54:51.752915 ARP, Request who-has 10.0.20.224 tell 10.0.20.3, length 28
13:54:51.752915 ARP, Request who-has 10.0.20.221 tell 10.0.20.3, length 28
13:54:51.752916 ARP, Request who-has 10.0.20.218 tell 10.0.20.3, length 28
13:54:51.752917 ARP, Request who-has 10.0.20.215 tell 10.0.20.3, length 28
13:54:51.752917 ARP, Request who-has 10.0.20.212 tell 10.0.20.3, length 28
13:54:51.848938 ARP, Request who-has 10.0.20.60 tell 10.0.20.3, length 28
13:54:51.848940 ARP, Request who-has 10.0.20.59 tell 10.0.20.3, length 28
13:54:51.848941 ARP, Request who-has 10.0.20.56 tell 10.0.20.3, length 28
13:54:51.848942 ARP, Request who-has 10.0.20.52 tell 10.0.20.3, length 28
13:54:51.848942 ARP, Request who-has 10.0.20.49 tell 10.0.20.3, length 28
13:54:51.848943 ARP, Request who-has 10.0.20.48 tell 10.0.20.3, length 28
13:54:51.848944 ARP, Request who-has 10.0.20.44 tell 10.0.20.3, length 28
13:54:51.848944 ARP, Request who-has 10.0.20.41 tell 10.0.20.3, length 28
13:54:51.848945 ARP, Request who-has 10.0.20.38 tell 10.0.20.3, length 28
13:54:51.848946 ARP, Request who-has 10.0.20.35 tell 10.0.20.3, length 28
13:54:51.848946 ARP, Request who-has 10.0.20.32 tell 10.0.20.3, length 28
13:54:51.848947 ARP, Request who-has 10.0.20.18 tell 10.0.20.3, length 28
13:54:51.948923 ARP, Request who-has 10.0.20.125 tell 10.0.20.3, length 28
13:54:51.948925 ARP, Request who-has 10.0.20.124 tell 10.0.20.3, length 28
13:54:51.948925 ARP, Request who-has 10.0.20.121 tell 10.0.20.3, length 28
13:54:51.948926 ARP, Request who-has 10.0.20.117 tell 10.0.20.3, length 28
13:54:51.948927 ARP, Request who-has 10.0.20.113 tell 10.0.20.3, length 28
13:54:51.948927 ARP, Request who-has 10.0.20.108 tell 10.0.20.3, length 28
13:54:51.948928 ARP, Request who-has 10.0.20.100 tell 10.0.20.3, length 28
13:54:51.948929 ARP, Request who-has 10.0.20.95 tell 10.0.20.3, length 28
13:54:51.948929 ARP, Request who-has 10.0.20.92 tell 10.0.20.3, length 28
13:54:51.948930 ARP, Request who-has 10.0.20.89 tell 10.0.20.3, length 28
13:54:51.948931 ARP, Request who-has 10.0.20.86 tell 10.0.20.3, length 28
13:54:51.948931 ARP, Request who-has 10.0.20.83 tell 10.0.20.3, length 28
13:54:51.976931 ARP, Request who-has 10.0.20.1 tell 10.0.20.3, length 28
13:54:51.976933 ARP, Request who-has 10.0.20.2 tell 10.0.20.3, length 28
13:54:51.976934 ARP, Request who-has 10.0.20.5 tell 10.0.20.3, length 28
13:54:51.976934 ARP, Request who-has 10.0.20.6 tell 10.0.20.3, length 28
13:54:51.976935 ARP, Request who-has 10.0.20.7 tell 10.0.20.3, length 28
13:54:51.976936 ARP, Request who-has 10.0.20.8 tell 10.0.20.3, length 28
13:54:51.976936 ARP, Request who-has 10.0.20.9 tell 10.0.20.3, length 28
13:54:51.976937 ARP, Request who-has 10.0.20.10 tell 10.0.20.3, length 28
13:54:51.976938 ARP, Request who-has 10.0.20.13 tell 10.0.20.3, length 28
13:54:51.976938 ARP, Request who-has 10.0.20.14 tell 10.0.20.3, length 28
13:54:51.976939 ARP, Request who-has 10.0.20.15 tell 10.0.20.3, length 28
13:54:51.976939 ARP, Request who-has 10.0.20.16 tell 10.0.20.3, length 28
13:54:52.072939 ARP, Request who-has 10.0.20.19 tell 10.0.20.3, length 28
13:54:52.072942 ARP, Request who-has 10.0.20.20 tell 10.0.20.3, length 28
13:54:52.072943 ARP, Request who-has 10.0.20.21 tell 10.0.20.3, length 28
13:54:52.072944 ARP, Request who-has 10.0.20.22 tell 10.0.20.3, length 28
13:54:52.072944 ARP, Request who-has 10.0.20.23 tell 10.0.20.3, length 28
13:54:52.072945 ARP, Request who-has 10.0.20.24 tell 10.0.20.3, length 28
13:54:52.072945 ARP, Request who-has 10.0.20.25 tell 10.0.20.3, length 28
13:54:52.072946 ARP, Request who-has 10.0.20.26 tell 10.0.20.3, length 28
13:54:52.072946 ARP, Request who-has 10.0.20.27 tell 10.0.20.3, length 28
13:54:52.072947 ARP, Request who-has 10.0.20.28 tell 10.0.20.3, length 28
13:54:52.072948 ARP, Request who-has 10.0.20.29 tell 10.0.20.3, length 28
13:54:52.072948 ARP, Request who-has 10.0.20.30 tell 10.0.20.3, length 28
13:54:52.168928 ARP, Request who-has 10.0.20.93 tell 10.0.20.3, length 28
13:54:52.168932 ARP, Request who-has 10.0.20.96 tell 10.0.20.3, length 28
13:54:52.168932 ARP, Request who-has 10.0.20.97 tell 10.0.20.3, length 28
13:54:52.168933 ARP, Request who-has 10.0.20.98 tell 10.0.20.3, length 28
13:54:52.168934 ARP, Request who-has 10.0.20.101 tell 10.0.20.3, length 28
13:54:52.168934 ARP, Request who-has 10.0.20.102 tell 10.0.20.3, length 28
13:54:52.168935 ARP, Request who-has 10.0.20.103 tell 10.0.20.3, length 28
13:54:52.168935 ARP, Request who-has 10.0.20.104 tell 10.0.20.3, length 28
13:54:52.168936 ARP, Request who-has 10.0.20.105 tell 10.0.20.3, length 28
13:54:52.168937 ARP, Request who-has 10.0.20.106 tell 10.0.20.3, length 28
13:54:52.168937 ARP, Request who-has 10.0.20.109 tell 10.0.20.3, length 28
13:54:52.168938 ARP, Request who-has 10.0.20.110 tell 10.0.20.3, length 28
13:54:52.233031 ARP, Request who-has 10.0.20.91 tell 10.0.20.3, length 28
13:54:52.233033 ARP, Request who-has 10.0.20.88 tell 10.0.20.3, length 28
13:54:52.233034 ARP, Request who-has 10.0.20.85 tell 10.0.20.3, length 28
13:54:52.233035 ARP, Request who-has 10.0.20.84 tell 10.0.20.3, length 28
13:54:52.233035 ARP, Request who-has 10.0.20.82 tell 10.0.20.3, length 28
13:54:52.233036 ARP, Request who-has 10.0.20.81 tell 10.0.20.3, length 28
13:54:52.233037 ARP, Request who-has 10.0.20.80 tell 10.0.20.3, length 28
13:54:52.233037 ARP, Request who-has 10.0.20.79 tell 10.0.20.3, length 28
13:54:52.233038 ARP, Request who-has 10.0.20.78 tell 10.0.20.3, length 28
13:54:52.233039 ARP, Request who-has 10.0.20.77 tell 10.0.20.3, length 28
13:54:52.233039 ARP, Request who-has 10.0.20.76 tell 10.0.20.3, length 28
13:54:52.233040 ARP, Request who-has 10.0.20.75 tell 10.0.20.3, length 28
13:54:52.233041 ARP, Request who-has 10.0.20.74 tell 10.0.20.3, length 28
13:54:52.233041 ARP, Request who-has 10.0.20.73 tell 10.0.20.3, length 28
13:54:52.233042 ARP, Request who-has 10.0.20.72 tell 10.0.20.3, length 28
13:54:52.233043 ARP, Request who-has 10.0.20.71 tell 10.0.20.3, length 28
13:54:52.233043 ARP, Request who-has 10.0.20.70 tell 10.0.20.3, length 28
13:54:52.233044 ARP, Request who-has 10.0.20.67 tell 10.0.20.3, length 28
13:54:52.233045 ARP, Request who-has 10.0.20.66 tell 10.0.20.3, length 28
13:54:52.233046 ARP, Request who-has 10.0.20.65 tell 10.0.20.3, length 28
13:54:52.233047 ARP, Request who-has 10.0.20.64 tell 10.0.20.3, length 28
13:54:52.233047 ARP, Request who-has 10.0.20.63 tell 10.0.20.3, length 28
13:54:52.233048 ARP, Request who-has 10.0.20.62 tell 10.0.20.3, length 28
13:54:52.233048 ARP, Request who-has 10.0.20.61 tell 10.0.20.3, length 28
13:54:52.233049 ARP, Request who-has 10.0.20.57 tell 10.0.20.3, length 28
13:54:52.233050 ARP, Request who-has 10.0.20.53 tell 10.0.20.3, length 28
13:54:52.233050 ARP, Request who-has 10.0.20.45 tell 10.0.20.3, length 28
13:54:52.233051 ARP, Request who-has 10.0.20.42 tell 10.0.20.3, length 28
13:54:52.233052 ARP, Request who-has 10.0.20.39 tell 10.0.20.3, length 28
13:54:52.233052 ARP, Request who-has 10.0.20.36 tell 10.0.20.3, length 28
13:54:52.233053 ARP, Request who-has 10.0.20.33 tell 10.0.20.3, length 28
13:54:52.233054 ARP, Request who-has 10.0.20.17 tell 10.0.20.3, length 28
13:54:52.233054 ARP, Request who-has 10.0.20.12 tell 10.0.20.3, length 28
13:54:52.233055 ARP, Request who-has 10.0.20.11 tell 10.0.20.3, length 28
13:54:52.264917 ARP, Request who-has 10.0.20.163 tell 10.0.20.3, length 28
13:54:52.264918 ARP, Request who-has 10.0.20.166 tell 10.0.20.3, length 28
13:54:52.264919 ARP, Request who-has 10.0.20.169 tell 10.0.20.3, length 28
13:54:52.264920 ARP, Request who-has 10.0.20.170 tell 10.0.20.3, length 28
13:54:52.264920 ARP, Request who-has 10.0.20.173 tell 10.0.20.3, length 28
13:54:52.264921 ARP, Request who-has 10.0.20.174 tell 10.0.20.3, length 28
13:54:52.264925 ARP, Request who-has 10.0.20.175 tell 10.0.20.3, length 28
13:54:52.264925 ARP, Request who-has 10.0.20.178 tell 10.0.20.3, length 28
13:54:52.264926 ARP, Request who-has 10.0.20.179 tell 10.0.20.3, length 28
13:54:52.264927 ARP, Request who-has 10.0.20.180 tell 10.0.20.3, length 28
13:54:52.264927 ARP, Request who-has 10.0.20.181 tell 10.0.20.3, length 28
13:54:52.264928 ARP, Request who-has 10.0.20.184 tell 10.0.20.3, length 28
13:54:52.333031 ARP, Request who-has 10.0.20.168 tell 10.0.20.3, length 28
13:54:52.333032 ARP, Request who-has 10.0.20.165 tell 10.0.20.3, length 28
13:54:52.333033 ARP, Request who-has 10.0.20.162 tell 10.0.20.3, length 28
13:54:52.333034 ARP, Request who-has 10.0.20.160 tell 10.0.20.3, length 28
13:54:52.333034 ARP, Request who-has 10.0.20.159 tell 10.0.20.3, length 28
13:54:52.333035 ARP, Request who-has 10.0.20.157 tell 10.0.20.3, length 28
13:54:52.333036 ARP, Request who-has 10.0.20.156 tell 10.0.20.3, length 28
13:54:52.333036 ARP, Request who-has 10.0.20.154 tell 10.0.20.3, length 28
13:54:52.333037 ARP, Request who-has 10.0.20.151 tell 10.0.20.3, length 28
13:54:52.333038 ARP, Request who-has 10.0.20.150 tell 10.0.20.3, length 28
13:54:52.333039 ARP, Request who-has 10.0.20.149 tell 10.0.20.3, length 28
13:54:52.333039 ARP, Request who-has 10.0.20.148 tell 10.0.20.3, length 28
13:54:52.333040 ARP, Request who-has 10.0.20.147 tell 10.0.20.3, length 28
13:54:52.333040 ARP, Request who-has 10.0.20.146 tell 10.0.20.3, length 28
13:54:52.333041 ARP, Request who-has 10.0.20.145 tell 10.0.20.3, length 28
13:54:52.333042 ARP, Request who-has 10.0.20.144 tell 10.0.20.3, length 28
13:54:52.333042 ARP, Request who-has 10.0.20.143 tell 10.0.20.3, length 28
13:54:52.333043 ARP, Request who-has 10.0.20.142 tell 10.0.20.3, length 28
13:54:52.333044 ARP, Request who-has 10.0.20.141 tell 10.0.20.3, length 28
13:54:52.333044 ARP, Request who-has 10.0.20.140 tell 10.0.20.3, length 28
13:54:52.333045 ARP, Request who-has 10.0.20.139 tell 10.0.20.3, length 28
13:54:52.333045 ARP, Request who-has 10.0.20.138 tell 10.0.20.3, length 28
13:54:52.333046 ARP, Request who-has 10.0.20.137 tell 10.0.20.3, length 28
13:54:52.333047 ARP, Request who-has 10.0.20.134 tell 10.0.20.3, length 28
13:54:52.333048 ARP, Request who-has 10.0.20.133 tell 10.0.20.3, length 28
13:54:52.333048 ARP, Request who-has 10.0.20.132 tell 10.0.20.3, length 28
13:54:52.333049 ARP, Request who-has 10.0.20.131 tell 10.0.20.3, length 28
13:54:52.333049 ARP, Request who-has 10.0.20.130 tell 10.0.20.3, length 28
13:54:52.333050 ARP, Request who-has 10.0.20.127 tell 10.0.20.3, length 28
13:54:52.333051 ARP, Request who-has 10.0.20.126 tell 10.0.20.3, length 28
13:54:52.360921 ARP, Request who-has 10.0.20.223 tell 10.0.20.3, length 28
13:54:52.360922 ARP, Request who-has 10.0.20.226 tell 10.0.20.3, length 28
13:54:52.360923 ARP, Request who-has 10.0.20.229 tell 10.0.20.3, length 28
13:54:52.360924 ARP, Request who-has 10.0.20.232 tell 10.0.20.3, length 28
13:54:52.360924 ARP, Request who-has 10.0.20.235 tell 10.0.20.3, length 28
13:54:52.360925 ARP, Request who-has 10.0.20.236 tell 10.0.20.3, length 28
13:54:52.360926 ARP, Request who-has 10.0.20.237 tell 10.0.20.3, length 28
13:54:52.360926 ARP, Request who-has 10.0.20.240 tell 10.0.20.3, length 28
13:54:52.360927 ARP, Request who-has 10.0.20.241 tell 10.0.20.3, length 28
13:54:52.360928 ARP, Request who-has 10.0.20.244 tell 10.0.20.3, length 28
13:54:52.360928 ARP, Request who-has 10.0.20.245 tell 10.0.20.3, length 28
13:54:52.360929 ARP, Request who-has 10.0.20.248 tell 10.0.20.3, length 28
13:54:52.457147 ARP, Request who-has 10.0.20.31 tell 10.0.20.3, length 28
13:54:52.457149 ARP, Request who-has 10.0.20.34 tell 10.0.20.3, length 28
13:54:52.457149 ARP, Request who-has 10.0.20.37 tell 10.0.20.3, length 28
13:54:52.457150 ARP, Request who-has 10.0.20.40 tell 10.0.20.3, length 28
13:54:52.457151 ARP, Request who-has 10.0.20.43 tell 10.0.20.3, length 28
13:54:52.457152 ARP, Request who-has 10.0.20.46 tell 10.0.20.3, length 28
13:54:52.457152 ARP, Request who-has 10.0.20.47 tell 10.0.20.3, length 28
13:54:52.457153 ARP, Request who-has 10.0.20.50 tell 10.0.20.3, length 28
13:54:52.457154 ARP, Request who-has 10.0.20.51 tell 10.0.20.3, length 28
13:54:52.457154 ARP, Request who-has 10.0.20.54 tell 10.0.20.3, length 28
13:54:52.457155 ARP, Request who-has 10.0.20.55 tell 10.0.20.3, length 28
13:54:52.457156 ARP, Request who-has 10.0.20.58 tell 10.0.20.3, length 28
13:54:52.457156 ARP, Request who-has 10.0.20.153 tell 10.0.20.3, length 28
13:54:52.457157 ARP, Request who-has 10.0.20.136 tell 10.0.20.3, length 28
13:54:52.457158 ARP, Request who-has 10.0.20.135 tell 10.0.20.3, length 28
13:54:52.457158 ARP, Request who-has 10.0.20.129 tell 10.0.20.3, length 28
13:54:52.457159 ARP, Request who-has 10.0.20.128 tell 10.0.20.3, length 28
13:54:52.457160 ARP, Request who-has 10.0.20.122 tell 10.0.20.3, length 28
13:54:52.457160 ARP, Request who-has 10.0.20.118 tell 10.0.20.3, length 28
13:54:52.457161 ARP, Request who-has 10.0.20.114 tell 10.0.20.3, length 28
13:54:52.457162 ARP, Request who-has 10.0.20.69 tell 10.0.20.3, length 28
13:54:52.457162 ARP, Request who-has 10.0.20.68 tell 10.0.20.3, length 28
13:54:52.457163 ARP, Request who-has 10.0.20.0 tell 10.0.20.3, length 28
13:54:52.457164 ARP, Request who-has 10.0.20.254 tell 10.0.20.3, length 28
13:54:52.457165 ARP, Request who-has 10.0.20.253 tell 10.0.20.3, length 28
13:54:52.457165 ARP, Request who-has 10.0.20.252 tell 10.0.20.3, length 28
13:54:52.457166 ARP, Request who-has 10.0.20.251 tell 10.0.20.3, length 28
13:54:52.457166 ARP, Request who-has 10.0.20.250 tell 10.0.20.3, length 28
13:54:52.457167 ARP, Request who-has 10.0.20.249 tell 10.0.20.3, length 28
13:54:52.457168 ARP, Request who-has 10.0.20.247 tell 10.0.20.3, length 28
13:54:52.457168 ARP, Request who-has 10.0.20.246 tell 10.0.20.3, length 28
13:54:52.457169 ARP, Request who-has 10.0.20.233 tell 10.0.20.3, length 28
13:54:52.457170 ARP, Request who-has 10.0.20.228 tell 10.0.20.3, length 28
13:54:52.457170 ARP, Request who-has 10.0.20.225 tell 10.0.20.3, length 28
13:54:52.457171 ARP, Request who-has 10.0.20.222 tell 10.0.20.3, length 28
13:54:52.457172 ARP, Request who-has 10.0.20.220 tell 10.0.20.3, length 28
13:54:52.457172 ARP, Request who-has 10.0.20.219 tell 10.0.20.3, length 28
13:54:52.457173 ARP, Request who-has 10.0.20.217 tell 10.0.20.3, length 28
13:54:52.457174 ARP, Request who-has 10.0.20.216 tell 10.0.20.3, length 28
13:54:52.457174 ARP, Request who-has 10.0.20.214 tell 10.0.20.3, length 28
13:54:52.457175 ARP, Request who-has 10.0.20.213 tell 10.0.20.3, length 28
13:54:52.457176 ARP, Request who-has 10.0.20.211 tell 10.0.20.3, length 28
13:54:52.457177 ARP, Request who-has 10.0.20.210 tell 10.0.20.3, length 28
13:54:52.457177 ARP, Request who-has 10.0.20.207 tell 10.0.20.3, length 28
13:54:52.457178 ARP, Request who-has 10.0.20.206 tell 10.0.20.3, length 28
13:54:52.457178 ARP, Request who-has 10.0.20.205 tell 10.0.20.3, length 28
13:54:52.457179 ARP, Request who-has 10.0.20.204 tell 10.0.20.3, length 28
13:54:52.457180 ARP, Request who-has 10.0.20.203 tell 10.0.20.3, length 28
13:54:52.457180 ARP, Request who-has 10.0.20.200 tell 10.0.20.3, length 28
13:54:52.457181 ARP, Request who-has 10.0.20.199 tell 10.0.20.3, length 28
13:54:52.552958 ARP, Request who-has 10.0.20.87 tell 10.0.20.3, length 28
13:54:52.552959 ARP, Request who-has 10.0.20.90 tell 10.0.20.3, length 28
13:54:52.552960 ARP, Request who-has 10.0.20.94 tell 10.0.20.3, length 28
13:54:52.552961 ARP, Request who-has 10.0.20.99 tell 10.0.20.3, length 28
13:54:52.552962 ARP, Request who-has 10.0.20.107 tell 10.0.20.3, length 28
13:54:52.552962 ARP, Request who-has 10.0.20.111 tell 10.0.20.3, length 28
13:54:52.552963 ARP, Request who-has 10.0.20.112 tell 10.0.20.3, length 28
13:54:52.552964 ARP, Request who-has 10.0.20.115 tell 10.0.20.3, length 28
13:54:52.552964 ARP, Request who-has 10.0.20.116 tell 10.0.20.3, length 28
13:54:52.552965 ARP, Request who-has 10.0.20.119 tell 10.0.20.3, length 28
13:54:52.552965 ARP, Request who-has 10.0.20.120 tell 10.0.20.3, length 28
13:54:52.552966 ARP, Request who-has 10.0.20.123 tell 10.0.20.3, length 28
13:54:52.552967 ARP, Request who-has 10.0.20.209 tell 10.0.20.3, length 28
13:54:52.552967 ARP, Request who-has 10.0.20.208 tell 10.0.20.3, length 28
13:54:52.552968 ARP, Request who-has 10.0.20.202 tell 10.0.20.3, length 28
13:54:52.552969 ARP, Request who-has 10.0.20.201 tell 10.0.20.3, length 28
13:54:52.552969 ARP, Request who-has 10.0.20.198 tell 10.0.20.3, length 28
13:54:52.552970 ARP, Request who-has 10.0.20.197 tell 10.0.20.3, length 28
13:54:52.552970 ARP, Request who-has 10.0.20.194 tell 10.0.20.3, length 28
13:54:52.552971 ARP, Request who-has 10.0.20.239 tell 10.0.20.3, length 28
13:54:52.648942 ARP, Request who-has 10.0.20.196 tell 10.0.20.3, length 28
13:54:52.648943 ARP, Request who-has 10.0.20.190 tell 10.0.20.3, length 28
13:54:52.648944 ARP, Request who-has 10.0.20.187 tell 10.0.20.3, length 28
13:54:52.648945 ARP, Request who-has 10.0.20.193 tell 10.0.20.3, length 28
13:54:52.648945 ARP, Request who-has 10.0.20.191 tell 10.0.20.3, length 28
13:54:52.648946 ARP, Request who-has 10.0.20.188 tell 10.0.20.3, length 28
13:54:52.648947 ARP, Request who-has 10.0.20.185 tell 10.0.20.3, length 28
13:54:52.648947 ARP, Request who-has 10.0.20.195 tell 10.0.20.3, length 28
13:54:52.648948 ARP, Request who-has 10.0.20.192 tell 10.0.20.3, length 28
13:54:52.648948 ARP, Request who-has 10.0.20.189 tell 10.0.20.3, length 28
13:54:52.648949 ARP, Request who-has 10.0.20.186 tell 10.0.20.3, length 28
13:54:52.680923 ARP, Request who-has 10.0.20.152 tell 10.0.20.3, length 28
13:54:52.680924 ARP, Request who-has 10.0.20.155 tell 10.0.20.3, length 28
13:54:52.680925 ARP, Request who-has 10.0.20.158 tell 10.0.20.3, length 28
13:54:52.680925 ARP, Request who-has 10.0.20.161 tell 10.0.20.3, length 28
13:54:52.680926 ARP, Request who-has 10.0.20.164 tell 10.0.20.3, length 28
13:54:52.680927 ARP, Request who-has 10.0.20.167 tell 10.0.20.3, length 28
13:54:52.680927 ARP, Request who-has 10.0.20.171 tell 10.0.20.3, length 28
13:54:52.680928 ARP, Request who-has 10.0.20.172 tell 10.0.20.3, length 28
13:54:52.680929 ARP, Request who-has 10.0.20.176 tell 10.0.20.3, length 28
13:54:52.680929 ARP, Request who-has 10.0.20.177 tell 10.0.20.3, length 28
13:54:52.680930 ARP, Request who-has 10.0.20.182 tell 10.0.20.3, length 28
13:54:52.680930 ARP, Request who-has 10.0.20.183 tell 10.0.20.3, length 28
13:54:52.726900 IP 10.0.20.3.57450 > 10.0.20.4.http: Flags [S], seq 1057964882, win 64240, options [mss 1460,sackOK,TS val 230224585 ecr 0,nop,wscale 7], length 0
13:54:52.726908 IP 10.0.20.4.http > 10.0.20.3.57450: Flags [R.], seq 0, ack 1057964883, win 0, length 0
13:54:52.776920 ARP, Request who-has 10.0.20.212 tell 10.0.20.3, length 28
13:54:52.776922 ARP, Request who-has 10.0.20.215 tell 10.0.20.3, length 28
13:54:52.776922 ARP, Request who-has 10.0.20.218 tell 10.0.20.3, length 28
13:54:52.776923 ARP, Request who-has 10.0.20.221 tell 10.0.20.3, length 28
13:54:52.776924 ARP, Request who-has 10.0.20.224 tell 10.0.20.3, length 28
13:54:52.776924 ARP, Request who-has 10.0.20.227 tell 10.0.20.3, length 28
13:54:52.776925 ARP, Request who-has 10.0.20.230 tell 10.0.20.3, length 28
13:54:52.776926 ARP, Request who-has 10.0.20.231 tell 10.0.20.3, length 28
13:54:52.776926 ARP, Request who-has 10.0.20.234 tell 10.0.20.3, length 28
13:54:52.776927 ARP, Request who-has 10.0.20.238 tell 10.0.20.3, length 28
13:54:52.776927 ARP, Request who-has 10.0.20.242 tell 10.0.20.3, length 28
13:54:52.776928 ARP, Request who-has 10.0.20.243 tell 10.0.20.3, length 28
13:54:52.872924 ARP, Request who-has 10.0.20.18 tell 10.0.20.3, length 28
13:54:52.872926 ARP, Request who-has 10.0.20.32 tell 10.0.20.3, length 28
13:54:52.872927 ARP, Request who-has 10.0.20.35 tell 10.0.20.3, length 28
13:54:52.872927 ARP, Request who-has 10.0.20.38 tell 10.0.20.3, length 28
13:54:52.872928 ARP, Request who-has 10.0.20.41 tell 10.0.20.3, length 28
13:54:52.872929 ARP, Request who-has 10.0.20.44 tell 10.0.20.3, length 28
13:54:52.872929 ARP, Request who-has 10.0.20.48 tell 10.0.20.3, length 28
13:54:52.872930 ARP, Request who-has 10.0.20.49 tell 10.0.20.3, length 28
13:54:52.872930 ARP, Request who-has 10.0.20.52 tell 10.0.20.3, length 28
13:54:52.872931 ARP, Request who-has 10.0.20.56 tell 10.0.20.3, length 28
13:54:52.872932 ARP, Request who-has 10.0.20.59 tell 10.0.20.3, length 28
13:54:52.872932 ARP, Request who-has 10.0.20.60 tell 10.0.20.3, length 28
13:54:52.968919 ARP, Request who-has 10.0.20.83 tell 10.0.20.3, length 28
13:54:52.968921 ARP, Request who-has 10.0.20.86 tell 10.0.20.3, length 28
13:54:52.968922 ARP, Request who-has 10.0.20.89 tell 10.0.20.3, length 28
13:54:52.968922 ARP, Request who-has 10.0.20.92 tell 10.0.20.3, length 28
13:54:52.968923 ARP, Request who-has 10.0.20.95 tell 10.0.20.3, length 28
13:54:52.968923 ARP, Request who-has 10.0.20.100 tell 10.0.20.3, length 28
13:54:52.968924 ARP, Request who-has 10.0.20.108 tell 10.0.20.3, length 28
13:54:52.968925 ARP, Request who-has 10.0.20.113 tell 10.0.20.3, length 28
13:54:52.968925 ARP, Request who-has 10.0.20.117 tell 10.0.20.3, length 28
13:54:52.968926 ARP, Request who-has 10.0.20.121 tell 10.0.20.3, length 28
13:54:52.968927 ARP, Request who-has 10.0.20.124 tell 10.0.20.3, length 28
13:54:52.968927 ARP, Request who-has 10.0.20.125 tell 10.0.20.3, length 28
13:54:53.257044 ARP, Request who-has 10.0.20.11 tell 10.0.20.3, length 28
13:54:53.257046 ARP, Request who-has 10.0.20.12 tell 10.0.20.3, length 28
13:54:53.257047 ARP, Request who-has 10.0.20.17 tell 10.0.20.3, length 28
13:54:53.257047 ARP, Request who-has 10.0.20.33 tell 10.0.20.3, length 28
13:54:53.257048 ARP, Request who-has 10.0.20.36 tell 10.0.20.3, length 28
13:54:53.257048 ARP, Request who-has 10.0.20.39 tell 10.0.20.3, length 28
13:54:53.257049 ARP, Request who-has 10.0.20.42 tell 10.0.20.3, length 28
13:54:53.257050 ARP, Request who-has 10.0.20.45 tell 10.0.20.3, length 28
13:54:53.257050 ARP, Request who-has 10.0.20.53 tell 10.0.20.3, length 28
13:54:53.257051 ARP, Request who-has 10.0.20.57 tell 10.0.20.3, length 28
13:54:53.257052 ARP, Request who-has 10.0.20.61 tell 10.0.20.3, length 28
13:54:53.257052 ARP, Request who-has 10.0.20.62 tell 10.0.20.3, length 28
13:54:53.257053 ARP, Request who-has 10.0.20.63 tell 10.0.20.3, length 28
13:54:53.257054 ARP, Request who-has 10.0.20.64 tell 10.0.20.3, length 28
13:54:53.257055 ARP, Request who-has 10.0.20.65 tell 10.0.20.3, length 28
13:54:53.257055 ARP, Request who-has 10.0.20.66 tell 10.0.20.3, length 28
13:54:53.257056 ARP, Request who-has 10.0.20.67 tell 10.0.20.3, length 28
13:54:53.257057 ARP, Request who-has 10.0.20.70 tell 10.0.20.3, length 28
13:54:53.257057 ARP, Request who-has 10.0.20.71 tell 10.0.20.3, length 28
13:54:53.257058 ARP, Request who-has 10.0.20.72 tell 10.0.20.3, length 28
13:54:53.257059 ARP, Request who-has 10.0.20.73 tell 10.0.20.3, length 28
13:54:53.257059 ARP, Request who-has 10.0.20.74 tell 10.0.20.3, length 28
13:54:53.257060 ARP, Request who-has 10.0.20.75 tell 10.0.20.3, length 28
13:54:53.257061 ARP, Request who-has 10.0.20.76 tell 10.0.20.3, length 28
13:54:53.257061 ARP, Request who-has 10.0.20.77 tell 10.0.20.3, length 28
13:54:53.257062 ARP, Request who-has 10.0.20.78 tell 10.0.20.3, length 28
13:54:53.257063 ARP, Request who-has 10.0.20.79 tell 10.0.20.3, length 28
13:54:53.257063 ARP, Request who-has 10.0.20.80 tell 10.0.20.3, length 28
13:54:53.257064 ARP, Request who-has 10.0.20.81 tell 10.0.20.3, length 28
13:54:53.257064 ARP, Request who-has 10.0.20.82 tell 10.0.20.3, length 28
13:54:53.257065 ARP, Request who-has 10.0.20.84 tell 10.0.20.3, length 28
13:54:53.257066 ARP, Request who-has 10.0.20.85 tell 10.0.20.3, length 28
13:54:53.257067 ARP, Request who-has 10.0.20.88 tell 10.0.20.3, length 28
13:54:53.257067 ARP, Request who-has 10.0.20.91 tell 10.0.20.3, length 28
13:54:53.353028 ARP, Request who-has 10.0.20.126 tell 10.0.20.3, length 28
13:54:53.353030 ARP, Request who-has 10.0.20.127 tell 10.0.20.3, length 28
13:54:53.353031 ARP, Request who-has 10.0.20.130 tell 10.0.20.3, length 28
13:54:53.353031 ARP, Request who-has 10.0.20.131 tell 10.0.20.3, length 28
13:54:53.353032 ARP, Request who-has 10.0.20.132 tell 10.0.20.3, length 28
13:54:53.353033 ARP, Request who-has 10.0.20.133 tell 10.0.20.3, length 28
13:54:53.353033 ARP, Request who-has 10.0.20.134 tell 10.0.20.3, length 28
13:54:53.353034 ARP, Request who-has 10.0.20.137 tell 10.0.20.3, length 28
13:54:53.353035 ARP, Request who-has 10.0.20.138 tell 10.0.20.3, length 28
13:54:53.353036 ARP, Request who-has 10.0.20.139 tell 10.0.20.3, length 28
13:54:53.353036 ARP, Request who-has 10.0.20.140 tell 10.0.20.3, length 28
13:54:53.353037 ARP, Request who-has 10.0.20.141 tell 10.0.20.3, length 28
13:54:53.353037 ARP, Request who-has 10.0.20.142 tell 10.0.20.3, length 28
13:54:53.353038 ARP, Request who-has 10.0.20.143 tell 10.0.20.3, length 28
13:54:53.353039 ARP, Request who-has 10.0.20.144 tell 10.0.20.3, length 28
13:54:53.353039 ARP, Request who-has 10.0.20.145 tell 10.0.20.3, length 28
13:54:53.353040 ARP, Request who-has 10.0.20.146 tell 10.0.20.3, length 28
13:54:53.353041 ARP, Request who-has 10.0.20.147 tell 10.0.20.3, length 28
13:54:53.353041 ARP, Request who-has 10.0.20.148 tell 10.0.20.3, length 28
13:54:53.353042 ARP, Request who-has 10.0.20.149 tell 10.0.20.3, length 28
13:54:53.353043 ARP, Request who-has 10.0.20.150 tell 10.0.20.3, length 28
13:54:53.353043 ARP, Request who-has 10.0.20.151 tell 10.0.20.3, length 28
13:54:53.353044 ARP, Request who-has 10.0.20.154 tell 10.0.20.3, length 28
13:54:53.353044 ARP, Request who-has 10.0.20.156 tell 10.0.20.3, length 28
13:54:53.353045 ARP, Request who-has 10.0.20.157 tell 10.0.20.3, length 28
13:54:53.353046 ARP, Request who-has 10.0.20.159 tell 10.0.20.3, length 28
13:54:53.353046 ARP, Request who-has 10.0.20.160 tell 10.0.20.3, length 28
13:54:53.353047 ARP, Request who-has 10.0.20.162 tell 10.0.20.3, length 28
13:54:53.353047 ARP, Request who-has 10.0.20.165 tell 10.0.20.3, length 28
13:54:53.353048 ARP, Request who-has 10.0.20.168 tell 10.0.20.3, length 28
13:54:53.481257 ARP, Request who-has 10.0.20.199 tell 10.0.20.3, length 28
13:54:53.481259 ARP, Request who-has 10.0.20.200 tell 10.0.20.3, length 28
13:54:53.481259 ARP, Request who-has 10.0.20.203 tell 10.0.20.3, length 28
13:54:53.481260 ARP, Request who-has 10.0.20.204 tell 10.0.20.3, length 28
13:54:53.481261 ARP, Request who-has 10.0.20.205 tell 10.0.20.3, length 28
13:54:53.481261 ARP, Request who-has 10.0.20.206 tell 10.0.20.3, length 28
13:54:53.481262 ARP, Request who-has 10.0.20.207 tell 10.0.20.3, length 28
13:54:53.481263 ARP, Request who-has 10.0.20.210 tell 10.0.20.3, length 28
13:54:53.481263 ARP, Request who-has 10.0.20.211 tell 10.0.20.3, length 28
13:54:53.481264 ARP, Request who-has 10.0.20.213 tell 10.0.20.3, length 28
13:54:53.481265 ARP, Request who-has 10.0.20.214 tell 10.0.20.3, length 28
13:54:53.481265 ARP, Request who-has 10.0.20.216 tell 10.0.20.3, length 28
13:54:53.481266 ARP, Request who-has 10.0.20.217 tell 10.0.20.3, length 28
13:54:53.481266 ARP, Request who-has 10.0.20.219 tell 10.0.20.3, length 28
13:54:53.481267 ARP, Request who-has 10.0.20.220 tell 10.0.20.3, length 28
13:54:53.481268 ARP, Request who-has 10.0.20.222 tell 10.0.20.3, length 28
13:54:53.481268 ARP, Request who-has 10.0.20.225 tell 10.0.20.3, length 28
13:54:53.481269 ARP, Request who-has 10.0.20.228 tell 10.0.20.3, length 28
13:54:53.481270 ARP, Request who-has 10.0.20.233 tell 10.0.20.3, length 28
13:54:53.481271 ARP, Request who-has 10.0.20.246 tell 10.0.20.3, length 28
13:54:53.481271 ARP, Request who-has 10.0.20.247 tell 10.0.20.3, length 28
13:54:53.481272 ARP, Request who-has 10.0.20.249 tell 10.0.20.3, length 28
13:54:53.481273 ARP, Request who-has 10.0.20.250 tell 10.0.20.3, length 28
13:54:53.481273 ARP, Request who-has 10.0.20.251 tell 10.0.20.3, length 28
13:54:53.481274 ARP, Request who-has 10.0.20.252 tell 10.0.20.3, length 28
13:54:53.481274 ARP, Request who-has 10.0.20.253 tell 10.0.20.3, length 28
13:54:53.481275 ARP, Request who-has 10.0.20.254 tell 10.0.20.3, length 28
13:54:53.481276 ARP, Request who-has 10.0.20.0 tell 10.0.20.3, length 28
13:54:53.481276 ARP, Request who-has 10.0.20.68 tell 10.0.20.3, length 28
13:54:53.481277 ARP, Request who-has 10.0.20.69 tell 10.0.20.3, length 28
13:54:53.481278 ARP, Request who-has 10.0.20.114 tell 10.0.20.3, length 28
13:54:53.481278 ARP, Request who-has 10.0.20.118 tell 10.0.20.3, length 28
13:54:53.481279 ARP, Request who-has 10.0.20.122 tell 10.0.20.3, length 28
13:54:53.481280 ARP, Request who-has 10.0.20.128 tell 10.0.20.3, length 28
13:54:53.481280 ARP, Request who-has 10.0.20.129 tell 10.0.20.3, length 28
13:54:53.481281 ARP, Request who-has 10.0.20.135 tell 10.0.20.3, length 28
13:54:53.481281 ARP, Request who-has 10.0.20.136 tell 10.0.20.3, length 28
13:54:53.481282 ARP, Request who-has 10.0.20.153 tell 10.0.20.3, length 28
13:54:53.577103 ARP, Request who-has 10.0.20.239 tell 10.0.20.3, length 28
13:54:53.577104 ARP, Request who-has 10.0.20.194 tell 10.0.20.3, length 28
13:54:53.577105 ARP, Request who-has 10.0.20.197 tell 10.0.20.3, length 28
13:54:53.577105 ARP, Request who-has 10.0.20.198 tell 10.0.20.3, length 28
13:54:53.577106 ARP, Request who-has 10.0.20.201 tell 10.0.20.3, length 28
13:54:53.577106 ARP, Request who-has 10.0.20.202 tell 10.0.20.3, length 28
13:54:53.577107 ARP, Request who-has 10.0.20.208 tell 10.0.20.3, length 28
13:54:53.577108 ARP, Request who-has 10.0.20.209 tell 10.0.20.3, length 28
13:54:53.672917 ARP, Request who-has 10.0.20.186 tell 10.0.20.3, length 28
13:54:53.672919 ARP, Request who-has 10.0.20.189 tell 10.0.20.3, length 28
13:54:53.672920 ARP, Request who-has 10.0.20.192 tell 10.0.20.3, length 28
13:54:53.672920 ARP, Request who-has 10.0.20.195 tell 10.0.20.3, length 28
13:54:53.672921 ARP, Request who-has 10.0.20.185 tell 10.0.20.3, length 28
13:54:53.672922 ARP, Request who-has 10.0.20.188 tell 10.0.20.3, length 28
13:54:53.672922 ARP, Request who-has 10.0.20.191 tell 10.0.20.3, length 28
13:54:53.672923 ARP, Request who-has 10.0.20.193 tell 10.0.20.3, length 28
13:54:53.672924 ARP, Request who-has 10.0.20.187 tell 10.0.20.3, length 28
13:54:53.672924 ARP, Request who-has 10.0.20.190 tell 10.0.20.3, length 28
13:54:53.672925 ARP, Request who-has 10.0.20.196 tell 10.0.20.3, length 28
13:54:55.048845 ARP, Request who-has 10.0.20.3 tell 10.0.20.4, length 28
13:54:55.048960 ARP, Reply 10.0.20.3 is-at 00:50:56:16:30:c7 (oui Unknown), length 28
^C
769 packets captured
769 packets received by filter
0 packets dropped by kernel
```

Another huge flurry of broadcast activity! My apologies for having you scroll
through it all, but scanning such output, and even better—analyzing what's going
on—is essential practice. Expect lots of output as part of these labs.

So what did we see in all that? The network scan output reported that `host3` and
`host4` were discovered; did you see where in the packet capture the ARP request
and replies are? And speaking of packet captures, we left `tcpdump` running back on
`host2`, so cancel that now and try to find any broadcasts involving `10.0.20.0/24`.

Assuming everything was done correctly, there should be nothing from that second
subnet, as shown in the (heavily shortened) output below. Very good.

```
(omitted)
...
13:54:05.768997 ARP, Request who-has 10.0.10.44 tell 10.0.10.1, length 28
13:54:05.768997 ARP, Request who-has 10.0.10.67 tell 10.0.10.1, length 28
13:54:05.897167 ARP, Request who-has 10.0.10.192 tell 10.0.10.1, length 28
13:54:05.897170 ARP, Request who-has 10.0.10.201 tell 10.0.10.1, length 28
13:54:05.897171 ARP, Request who-has 10.0.10.203 tell 10.0.10.1, length 28
13:54:05.897171 ARP, Request who-has 10.0.10.90 tell 10.0.10.1, length 28
13:54:05.993145 ARP, Request who-has 10.0.10.248 tell 10.0.10.1, length 28
13:54:07.432845 ARP, Request who-has 10.0.10.1 tell 10.0.10.2, length 28
13:54:07.432890 ARP, Request who-has 10.0.10.2 tell 10.0.10.1, length 28
13:54:07.432893 ARP, Reply 10.0.10.2 is-at 00:50:56:ad:0e:33 (oui Unknown), length 28
13:54:07.432898 ARP, Reply 10.0.10.1 is-at 00:50:56:94:55:70 (oui Unknown), length 28
^C
769 packets captured
769 packets received by filter
0 packets dropped by kernel
```

## Conclusion and Next Lab

So there you have it: use VLANs around your subnets to maintain full control over
your broadcast domains and take advantage of all the other benefits of VLANs that
I briefly mentioned in the beginning, which we'll dig into in new labs.

Now isolating subnets against unwanted broadcast traffic is great, but what about
unicast, multicast, or even anycast traffic that needs to be able to reach across
subnets? It's time to bring in Layer 3 and the role of routing to enable subnets
to communicate and full networks to function. But in order to do that, we'll need
to add a router to our lab.

In the next lab, [Installing VyOS on Proxmox](../vyos/installing-vyos-on-proxmox.md),
we'll learn how to install VyOS, a fantastic opensource network operating system
(NOS), on Proxmox so we can use it as our router and firewall in future labs.
