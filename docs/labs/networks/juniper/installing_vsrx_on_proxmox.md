# Installing Juniper vSRX on Proxmox

## Step 1: Download a vSRX Image From Juniper

Unless you have a licensed copy of vSRX, you will need to download the evaluation version for home lab use from [Juniper's website](https://www.juniper.com).
And in order to do that, you must have an account and authorization to download evaluation software.

If you already can do this, skip ahead to step two. Otherwise, perform the following tasks:

* [Log into or register for your Juniper account](https://iam-signin.juniper.net/).
* Request access to the trial version of vSRX by [following these instructions](https://www.juniper.net/us/en/dm/download-next-gen-vsrx-firewall-trial.html).
* Once your account is authorized by JTAC, [download the trial version of vSRX here](https://support.juniper.net/support/downloads/?p=vsrxeval). For Proxmox VE, choose the vSRX KVM Appliance.
* Download and install your trial license here.



## Step 2: Upload the vSRX Image to a Proxmox VE

From the shell of a user in PVE

```
wget "https://cdn.juniper.net/software/vsrx-eval/20.2R1.10/junos-vsrx3-x86-64-20.2R1.10.qcow2?SM_USER=user@example.com&__gda__=1686427804_2f3809fad6ba702dd1e8258c195f9909"
```

`scp junos-vsrx3-x86-64-20.3R1.8.qcow2 <user>@<host>:~`

## Step 3: Create the VM

The minimum requirements for running vSRX in KVM are

- 2 CPUs
- 4 GB RAM
- 10 GB storage

`qm importdisk <vm-id> junos-vsrx3-x86-64-20.3R1.8.qcow2 local-lvm`