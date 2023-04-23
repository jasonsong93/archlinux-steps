# Arch Linux Installation 
Personal notes on how to actually set this up, following the official ArchWiki.  
This assumes that you have already acquired an ISO installation image which can be found below.

Download links:  
[Arch Linux Downloads](https://archlinux.org/download/)  
[Rufus to burn ISO](https://rufus.ie/en/)

## 0. My setup
Just wanted to note what setup I had, since it may change some installation steps for you:
- CPU: AMD Ryzen 7 2700x  
- GPU: NVIDIA RTX 3090
- Mouse: Logitech G502 Hero
- Keyboard: Keychron Q3
- Motherboard: Aorus x470


## 1. Set the console keyboard layout
Use this to check a list of keyboard layouts  
```
ls /usr/share/kbd/keymaps/**/*.map.gz
```

To set the keyboard layout, pass a corresponding file name to loadkeys, omitting path and file extension. For example, to set a German keyboard layout:  
```
loadkeys de-latin1
```
I leave this as default and don't change anything (US).

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
If unsure which one to use, use `gdisk` (explained [here](https://unix.stackexchange.com/questions/104238/fdisk-vs-parted)). I will experiment with this when I have time, but it works fine for now.

### **Create a partition table and partitions**
To use gdisk, run the program with the name of the block device you want to change/edit. This example uses `/dev/nvme0n1`:
```
gdisk /dev/nvme0n1
``` 
### **Create new table**
To create a new GUID Partition Table and clear all current partition data, type `o` at the prompt. Skip this step if the table you require has already been created.

### ***Create partitions***
Create a new partition with the n command. You must enter the partition number, first sector, last sector and the partition type.

In my example, I'll do the following:

| Mount Point |          Partition          |     Partition Type    |                Suggested Size                 |
| ----------- | --------------------------- | --------------------- | --------------------------------------------- |   
| `/mnt/boot` | `/dev/efi_system_partition` | EFI system partition  | 1GB since I might use Windows (~300MB if not) |
| `[SWAP]`    | `/dev/swap_partition`       | Linux swap            | More than 512 MiB                             |  
| `/mnt`      | `/dev/root_partition`       | Linux x86-64 root (/) | Remainder of the device                       | 

[Do I need swap?](https://chrisdown.name/2018/01/02/in-defence-of-swap.html)

Follow the prompts to create the various partitions. Follow [this](https://wiki.archlinux.org/title/GPT_fdisk#gdisk_EFI_application) link for the Hex code/GUID as well as further information.


## 6. Format the Partitions
For example, to create an Ext4 file system on /dev/root_partition, run:
```
mkfs.ext4 /dev/root_partition
```
If you created a partition for swap, initialize it with mkswap:
```
mkswap /dev/swap_partition
```

If you created an EFI system partition, format it to FAT32 using mkfs.fat:
```
mkfs.fat -F 32 /dev/efi_system_partition
```

## 7. Mount the File Systems 
Mount the root volume to /mnt. For example, if the root volume is /dev/root_partition:
```
mount /dev/root_partition /mnt
```
For UEFI systems, mount the EFI system partition:
```
mount --mkdir /dev/efi_system_partition /mnt/boot
```
If you created a swap volume, enable it with `swapon`:
```
swapon /dev/swap_partition
```

If you run `lsblk`, you should see all the partitions and their corresponding mountpoints.

## 8. Update mirrors for Installation and Downloads
To enable mirrors, edit /etc/pacman.d/mirrorlist and locate your geographic region. Uncomment mirrors you would like to use.
[This](https://archlinux.org/mirrorlist/) link auto-generates an mirrorlist file for you based on the country you choose.

Edit the mirrorlist by using:
```
vim /etc/pacman.d/mirrorlist
```

During installation, I prefer to use this command. It selects the HTTPS mirrors synchronized within the last 24 hours and located in either Australia (you can select multiple countries with a comma such as Australia,France), sort them by download speed, and overwrite the file /etc/pacman.d/mirrorlist with the results:
```
reflector --country Australia --age 24 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

## 9. Install Essential Packages
*Note: No software or configuration (except for /etc/pacman.d/mirrorlist) get carried over from the live environment to the installed system.*

Use the `pacstrap` script to install the `base` package, Linux `kernel` and firmware for common hardware.

The Arch Linux page recommends the following:
> The base package does not include all tools from the live installation, so installing more packages may be necessary for a fully functional base system. To install other packages or package groups, append the names to the pacstrap command above (space separated) or use pacman to install them while chrooted into the new system. In particular, consider installing:
>- userspace utilities for the management of file systems that will be used on the system,
>- utilities for accessing RAID or LVM partitions,
>- specific firmware for other devices not included in linux-firmware (e.g. sof-firmware for sound cards),
>- software necessary for networking (e.g. a network manager or a standalone DHCP client, authentication software for Wi-Fi, ModemManager for mobile broadband connections),
>- a text editor,
>- packages for accessing documentation in man and info pages: man-db, man-pages and texinfo.

There's a LOT of information on the internet for what to install.
I'd rather not be prescriptive, so here's what I think are essential (and reasons):
- `git`: Version control system
- `base-devel`: it has a lot of tools, which would get installed randomly by other packages you might want
- `neofetch`: show off that "I use Arch btw"
- `neovim`: I use this as my primary editor, feel free to use whatever you would like
- `openssh`: In case you need to perform remote work 
- `man-db, man-pages` and `texinfo`: Specified above by Arch (documentation and man pages)

```
pacstrap -K /mnt base linux linux-firmware neovim base-devel neofetch openssh man-db man-pages texinfo
```

I will add more to this list over time for things I find essential. Note that later on in installation, we will have more chances to configure packages.

## 10. Configure the System
### **Generate an FStab file**
Generate an fstab file (use -U or -L to define by UUID or labels, respectively):
```
genfstab -U /mnt >> /mnt/etc/fstab
```

### **Chroot**
Change root into the new system:
```
arch-chroot /mnt
```

### **Time zone**
Set the time zone:
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
In my case,
```
ln -sf /usr/share/zoneinfo/Australia/Melbourne /etc/localtime
```

Run hwclock(8) to generate `/etc/adjtime`:
```
hwclock --systohc
```

### **Localization**
Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed locales.  
In my case, I uncommented `en_AU.UTF-8 UTF-8`

Then generate the locales by running:
```
locale-gen
```

Create the `locale.conf` file in `/etc/locale.conf` and update with:
```
LANG=en_US.UTF-8
```
You could also run the following, but I usually just manually create it in the previous steps:
```
localectl set-locale LANG=en_US.UTF-8
```

You can also choose to change the keyboard layout, but I don't modify this since I leave it as default (refer to [step 1](#1-set-the-console-keyboard-layout)). If you set the console keyboard layout, make the changes persistent in [`vconsole.conf`](https://man.archlinux.org/man/vconsole.conf.5). 


### **Network Configuration**
Create the hostname file `/etc/hostname` and edit it with your new hostname. See [this](https://datatracker.ietf.org/doc/html/rfc1178) link for how to come up with a good hostname.
You can also use the following command:
```
hostnamectl set-hostname myhostname
```

We then need to configure the hosts. Edit the `/etc/hosts` file. It should look something like 12:22 of [this](https://youtu.be/FFXRFTrZ2Lk?t=742) video.

I need to double check the following packages, but install these (according to that video) for networking:
```
pacman -S efibootmgr networkmanager network-manager-applet wireless_tools wpa_supplicant dialog os-prober mtools dosfstools linux-headers
```

### **Initramfs**
Not sure if this is 100% required, follow up. Creating a new initramfs is usually not required, because mkinitcpio was run on installation of the kernel package with pacstrap.
```
mkinitcpio -P
```

### **Root Password**
Set the root password with the `passwd` command

### **Install Bootloader**
I will install [systemd](https://wiki.archlinux.org/title/systemd-boot) - feel free to use GRUB if that's your preferred option. 
```
bootctl --path=/boot install
```
We need to modify 2 files here. One is `loader.conf` located under `/boot/loader/`. It should look like this:
>timeout 3
>
> #console-mode keep
>
>default arch-*

We also need to create `/boot/loader/entries/arch.conf`.

If you have an Intel or AMD CPU, enable microcode updates in addition. Since I have an AMD CPU, I did the following:
```
nvim /boot/loader/entries/arch.conf
```
and added the following block to the file:
>title   Arch Linux
>
>linux   /vmlinuz-linux
>
>initrd  /initramfs-linux.img
>
>options root=/dev/`root-partition` rw

We also need to allow network manager to start on boot.
```
systemctl enable NetworkManager
```

Let's create a user while we are here.



## 11. Reboot
Congratulations! You've made it to the end of the install.
Exit the chroot environment by typing exit or pressing Ctrl+d.
Optionally manually unmount all the partitions with umount -R /mnt.

Finally, restart the machine by typing reboot: any partitions still mounted will be automatically unmounted by systemd. Remember to remove the installation medium and then login into the new system with the root account.

But wait ... there's more.

## 12. System Administration
Once you reboot, log in as the `root` user.
It's not recommended to use the root user, so let's create another user with elevated privileges.

Use the following command to create a user with a group, replacing `username` with whatever you prefer:
```
useradd -mG wheel username
```
Give your user a password
```
passwd
```
Now we need to ensure that group is a super user. Type the following command (feel free to replace nvim with nano or any other editor of your choice).
```
EDITOR=nvim visudo
```
Find the comment that says 
> \##Uncomment to allow members of group wheel to execute any command 
>
> \# %wheel ALL=(ALL) ALL

Reboot the machine, and log back in.

## 13. Install Graphics Drivers
The commands will differ depending on what card you have.

>Intel:
```
sudo pacman -S xf86-video-intel
```

>AMD:
```
sudo pacman -S xf86-video-amdgpu
```

>Nvidia:
```
sudo pacman -S nvidia nvidia-utils
```
Note: Remove kms from the HOOKS array in /etc/mkinitcpio.conf and regenerate the initramfs. This will prevent the initramfs from containing the nouveau module making sure the kernel cannot load it during early boot.
