# Creating a Network Host Template in Proxmox

For a long time, network simulators like Packet Tracer and GNS3 would provide a very
basic network host such as [VPCS](https://docs.gns3.com/docs/emulators/vpcs/),
which could only do a very limited subset of network functions such as serving as a DHCP client and responding to pings.

But the role of network hosts in your labs can become capable of so much more
functionality by using real operating systems such as Linux. Using them offers
an opportunity to not only become more familiar with real hosts, but gain a deeper
understanding into networking fundamentals as well.

In addition, because nearly every network operating system (NOS) is built on Linux,
understanding the underlying networking capabilities of the Linux kernel and all the
included tools in a Linux host provide insights into how routers, switches, and
firewalls work.

In this lab we're going to create an Ubuntu Linux "template" inside Proxmox VE that can
be cloned and used as powerful network hosts to interact with inside our network labs.

## Step 1: Download Ubuntu 22.04 LXC Template Onto a Proxmox VE Node

A big advantage of Proxmox for creating labs is the availability of [Linux Containers
(LXC)](https://linuxcontainers.org/), a lightweight way to create isolated instances
of Linux that are much more efficient than using full virtual machines. The Proxmox
UI makes it extremely easy to install and manage containers, so that's what we'll do
here.

* In the Proxmox VE tree, select the `local` storage view, and select the
`CT Templates` submenu item.
* At the top of the view, click the `Templates` button and search for
`ubuntu-22.04-standard` in the search box.
* Select that item in the list and click the `Download` button to save that template
to your Proxmox VE storage.

Screenshot

## Step 2: Create a Proxmox CT Using the Ubuntu LXC Template

With the Ubuntu 22.04 LXC template downloaded, we can now create a container in
Proxmox (abbreviated to CT) using this template. Click the `Create CT` button at the
top-right of the Proxmox VE window to get started.

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

## Step 3: Update and Configure the Ubuntu Linux CT for Use as a Network Host

Before this CT can become a custom template for cloning, there is some
configuration to it that must be done. Select the newly-created `host0` CT from the
list, then select the `Console` tab and click the `Start Now` button to run the CT.

From there, log into the CT's root account using the password entered in Step 2
above, and perform the following configuration tasks.

Add a non-root administrator account and fill out the proceeding prompts:

```
adduser <user_name>
```

Add the newly-created account to the sudo group to use administrator priveleges:

```
addgroup <user_name> sudo
```

Log out of the root account then log in again using your new user name and password:

```
logout
```

!!! tip

    Upon login, you should see the following message confirming sudo priveleges:

    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.

Upgrade all existing packages:

```
sudo apt update && sudo apt dist-upgrade
```

Install several additional packages:

```
sudo apt install network-manager nmap ipcalc
```

Clean up the package database:

```
sudo apt clean
```

Clean up unnecessary packages:

```
sudo apt autoremove
```

Remove the current SSH host keys so each clone regenerates new keys:

```
sudo rm /etc/ssh/ssh_host_*
```

Remove the current machine ID so each clone regenerates a new ID:

```
sudo truncate -s 0 /etc/machine-id
```

Shut down the CT:

```
sudo systemctl poweroff
```

## Step 4: Convert the Ubuntu Linux CT into a Custom Container Template

You're now ready to convert this CT into a custom template.

* With the CT powered down, click the `More` drop-down button at the top-right of
the Proxmox VE window, then click `Convert to template`.
* Click the `Yes` button to proceed, and the CT will be converted into a template.

Your network host template is now ready. Complete the
[Cloning a Network Host Template in Proxmox](cloning-a-network-host-template-in-proxmox.md)
lab to practice creating Ubuntu Linux network hosts for use in upcoming labs.
