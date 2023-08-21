# Installing MikroTik RouterOS on Proxmox

## Introduction

To enable our subnets on Linux bridges to communicate within our labs across
increasingly complex network topologies, we have to add Layer 3 and 4 capabilities
to our setup. To do this, we're going incorporate MikroTik's RouterOS into Proxmox
and serve in this capacity.

If you're not familiar with RouterOS and why it's a fantastic platform to learn and
use in your networks, [check out this blog post](/blog/learn-mikrotik-to-learn-networking)
where we cover what it is, its background and architecture, and its advantages for both
practice labs and production networks.

In this lab, we're going to cover the process to install a RouterOS virtual machine
(VM) in Proxmox and prepare it for use as a full-featured router/firewall/vpn
in our networks.

## Step 1: Create a MikroTik Account

RouterOS is available to use for free, but with several caveats that you must be
aware of:

1. Every instance of RouterOS you use must still be registered via a MikroTik
account, even the free version.

2. The free version is limited to 

## Step 2: Adding RouterOS as an Image in Proxmox

### Option A: Use 

## Step 3: Creating and Configuring a VM for RouterOS

## Step 3: Installing the RouterOS Image on a VM

## Conclusion and Next Lab
