# Cloning a Network Host Template

In the [Creating a Network Host Template]() lab, you learned how to create a
custom template of Ubuntu Linux to use as a network host for labs.

In this lab, you'll learn how to clone and configure that template so it's ready
to easily deploy new network hosts in your labs.

## Step 1: Create the Cloned Network Host

* To clone the template, select the `host0` CT template in the list to bring up
that CT's views.
* Click the `More` drop-down button at the top-right of the Proxmox VE window, then
click `Clone`.
* Select the target node (if you have a Proxmox VE cluster), otherwise leave the
default selected.
* Type `host1` as the Hostname to identify this node.
* Select `Full Clone` as the Mode to create a full and unlinked copy of the template.
* Click the `Clone` button to proceed.

Clone Dialog

Once the cloning is done, you will see a new CT named `host1` in the list.

* Select the `host1` CT and start it up.
* Using the `Console` view, log in with the same non-root administrator account
as you used for `host0`.

## Step 2: Configure the Cloned Network Host

Before using the new network host in labs, there are several changes that have to
be made to make it different from the template. The hostname, machine ID, and
network interface's MAC address are changed as part of the cloning process itself.

However there is one crucial change that you'll have to manually make for each new
host after the clone is created: generating new SSH keys. This is an essential step
for any VM or CT, so always verify this was properly done.

* Run `ls /etc/ssh` to ensure no files starting with `ssh_host` exist.
* Run `sudo dpkg-reconfigure openssh-server` generate new key pairs.
* Run `ls /etc/ssh` again to confirm the creation of the new key pairs.

## Step 3: Create Three More Cloned Network Hosts

Now your network host is ready for use. Remember to always complete the steps above
when creating new hosts. Before we wrap this lab up, you'll get some more practice.

Use the steps above to create three more hosts named `host2`, `host3`, and `host4`,
which we'll be using in the next lab, [Directly Connecting Network Hosts]().
