+++
title = "Installing Linux (native)"
description = "Step-by-step guide on how to install Linux using Native method."
date = 2023-11-22T18:00:00+00:00
updated = 2023-11-22T18:00:00+00:00
draft = false
weight = 10
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = 'Learn how to install Linux using the native method, including preparing your system, partitioning, and system setup.'
toc = true
top = false
+++

## Preparing for Installation
Before starting, ensure that you have a bootable USB drive with the Arch Linux ISO. You can create this drive using tools like Rufus or dd.

## Booting from USB
Insert your USB drive and reboot your computer. Make sure to enter the BIOS or UEFI settings to prioritize booting from USB.

## Live Environment
Once booted from the USB, you will be in the Arch Linux live environment. You can begin the installation process here.

### Set the Keyboard Layout
The default keyboard layout is US. If you need a different layout:
```bash
loadkeys [layout]
```

### Verify the Boot Mode
Check if your system supports UEFI:
```bash
ls /sys/firmware/efi/efivars
```

### Connect to the Internet
Ensure that you are connected to the internet. For a wired connection, it should be automatic. For Wi-Fi:
```bash
iwctl
get-networks
station wlan0 connect <your networks SSID>
```

### Update the System Clock
Ensure the system clock is accurate:
```bash
timedatectl set-ntp true
```

### Partition the Disks
Partition your disk as per your preference using tools like fdisk or parted.

### Format Partitions
Format the partitions. For example, for a root partition:
```bash
mkfs.ext4 /dev/sdaX
```

### Mount the Partitions
Mount your partitions to the /mnt directory.

### Installation base packages
Now, install the base packages:
```bash
pacstrap /mnt base linux linux-firmware
```

### Generate Fstab
Generate an fstab file:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot
Change root into the new system:
```bash
arch-chroot /mnt
```

### Time Zone
Set the time zone:
```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

### Localization
Uncomment en_US.UTF-8 UTF-8 and other needed locales in `/etc/locale.gen`, and generate them:
```bash
locale-gen
```

### Network Configuration
Create a hostname file and configure the hosts file.

### Initramfs
Create a new initramfs:
```bash
mkinitcpio -P
```

### Root Password
Set the root password:
```bash
passwd
```

### Boot Loader
Install and configure a bootloader like GRUB.

### Reboot
Exit chroot, unmount the partitions, and reboot. Remove the USB drive before rebooting.
```bash
exit
sudo reboot
```

Congratulations! You should now have a basic Arch Linux system installed.






