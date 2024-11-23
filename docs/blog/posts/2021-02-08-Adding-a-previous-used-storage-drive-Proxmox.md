---
title: Adding a previously used storage drive to Proxmox
description: How to add a previously used storage drive to Proxmox
date: 2021-02-08
categories:
  - Proxmox
  - Tutorials
---
# Adding a previously used storage (i.e. physical) drive to Proxmox

If you want to use an old hard drive (or SSD, etc…) then it needs to be blank for Proxmox to accept it and offer it up as an option for adding as a LVM or LVM-thin.

To make it visible and valid, you need to wipe all the partitions. You can do this from the shell within Proxmox but there is a subtle difference between NVME devices and SATA devices.

To start, you can list all devices by typing the following in the shell:

```bash
fdisk -l
```

For SATA drives, you would use /dev/sda, /dev/sdb, etc… but for NVME drives, you use the namespace so /dev/nvme0n1. For this example, I'll be using an NVME drive but replace the value as needed for you situation.

Type the following command into the shell:

```bash
cfdisk /dev/nvme0n1
```

Press a key to acknowledge the warning. You may be prompted to choose a label type. For ease, pick `gpt`.

Next highlight any existing partitions and delete them. Once they're all deleted, choose the `Write` option and then `Quit`. The drive should now be available to add in Proxmox under the `Create: Volume Group` and `Create: Thinpool` options.