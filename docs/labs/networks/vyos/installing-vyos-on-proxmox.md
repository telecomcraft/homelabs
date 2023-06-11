The first task is installing VyOS as a VM:

1. Go to [https://vyos.io](https://vyos.io/) and look for the Download link in the main menu. Select “Free Download” from the dropdown menu, and look for the list item that says “[vyos-rolling-latest.iso](https://s3-us.vyos.io/rolling/current/vyos-rolling-latest.iso).” Instead of downloading it, however, just copy the link URL.
2. Go to the “local” storage page of your Proxmox hypervisor and click on ISO Images.
3. Click on “Download from URL” and paste the link URL we just copied into the URL field.
4. Click the “Query URL” button to verify the file exists and click the “Download” button to save the latest VyOS rolling release ISO file locally on your hypervisor. Keep the file name the same.
5. Click on “Create VM” to setup a blank VM configuration.
6. On the General tab, name your VM “vyos-r1” in the Name field, then click Next.
7. On the OS tab, select “Use CD/DVD disc image file (iso)” with Storage as “local” and ISO Image as “vyos-rolling-latest.iso.” Click Next.
8. On the System tab, select “VirtIO SCSI single” for SCSI controller if available (use default otherwise) and click “Qemu Agent.” Click Next.
9. On the Disks tab, change “Disk size (GiB)” to 10, and leave the rest as the defaults. Click Next.
10. On the CPU tab, use 1 for the Sockets and Cores fields. Click Next.
11. On the Memory tab, use 1024 for the “Memory (MiB)” fielf. Click Next.
12. On the Network tab, ensure “No network device” is unchecked, select “vmbr0” for Bridge, uncheck Firewall, select VirtIO (paravirtualized) for Model, and use “auto” for MAC address.
13. On the Confirm tab, review all your settings and click Finish to close the dialog box.
14. Select the VyOS VM and open the Hardware configuration. A router isn’t very useful with just one interface, so let’s add four more. Within the hardware list, click the Add drop-down button and select “Network Device.” Using the same configuration as before in Step 12, create four additional network devices, so our router will have five total routers to work with.
15. Now we’re ready to install VyOS onto our VM. Click the “Start” button to power on the VM and boot up the VyOS installer. When VyOS is finished booting, enter the username `vyos` and password `vyos` to log in.
16. The VyOS installer is also a live demo instance of a running router, but we want to get it installed in our VM. At the command prompt, type `install image` to begin the installation.
17. The installation script will begin prompting you to for each step of the process. The options will be in parenthesis and separated with a slash, such as (Yes/No), and the default choice will also appear in square brackets, such as [Yes]. To accept the default choice, simply press the Enter key, or type out another option then press Enter. At the first question prompt, press Enter to continue.
18. The next prompt asks how to partition the VM drive used to store VyOS. Press Enter to accept the Auto method.
19. There will only be one drive available, so press Enter to select this drive to install VyOS on. You will again be prompted to confirm any existing data on the drive could be destroyed. This time, type `yes` to continue.
20. Next you will be asked what size the root partition should be. Press Enter to accept the default size.
21. The VyOS filesystem will be created on the disk, then you will be prompted on what to name this VyOS installation. Press Enter to accept the default name.
22. Next, you will be prompted to select which configuration template to start with. Press Enter to accept the default configuration.
23. The next two prompts will be to create and confirm the password for the initial administrator user, `vyos`. Select something secure and document it somewhere safe, as you’ll need this to log into VyOS for the first time.
24. Finally, the last prompt will ask