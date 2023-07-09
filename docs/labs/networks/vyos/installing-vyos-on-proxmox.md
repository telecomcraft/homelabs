---
tags:
  - VyOS
  - Proxmox
---

# Installing VyOS on Proxmox

## Introduction

To enable our subnets on Linux bridges to communicate within our labs across
increasingly complex network topologies, we have to add Layer 3 and 4 capabilities
to our setup. To do this, we're going incorporate VyOS into Proxmox and serve in
this capacity.

If you're not familiar with VyOS and why it's a fantastic platform to learn and
use in your networks, [check out this blog post](/blog/learn-vyos-to-learn-networking)
where we cover what it is, its background and architecture, and its advantages for both practice labs and production networks.

In this lab, we're going to cover the process to install a VyOS virtual machine
(VM) in Proxmox and prepare it for use as a full-featured router/firewall/vpn
in our networks.

## Step 1: Adding VyOS as an Image in Proxmox

To create a VM in Proxmox, you'll first need an "image" file of the software that
will run as the operating system for your VM. This can come in different formats,
but it often has a `.iso` file extension. Let's get the VyOS image we'll use for
our router/firewall VMs.

Go to [https://vyos.io](https://vyos.io/) and look for the "Download" link in the
main menu. Select “Free Download” from the dropdown menu, and look for the list
item that says “[vyos-rolling-latest.iso](https://s3-us.vyos.io/rolling/current/vyos-rolling-latest.iso).”
Instead of downloading it, however, just copy the link URL.

!!! info

    A quick explanation on the VyOS release models, excerpted from [the project's
    FAQs](https://support.vyos.io/en/support/solutions/articles/103000096261-what-s-vyos-release-model-):

    VyOS is split into two branches: long term support and rolling release.

    * The rolling release branch (git branch “current”) includes the latest code
    from maintainers and all contributions from community members are merged into
    it. It’s meant for testing and home lab/non-critical router use and is not
    guaranteed to be stable.

    * Long term support branches are periodically split from the current branch.
    They are stable, and only proven, strictly compatible changes are merged or
    backported into it. Use this for production networks.

    For our home lab use, we will be using the rolling releases, and in a later
    video, we'll cover how to regularly update to the latest rolling release as
    new features become available.

Go to the “local” storage view of your Proxmox hypervisor and click on "ISO
Images". Click on the “Download from URL” button and paste the link URL we just
copied into the "URL" field of the dialog box.

Click the “Query URL” button to verify the file exists and click the “Download”
button to save the latest VyOS rolling release ISO file locally on your
hypervisor. Keep the file name the same.

Whenever we need to create a new VyOS VM, we'll be able to easily use this local
image already stored in Proxmox. Let's do that now.

## Step 2: Creating and Configuring a VM for VyOS

The next step is to create and configure the actual VM that VyOS will run within.
For this we'll model the VM after a physical router device with five ports.

On the top-right of the Proxmox UI, click the “Create VM” button to set up a
blank VM configuration. This will bring up a dialog box and walk you through the
initial VM setup process:

* On the "General" tab, name your VM `vyos-r1` in the "Name" field, then click
the "Next" button.

* On the "OS" tab, select `Use CD/DVD disc image file (iso)` with "Storage" as
`local` and "ISO Image" as `vyos-rolling-latest.iso.` Click the "Next" button.

* On the "System" tab, select `VirtIO SCSI single` for "SCSI Controller" if
available (use default otherwise) and check the "Qemu Agent". Click the "Next"
button.

* On the "Disks" tab, change “Disk size (GiB)” to `10`, and leave the rest as the
defaults. Click the "Next" button.

* On the "CPU" tab, use `1` for the "Sockets and Cores" fields. Click the "Next"
button.

* On the "Memory" tab, use `1024` for the “Memory (MiB)” field. Click the "Next"
button.

* On the "Network" tab, ensure “No network device” is unchecked, select `vmbr0`
for Bridge, uncheck "Firewall", select `VirtIO (paravirtualized)` for "Model",
and use `auto` for "MAC address". Click the "Next" button.

* On the "Confirm" tab, review all your settings and click the "Finish" button to
close the dialog box. The VM is now created, but there's more configuration to
complete.

Select the `vyos-r1` VM and open the "Hardware" configuration. A router isn’t very
useful with just one interface, so let’s add four more. Within the hardware list,
click the "Add" drop-down button and select “Network Device.”

Using the same configuration as before on the "Network" tab above, create four
additional network devices, so our router will have five total ports to work with.

Well done; you've just modeled a real-world router as a VM!

## Step 3: Installing the VyOS Image on a VM

Now we’re ready to install VyOS onto our VM. Click the “Start” button to power on
the VM and boot up the VyOS installer. When VyOS is finished booting, enter the
username `vyos` and password `vyos` to log in.

The VyOS installer is also a live demo instance of a running router, but we want
to get it installed in our VM. At the command prompt, type `install image` to
begin the installation.

The installation script will begin prompting you to for each step of the process.
The options will be in parenthesis and separated with a slash, such as (Yes/No),
and the default choice will also appear in square brackets, such as [Yes]. To
accept the default choice, simply press the "Enter" key, or type out another
option then press "Enter". At the first question prompt, press "Enter" to
continue.

The next prompt asks how to partition the VM drive used to store VyOS. Press
"Enter" to accept the "Auto" method.

There will only be one drive available, so press "Enter" to select this drive to
install VyOS on. You will again be prompted to confirm any existing data on the
drive could be destroyed. This time, type `Yes` and press "Enter" to continue.

Next you will be asked what size the root partition should be. Press "Enter" to
accept the default size.

The VyOS filesystem will be created on the disk, then you will be prompted on what
to name this VyOS installation. Press "Enter" to accept the default name.

Next, you will be prompted to select which configuration template to start with.
Press "Enter" to accept the default configuration.

The next two prompts will be to create and confirm the password for the initial
administrator user, `vyos`. Select something secure and document it somewhere
safe, as you’ll need this to log into VyOS each time, then press "Enter" to
continue.

Finally, the last prompt will ask which drive should be modified for the boot
partition. Press "Enter" to access the default drive.

At this point, VyOS is now installed. Type `reboot` and confirm to proceed with
rebooting the VM by typing `y` and pressing "Enter". When the VM reboots, you'll
be ready to perform the initial configuration.

## Conclusion and Next Lab

In this lab we created a VM and installed an operating system on it. Well done!

You probably noticed there were a lot more steps to do this compared to creating
the CTs we used for our network hosts. We'll detail the differences in the server
labs section later, but for now, this process is enough to get good at.

We'll be creating many router VMs, so be sure to become comfortable with this
process. Feel free to practice performing the process multiple times, deleting
the VM after each installion.

We've still got work to do on our router VM before we can use it in our network,
so once you're ready to move on, head over to the [Initial VyOS Configuration Best
Practices](initial-vyos-configuration-best-practices.md) lab.
