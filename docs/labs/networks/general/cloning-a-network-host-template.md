# Cloning a Network Host Template

In the [Creating a Network Host Template](), you learned how
to create a custom template of Ubuntu Linux to use as a network
host for labs.

In this lab, you'll learn how to clone and configure that
template so it's ready to easily deploy new network hosts in
your labs.

## Step 1: Create the Cloned Network Host

* To clone the template, select the `host0` CT template in the list to bring up
that CT's views.
* Click the `More` drop-down button at the top-right of the Proxmox VE window, then
click `Clone`.
* Select the target node (if you have a Proxmox VE cluster), otherwise leave the
default selected.
* Type `host1` as the Hostname to identify this node.
* Select `Full Clone` as the Mode to create a full and unlinked copy of the template.
* Click the `Clone` button to proceed,

Clone Dialog

## Step 2: Configure the Cloned Network Host

* Power on
* 
