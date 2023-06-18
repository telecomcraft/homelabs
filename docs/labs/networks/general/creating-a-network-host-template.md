# Creating and Configuring an LXC Container Template for Cloning as Fully-functional Network Hosts

For a long time, network simulators like Packet Tracer and GNS3 would provide a very
basic network host such as [VPCS](https://docs.gns3.com/docs/emulators/vpcs/),
which could only do a very limited subset of network functions such as serving as a DHCP client and responding to pings.

But the role of network hosts in your labs can become capable of so much more
functionality by using real operating systems such as Linux. Using them offers
an opportunity to not only become more familiar with real hosts, but gain deeper
insights into networking fundamentals as well.

In addition, because nearly every network operating system (NOS) is built on Linux,
understanding the underlying networking capabilities of the Linux kernel and all the
included tools in a Linux host provide insights into how routers, switches, and
firewalls work.

In this lab we're going to create an Ubuntu Linux "template" inside Proxmox VE that
can be cloned and used as powerful network hosts to interact with inside our network
labs.

## Step 1: Download Ubuntu 22.04 LXC Template onto a Proxmox VE Node

A big advantage of Proxmox for creating labs is the availability of [Linux Containers
(LXC)](https://linuxcontainers.org/), a lightweight way to create isolated instances
of Linux that is much more efficient than using full virtual machines. The Proxmox
UI makes it extremely easy to install and manage containers, so that's what we'll do
here.

* In the Proxmox VE explorer, select the `local` storage view, and select the
`CT Templates` submenu item.
* At the top of the view, click the `Templates` button and search for `ubuntu-22.04-standard` in the search box.
* Select that item in the list and click the `Download` button to save that template to your Proxmox VE storage.

Screenshot

## Step 2: Create a Proxmox CT Using the Ubuntu LXC Template

With the Ubuntu 22.04 LXC template downloaded, we can now create a container in
Proxmox (abbreviated to CT) using this template. Click the `Create CT` button at the top-right of the Proxmox VE window to get started.

* On the `General` tab, make sure the correct node (if you have a Proxmox VE cluster)
that you saved the Ubuntu LXC template to is selected.
* Set the hostname to `host0`.
* Set and confirm the root password for the CT.

Screenshot of General

* On the `Template` tab, make sure Storage is set to `local`, and select
`ubuntu-22.04-1-standard_22.04-1_amd64.tar.zst` (or your equivalent based on the
platform) for Template.

Screenshot of Template

* One the `Disks` tab, leave all default configurations in place.

Screenshot of Disks

* One the `CPU` tab, leave all default configurations in place.

Screenshot of CPU

* One the `Memory` tab, leave all default configurations in place.

Screenshot of Memory

* One the `Network` tab, change the IPv4 to `DHCP`. Note that this assumes the
default bridge `vmbr0` is connected to a gateway that can respond to DHCP requests
on an Internet-connected LAN subnet.

Screenshot of Network

* One the `DNS` tab, leave all default configurations in place.

Screenshot of DNS

* One the `Confirm` tab, review all your settings and when complete, click
the `Finish` button.

Screenshot of Confirm

## Step 3: Update and Configure the Ubuntu Linux Server for Use as a Network Host

## Step 4: Convert the Ubuntu Linux Server into a Custom Container Template