# Migrating a Hyper-V VM to Proxmox

I'm sure there are many ways out there to do this but this is how I did it for about 20 VMs.

Firstly, in Hyper-V, I suggest compacting any disks you plan on migrating, especially if you know they're smaller than being reported.

Once that's done, not down CPU, memory and HD sizes as we'll manually need to re-create the VMs in Proxmox.

Choose the Export option and write the exported data to somewhere that will be accessible after installing Proxmox. If you're replacing an existing machine, use either external storage or a NAS/network share. I used the latter.

In the shell of Proxmox, run the following command to get access to the share:

```bash
mkdir /mnt/exports
mount -t cifs -o username=you@yourdomain.com,password=YourPassword123 //yournas/exports /mnt/exports
```

Now you need to set up a VM using the values you noted earlier. If migrating a Windows machine, I've noticed it it defaults to IDE for the hard drive but I've changed it to SATA in each case. If you have multiple drives, add the extra ones once you've set up the VM. For the purposes of this demo, use VM-Local (if you followed my previous setup guide). Take a note of the VM's ID.

Next change to the VM's directory which will be something like:

```bash
cd /mnt/vm-local/images/1xx
```

We'll now copy over and migrate the VM from a VHD/VHDX image to a QCOW2 image using the following command (note your export subfolders will differ) and replace 1xx with your VM's ID:

```bash
qemu-img convert -O qcow2 /mnt/exports/to-import.vhdx vm-1xx-disk-0.qcow2
```

For subsequent disks, repeat the process but increase the number after “disk-” by 1 each time.

Once all the disks are imported, start the VM. Note that things like network cards will have changed so you'll likely need to re-setup your static IPs if you have them.