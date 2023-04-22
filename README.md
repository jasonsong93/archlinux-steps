# Arch Linux Installation 
Personal notes on how to actually set this up, following the official ArchWiki.  
This assumes that you have already acquired an ISO installation image which can be found below.

Download links:  
[Arch Linux Downloads](https://archlinux.org/download/)  
[Rufus to burn ISO](https://rufus.ie/en/)

## 1. Set the console keyboard layout
Use this to check a list of keyboard layouts  
```
ls /usr/share/kbd/keymaps/**/*.map.gz
```

To set the keyboard layout, pass a corresponding file name to loadkeys, omitting path and file extension. For example, to set a German keyboard layout:  
```
loadkeys de-latin1
```
I left this as default.

## 2. Verify the boot mode
To verify the boot mode, list the efivars directory:  
```
ls /sys/firmware/efi/efivars
```
If the command shows the directory **without** error, then the system is booted in UEFI mode.

## 3. Connect to the internet
Ensure your network interface is listed and enabled:
```
ip link
```
We can configure DHCP (provided by systemd-networkd and systemd-resolved for Ethernet) or a Static IP address.  
For more information regarding Static IP addresses, click [here](https://wiki.archlinux.org/title/Network_configuration#Static_IP_address)  

If using Wi-Fi, follow [**iwctl**](https://wiki.archlinux.org/title/Iwd#iwctl).

Otherwise, to test if the connection is successful, run 
```
ping archlinux.org
```

> **_Note_**: In the installation image, systemd-networkd, systemd-resolved, iwd and ModemManager are preconfigured and enabled by default. That will **not** be the case for the installed system. UPDATE THIS <- DO I NEED TO INSTALL THESE LATER FOR INTERNET TO WORK?

## 4. Update the System Clock
Use `timedatectl` to ensure the system clock is accurate.

List all timezones and then set your appropriate timezone. For example, mine will be Australia/Melbourne:
```
timedatectl list-timezones
timedatectl set-timezone Australia/Melbourne
```

## 5. Partition the disks
### **List the disks**
Disks are assigned to a block device such as /dev/sda, /dev/nvme0n1 or /dev/mmcblk0. To identify these devices, use `lsblk ` or `fdisk`. I prefer `lsblk` here.
```
lsblk
```
or
```
fdisk -l
```

My target drive is an nvme drive, and is located at `nvme0n1`

`lsblk` output looked like (I have my 2 SSDs, 1 bootable USB and my NVMe where the OS is going to be installed):
```
sda 
|--sda1
|--sda2
sdb 
|--sdb1
sdc (the USB)
|--sdc1
nvme0n1 (primary OS)
|--nvme0n1p1
|--nvme0n1p2
```


### **Partitioning the disks**
The following partitions are required for a chosen device:
- One partition for the root directory /.
- For booting in UEFI mode: an EFI system partition. This is if you followed [step 2](#2-verify-the-boot-mode) and you are in UEFI mode.

The following is not required, but recommended (I will be doing this, since I have lots of spare space on my NVMe):
- Swap space: can be set on a swap file for file systems supporting it. 

If you want to create any stacked block devices for LVM, system encryption or RAID, do it now (I don't really know what this is at the moment, writing here for future reference).

Use fdisk or parted/gdisk to modify partition tables. For example:
```
fdisk /dev/the_disk_to_be_partitioned
```

If unsure which one to use, use gdisk (explained [here](https://unix.stackexchange.com/questions/104238/fdisk-vs-parted)). I will experiment with this when I have time, but it works well for now.

```
gdisk -l /dev/nvme0n1
```
