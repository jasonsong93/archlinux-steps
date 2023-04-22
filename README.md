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

To set the keyboard layout, pass a corresponding file name to loadkeys(1), omitting path and file extension. For example, to set a German keyboard layout:  
```
loadkeys de-latin1
```

## 2. Verify the boot mode
To verify the boot mode, list the efivars directory:  
```
ls /sys/firmware/efi/efivars
```
If the command shows the directory without error, then the system is booted in UEFI mode.

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
Use timedatectl to ensure the system clock is accurate:
```
timedatectl
```
Use this to list all the timezones, and then set your corresponding timezeone. For example, mine will be Australia/Melbourne.
```
timedatectl list-timezones
timedatectl set-timezone Australia/Melbourne
```