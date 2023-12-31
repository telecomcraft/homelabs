# Initial Proxmox Network Configuration

In this lab we'll review Proxmox's initial configuration and discuss some
preparatory changes to support our lab activities.

!!! warning

    Be sure to review the following recommendations before beginning the
    labs to ensure your setup is configured properly and help
    you avoid conflicts or downtime on your home LAN.

## Default Network Configuration

Out of the box, Proxmox detects all compatible network interface cards (NICs) on
each node and adds them to the node's list of network devices in the "System" >
"Network" panel. The Proxmox installation will also create a Linux Bridge called
`vmbr0` that maps to one of the hardware NICs for LAN connectivity.

This bridge will be configured
with IP addressing for the node itself and the default gateway for the LAN the
node is connected to. The IP addresses for `vmbr0` will be used for accessing the
node via the Web and SSH, and also will provide connectivity for clustering with other nodes.

I recommend considering `vmbr0` to be the node's management network, and
restricted for these roles. Depending on the setup, it may also be used for LAN
access for production VMs and CTs (often on separate VLANs).

For our labs, however, we'll create separate bridges that allow us to create
self-contained networks, and eventually we'll implement functionality
to extend our networks to additional hardware and the Internet.

## Lab Preparation Changes

### IP Addressing Recommendations

There are many ways to plan out your home lab's IP addressing, and as your skills
improve your preferences will evolve along with your lab's needs. However, at the
very least I strongly suggest having a separate address space from your home LAN
and isolate your home lab activities from taking down the LAN that others in your
home depend on.

My personal IPv4 address plan is the following:

* `192.168.0.0/16` for my home's LAN, split into separate subnets for secure
Wi-Fi, guest Wi-Fi, IoT, etc.
* `172.16.0.0/12` for my home's management network, also split into separate
subnets for network equipment, server/storage equipment, security devices,
production VMs/CTs, etc.
* `10.0.0.0/8` for my home lab subnets, as necessary. We'll be using this address
space here in the documentation, as well, so you're aware.
* In addition to the `10/8` prefix, we'll also be using some non-RFC 1918 prefixes
reserved for private use to represent public address space in our labs. This
includes the TEST-NET prefixes `192.0.2.0/24`, `198.51.100.0/24`,
`203.0.113.0/24`, and the non-allocated prefix `128.66.0.0/16`.

Not included yet is a IPv6 address plan, which should also be considered an
essential discipline for network training. Mastering IPv6 through my home labs
is a personal priority of mine, so stay tuned for more on that.

### Hardware Recommendations

The device used to install Proxmox should have at least two NICs:

* One for `vmbr0` and the management network roles described above
* Another for lab VM access to your LAN

Ideally, it would be better to have at least four NICs:

* One for (ideally isolated) network management
* One for cluster communication between nodes
* One for production VMs and CTs to access your LAN
* One for lab VMs and CTs to access your LAN

We'll expand on this setup in future labs, as well as upgrading your node with
the necessary hardware, but for now we'll focus on a two NIC configuration.

### Custom MAC Address Prefix

!!! notice

    This section is for users of PVE 8.0 and older. PVE 8.1 introduced a new option
    that provides Proxmox's recently acquired UAA by default.

Prior to PVE 8.1, Proxmox used unregistered and randomized MAC addresses for the virtual
network devices for VMs and CTs, which are listed within the "Hardware" view for
VMs and the "Network" view for CTs. These are called Locally Administered
Addresses (LAAs).

Used for server VMs and CTs, this doesn't seem to be an issue. However, I ran into
an issue when installing VyOS on Proxmox where it wouldn't add all detected
interfaces to the configuration because the the MAC addresses weren't Universally
Administered Address (UAAs). It may also affect other network operating systems
(NOSs) as well, but I haven't yet verified that.

It won't prevent you from configuring and using VyOS on Proxmox, but it can help
new users to have this functionality in place and the fix is easy: instead of
using LAA MAC addresses, set a custom prefix that Proxmox will use as a UAA OUI for
all network devices.

To do this, click on the "Datacenter" your node is in, then click on "Options"
and edit the "MAC address prefix" to be `BC:24:11`. This is the new Proxmox OUI used
for network devices within PVE, and VyOS (or any other OS) should properly recognize it.

From now on all virtual network devices will be issued a UAA MAC address with the
`BC:24:11` OUI and a random suffix. For some labs I may have you configure
specific MAC addresses for lab tasks, so maintaining consistency will be helpful.
