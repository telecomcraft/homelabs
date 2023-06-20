# Connecting and Configuring Network Hosts

Imagine having two physical Linux servers, each with a single network adapter.
If you interconnect the two servers with a patch cord, and configure the network
adapters so they can communicate, what will you have?

A simple, but fully-functional network. And that's where you'll begin your
learning on building networks, which is all about connecting hosts together.

In this lab, we're going to directly connect our network hosts together virtually
by adding them to a Linux bridge and configure them with the modern Linux
networking tools NetworkManager and iproute2. No additional routers, switches, or
firewalls will be necessary at this point; just the built-in functionality of
Linux and Proxmox.

## Step 1: Create a Linux Bridge for Lab Connectivity

A [Linux bridge](https://developers.redhat.com/articles/2022/04/06/introduction-linux-bridging-commands-and-features#bridge_switchdev)
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
