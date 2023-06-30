---
draft: true 
date: 2023-06-30
authors:
  - eronlloyd
categories:
  - Learning
---

# Learn VyOS to Learn Networking

## Overview

As a supplement to our growing amount of VyOS-based labs, this article offers and introduction
to VyOS itself and discusses why it's worth learning. The objective of this article is to help
you become more familiar with the software and the community behind it, before we start learning
how to use it. If you’re already very familiar with VyOS, feel free to skip ahead, but I believe
most VyOS users will benefit from a bit of more background.

## What is VyOS?

According to [the official documentation](https://docs.vyos.io/en/latest/introducing/about.html), VyOS is an open source “network operating system” (NOS), based on [Debian GNU/Linux](https://www.debian.org/). As a NOS, it offers [a wealth of networking functions](https://vyos.io/products/#vyos-router), but primarily it’s considered a router, firewall, and VPN platform.

VyOS [officially began in 2013](https://blog.vyos.io/index.php/2013/10/13/complete-source-code-fork) as a fork of the open source, community edition of Vyatta Core NOS, which was being discontinued by it’s parent company, Brocade. You’ll often see references to Vyatta both in the VyOS software and within the VyOS community, so I believe it’s helpful to have a basic understanding of the relationship.

Vyatta was both the name of the NOS and its original company, which was founded in 2005. At the time, it was an exciting and ambitious effort to produce an open source, Linux-based NOS that could run on commodity x86 hardware directly or as a virtual machine in a hypervisor such as ESXi or KVM. This is in comparison to having to purchase more expensive and purpose-built networking “appliances” for routing and firewalling functions.

The earliest publicly available release of Vyatta was in 2006, and the developers steadily added functionality to the software, as you can see it the [Vyatta Wikipedia article](https://en.wikipedia.org/wiki/Vyatta). The company was bought out by Brocade in 2013, after which the Vyatta code was closed and VyOS was created as a fork, and Brocade was subsequently bought out in 2016 and became what we now know as [Broadcom](https://www.broadcom.com/).

After the fork of Vyatta Core 6.6R1, the new [VyOS community](https://vyos.org/) continued to improve the software and began laying out long-term plans to restructure the architecture for the future. Release numbering was restarted, and [version 1.0 of VyOS was released on December 22, 2013](https://blog.vyos.io/index.php/2013/12/22/100-release). From there, the plan was to rewrite a number of configuration backend components, [to be released as VyOS 2.0](https://blog.vyos.io/vyos-2-dot-0-development-digest-number-1).

That was scrapped for a more evolutionary approach, however, and as of this writing, VyOS 1.4 is the most bleeding edge version. I’ll talk more about the project release numbering later on in the tutorial, as it’s a very important thing to understand.

As far as providing you some background, I think I’m going to leave it there for now. I admittedly summarized a lot, but hopefully I’ve have given you enough details to make you more familiar and point you in the right directions for further information. From here, let’s dig more into VyOS’s capabilities and use cases.

## How Can I Use It?

At the beginning of the tutorial, I explained that VyOS is a “Layer 3+” NOS, able to perform network functions such as routing, firewalling, and VPN. Some NOS products are more Layer 2 focused, although the lines are somewhat blurred with the commodification of the underlying software and hardware used to run network equipment. In addition to its core functions, there are many other network services and utilities built into the NOS, which we’ll cover in future tutorials.

If you’re familiar with Cisco’s [Internetwork Operating System (IOS)](https://www.cisco.com/c/en/us/products/ios-nx-os-software/index.html) or Juniper’s [Juniper Operating System (JUNOS)](https://www.juniper.net/us/en/products-services/nos/junos/), then VyOS should quickly become familiar. You log into the CLI, where you interact with the NOS using an intuitive and well-designed set of commands to operate and configure the network functions. For more advanced automation needs, an API over HTTP is now available, too.

And for those uncomfortable with the CLI or API, several projects are underway to provide a GUI as well; this was something that existed within Vyatta but was not open sourced. Once this is available, we’ll cover that with tutorials as well.

VyOS can run in two primary ways:

1. Installed on bare metal hardware, such as a server or network appliance
2. Installed in a virtualized environment running on a hypervisor

Because it is built on top of Linux, there are many interesting devices you may be able to get it to run on. For now, though, we’ll focus on these two common approaches.

Regardless of how you run it, VyOS can serve you well at nearly any scale. If you’re just starting out, I’ll teach you the basics in a simple virtual lab environment you can run on nearly any desktop or laptop. From there, you can run it as your home router, incorporate it into business networks, and even deploy it in data centers and on cloud providers. Of course all of these are great tutorial topics, so stay tuned!

## Where Can I Get It?

If you’re still reading this tutorial, you’re probably considering installing it. Excellent! I’ll be doing multiple tutorials on this, but here are the simple steps to get you started.

## Why Should I Learn It?

## How Can I Get More Involved?

Once you start using an open source product, and possibly begin to depend on it, a common thought is to get involved in the community, and from there, possibly begin actively contributing to the project’s efforts.