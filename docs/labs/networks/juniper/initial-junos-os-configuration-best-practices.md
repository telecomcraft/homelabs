---
tags:
  - Juniper
---

# Initial Junos OS Configuration on a vSRX

After installing vSRX in a VM, we’re ready to perform the initial configuration. Log in as user `root` (no password required), and enter configuration mode with `configure`.

## Set root password

Before making any changes to the factory-installed configuration, you will need to first set a password for the root user. Let’s do that before making any other changes:

1. `set system root-authentication plain-text-password`
2. Enter secure password twice when prompted
3. `commit and-quit`
4. `quit`
5. Log back into Junos OS using the new account credentials to verify

## Create a non-root user

Using the root account for device operation is almost never a good idea, so next let’s create a non-root user with all privileges using the local database:

1. `set system login user <user_name> full-name "<full_name>"`
2. `set system login user <user_name> class super-user`
3. `set system login user <user_name> authentication plain-text-password`
4. Enter secure password twice when prompted
5. `commit and-quit`
6. `quit`
7. Log back into Junos OS using the new account credentials to verify
8. Re-enter configuration mode with `configure`

## Set management interface

Every network device should have a management interface, ideally out-band-band from the rest of the interfaces, to securely operate the device. For the vSRX, it’s the `fxp0` interface. For now, let’s have it get an address via DCHP so we can use that address to access the device from here via SSH:

1. `set interfaces fxp0 unit 0 family inet dhcp`

## Enable SSH access

Instead of accessing vSRX using a console connection or hypervisor virtual terminal, we can now use SSH through the management interface. Enable it with:

1. `set system services ssh`

## Set host name

## Set domain name

## Set DNS servers

## Set static default route with no-readvertise option

You should be as specific about the route as possible. You can also use the no-readvertise option for the static route used for management traffic. This marks the route ineligible for readvertisement through routing policy.

## Set time zone

## Set NTP servers

## Set rescue configuration

You can configure a single logical unit for the lo0 interface for each routing instance, and each logical unit associated with a given routing instance can have multiple configured IP addresses.

### Commit changes

`commit comment "Initial configuration performed"`

## Verification

- Using the Proxmox console, check the management network interface IP address with `run show dhcp client binding fxp0.0`.
    
    ```
    eron@vsrx-r1> show dhcp client binding fxp0.0
    IP address        Hardware address   Expires     State      Interface
    172.16.0.110      a2:2f:9d:45:aa:2a  263         BOUND      fxp0.0
    ```
    
- Using the IP address above, log into the router with the new non-root user via an SSH client with `ssh <user_name>@<ip_address>`.
    
    ```
    PS C:\Users\eronl> ssh eron@172.16.0.110
    Password:
    Last login: Tue Jun  6 02:16:18 2023 from 172.16.0.53
    --- JUNOS 20.3R1.8 Kernel 64-bit XEN JNPR-11.0-20200908.87c9d89_buil
    eron@vsrx-r1>
    ```
