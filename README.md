# Arch Linux Installation 
Personal notes on how to actually set this up, following the official ArchWiki.  
This assumes that you have already acquired an ISO installation image which can be found below.

Download links:  
[Arch Linux Downloads](https://archlinux.org/download/)  
[Rufus to burn ISO](https://rufus.ie/en/)


## 1. Set the console keyboard layout
Use this to check a list of keyboard layouts  
`ls /usr/share/kbd/keymaps/**/*.map.gz`

To set the keyboard layout, pass a corresponding file name to loadkeys(1), omitting path and file extension. For example, to set a German keyboard layout:  
`loadkeys de-latin1`