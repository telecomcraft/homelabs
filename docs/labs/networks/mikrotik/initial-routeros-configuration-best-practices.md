---
tags:
- MikroTik
- RouterOS
---

# Initial RouterOS Configuration Best Practices

After installing RouterOS in a VM, we’re ready to perform the initial configuration.

Log in as the user `admin`, without a password.

## Secure the Admin User

Using the default account for device operation is almost never a good idea, so
let’s create a new admin user with all privileges and remove the `vyos` account.

1. `set system login user <user_name> full-name "<full_name>"`
2. `set system login user <user_name> class super-user`
3. `set system login user <user_name> authentication plain-text-password`
4. Enter secure password twice when prompted
5. `commit`
6. `quit`
7. Log back into VyOS using the new account credentials to verify
8. Re-enter configuration mode with `configure`

## Configure a Management Interface

Every network device should have a management interface, ideally out-band-band
from the rest of the interfaces, to securely operate the device. For VyOS, let's
standardize on the `eth0` interface and have it get an address via DCHP so we
can use that address to access the device via SSH.


```
set interfaces fxp0 unit 0 family inet dhcp
```

## Enable SSH access

Instead of accessing vSRX using a console connection or hypervisor virtual terminal, we can now use SSH through the management interface. Enable it with:

1. `set system services ssh`

## Set host name

## Set domain name

## Set DNS servers

## Set static default route with no-readvertise option

You should be as specific about the route as possible. You can also use the
no-readvertise option for the static route used for management traffic. This marks
the route ineligible for readvertisement through routing policy.

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
