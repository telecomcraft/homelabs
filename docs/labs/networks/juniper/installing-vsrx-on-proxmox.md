# Installing Juniper vSRX on Proxmox

## Step 1: Download a vSRX Image From Juniper

Unless you have a licensed copy of vSRX, you will need to download the trial version for home lab use from [Juniper's website](https://www.juniper.com).
And in order to do that, you must have an account and authorization to download trial software.

If you already can do this, skip ahead to step two. Otherwise, perform the following tasks:

* [Log into or register for your Juniper account](https://iam-signin.juniper.net/).
* Request access to the trial version of vSRX by [following these instructions](https://www.juniper.net/us/en/dm/download-next-gen-vsrx-firewall-trial.html).
* Once your account is authorized by JTAC, [download the trial version of vSRX here](https://support.juniper.net/support/downloads/?p=vsrxeval). For Proxmox VE, choose the vSRX KVM Appliance.
* Download and save your trial license, but just hold onto this for now. It shouldn't be necessary for most functionality.

## Step 2: Upload the vSRX Image to a Proxmox VE

Next you'll need to upload the vSRX trial image to a Proxmox VE machine in preparation for installation. In this case I did
`scp junos-vsrx3-x86-64-20.3R1.8.qcow2 <user>@<pve>:~` to copy the file to my Proxmox VE user's home folder, which is good enough for this step.

Verify the file was successfully transferred, and we'll revisit this again after the next step.

## Step 3: Create the Virtual Machine

vSRX will run as a VM on Proxmox VE, but it's not ready out of the box. You'll first have to create the VM configuration within Proxmox VE, including naming and hardware settings.

The [minimum requirements for running vSRX in KVM](https://www.juniper.net/documentation/us/en/software/vsrx/vsrx-consolidated-deployment-guide/vsrx-kvm/topics/concept/security-vsrx-system-requirement-with-kvm.html) are

- 2 CPUs
- 4 GB RAM
- 20 GB storage

`qm importdisk <vm-id> junos-vsrx3-x86-64-20.3R1.8.qcow2 local-lvm`

## Step 4: Verify VM Bootup and Initial Login
