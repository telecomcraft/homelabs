# Installing Juniper vSRX on Proxmox

## Step 1: Obtain a vSRX Image From Juniper

* Log into your Juniper account,
https://www.juniper.net/documentation/us/en/software/junos/srx-upgrade/topics/topic-map/vsrx-upgrade.html

## Step 2: Download the vSRX Image to a Proxmox VE

```
wget "https://cdn.juniper.net/software/vsrx-eval/20.2R1.10/junos-vsrx3-x86-64-20.2R1.10.qcow2?SM_USER=user@example.com&__gda__=1686427804_2f3809fad6ba702dd1e8258c195f9909"
```

`scp junos-vsrx3-x86-64-20.3R1.8.qcow2 <user>@<host>:~`

`qm importdisk <vm-id> junos-vsrx3-x86-64-20.3R1.8.qcow2 local-lvm`

## KVM VM Configuration

The minimum requirements for running vSRX in KVM are

- 2 CPUs
- 4 GB RAM
- 10 GB storage
