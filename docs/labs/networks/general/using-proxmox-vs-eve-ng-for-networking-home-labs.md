Welcome to the first video of the Networking Home Lab series here on the channel. I’m Eron Lloyd, and today we’re going to talk about why I decided to use Proxmox instead of EVE-NG for doing networking labs.

In Episode 1 of the Fiber Nerds podcast, we talked a bit about our plans for the networking home lab, including my focus on studying for the Juniper JNCIS-SP, a certification covering intermediate switching, routing, and multiprotocol label switching (MPLS), which is sometimes called Layer 2.5.

Last year I starting learning Juniper when I found out about all the free resources they made available, as well as the very affordable $50 exam fees to get certified in this highly valuable vendor platform. I already preferred Junos OS over Cisco, so I dove straight in.

I got my JNCIA-Junos in four weeks over the Summer in 2022, and that Fall I also received my JNCDA. Since I’m now going to be focusing a lot more on network engineering in my job, I need to pick up where I left off last year and solidify my skills by studying for and passing the JNCIS-SP.

So I’m going to need to lab, a lot.

I admit I didn’t get much hands-on experience while taking either the JNCIA-Junos or JNCDA; unfortunately there’s no lab exercises as part of the free Juniper course offerings. But what they do offer is a full library of free books and other resources you can self-study with to supplement your course work, so that’s what I’ll be doing here in the labs.

To add to that, Christian Scholz has put out some great resources for doing Juniper labs with EVE-NG. I joined one of his webinars, and even setup a bare metal EVE-NG instance last fall to get started with. EVE-NG is awesome for labbing, and like GNS3 is one of the most popular ways to practice network engineering. But I needed something more.

In addition to leveling up my networking skills this year, I’m also focused on sharpening my server virtualization skills, using Proxmox. I already had a hypervisor cluster setup for that, and realized that since EVE-NG is essentially a specialized hypervisor, I could accomplish both goals together by building my networking AND server labs in Proxmox.

So that’s what I’ve done. It’s now been almost six months of experimenting, and mainly learning Proxmox from the ground up, but everything is working just as I hoped.

But here’s what I’ve realized now that I’m had some time with my new lab: I believe it actually makes more sense to use a production-class hypervisor like Proxmox, ESXi, or Xen to build a lab instead of a simulator-class hypervisor like EVE-NG or GNS3.

This is because you no longer are simulating a production environment, but actually building networks using the same platforms you would in a production environment. Now you’re not only practicing with the virtual networks, but the entire virtualization infrastructure—including the hypervisors, VMs, and containers—and seeing how everything interacts.

There are some aspects of EVE-NG that you won’t have available using Proxmox, such as the visual diagrams of the networks, but I’ll show you other real-world tools for designing, planning, and even monitoring your networking lab as part of this series.

I won’t be going into a lot of detail on Proxmox itself in the networking labs, as we’ll be covering that instead in our server labs, and outside of getting the NOSs installed on Proxmox, you should be able to follow along for everything else if you’re using another virtualization platform or even hardware gear for your own labs.

With all that out of the way, let me give you a quick tour of my Proxmox cluster from the perspective of the networking labs…

- Cluster consists of three Lenovo System x3100 M5 tower servers running Proxmox 7.4
- The first hypervisor, pve1, consists of Linux servers running DNS, DHCP, databases, web applications, network monitoring, and performing many other roles. We’ll be getting into all of this during upcoming networking and server lab videos.
- The second and third hypervisors host the guest OSs such as vSRX and VyOS used for the networking home labs.
- Here are the details for one of the vSRXs, which I’ve modeled after the Juniper SRX-300 branch gateway. You can see the built-in dashboard and configuration screens for each VM, which we’ll cover in the upcoming videos on installing and configuring vSRX.
- Within each hypervisor are virtual networks managed by Proxmox, composed of Open vSwitches and Linux bridges. These are used to both interconnect guest OSs and throughout the hypervisors, and what we’ll use to build a virtual networking lab on top of.
- I already have set up entire lab networks inside of the cluster, so I can, for instance have these two vSRXs communicate just like physical gear interconnected together. There’s so much we can begin doing from here.

I hope I’ve got you intrigued, or at least interested, in what we can do with networking inside a Proxmox hypervisor. In the next video, I’m going to show you how I’m using Netbox and draw.io to design and document my home lab. These skills are just as important as being able to configure a switch or router, so your home lab should integrate these activities from the start, too.

If you like what you’ve heard so far, please subscribe to our channel and click the bell icon to be notified of every new video we release moving forward. And if you have any comments or questions, please share them below.