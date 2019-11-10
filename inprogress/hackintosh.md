---
layout: post
title: "Triple Boot Hackintosh (Windows, Linux & macOS) Dell XPS 15 9570"
date: 2018-07-25 19:37:00 +0200
---

In my previous post, I explained how to install Linux Mint on the Dell XPS 9570 solely or along with the pre-installted Windows 10. This post explains how to turn your XPS 9570 into a Hackintosh running macOS Mojave. And, because we can, we keep Windows 10 and Linux Mint installed *TODO keep or reinstall?*. So, we end up with a triple boot system running Windows 10, Linux and macOS.


# Prerequisites
- Dell XPS 9570 laptop
- A usb drive (if you have two usb drives then things become a bit more comfortable)
- Linux Mint 19.2 (Tiana)
- The macOS Mojave Installer from Apple's App Store
- A dump of the XPS' factory default disk partitions


# Install Windows 10
I removed Windows 10 and used my xps 9570 with Linux Mint only in the past months. That's why this part explaings how to restore Windows 10 from the disk dump that I created before using my laptop. Of course, you did that too, right? I recommended so in my last post!

- bzcat nvme0n1_first_100M.bz2 | sudo dd of=/dev/nvme0n1 bs=1G
- bzcat nvme0n1p1.bz2 | sudo dd of=/dev/nvme0n1p1 bs=1G
- bzcat nvme0n1p2.bz2 | sudo dd of=/dev/nvme0n1p2 bs=1G
- bzcat nvme0n1p3.bz2 | sudo dd of=/dev/nvme0n1p3 bs=1G

I skipped nvme0n1p[4-6] because these are recovery partitions and we just used our own recory images.

After rebooting the systen, Windows 10 tries to boot but it ends in an error. Don't worry. Retry until Windows offers you to boot into "secured mode". Do so and the Windows setup should appear. That's where Cortana starts speaking and you immediately start searching for the mute button which is on the bottom right. You're welcome.

After setting up Windows, your laptop should boot fine into Windows. First OS done. Two left.


# Install Linux Mint
Before installing Linux Mint we need to reduce the disk space used by Windows 10:
- In Windows, hit  "Windows-Key" and enter "partition". The "create and format disks" screen appears.
- Select your Windows partition
- Right-click and choose "resize"
- Enter the amount of space you want to be available for Linux AND macOS (in sum). The value entered here is the amount of space we make unusable for Windows.
- Insert the Linux Mint usb drive and reboot
- Linux Mint live cd will boot up.
- Begin the installation by starting the "Install Linux Mint" shortcut on the Desktop.
- When asked to *TODO* select *TODO custom configuration TODO*
- Select the empty space following the Windows 10 partition.
- Create two new partitions here. One for Linux Mint where we will instal now. And the other one for macOS. I used ext4 for Linux and fat32 for the macOS partition. Don't worry. We won't install macOS on fat32. But later, the macOS installer will recognize and show the fat32 partition. This way it's easier to select the partition and we will change the filesystem then.
- Continue the installation and refer to the previous post about how to configure Linux Mint to make it running smoothly on your xps 9570.

If you reboot now then you should see the grub bootloader offering you to run Windows 10 or Linux Mint. Second OS done. One left.

# Install macOS

- Create a macOS-installation usb drive:
https://hackintosh.gitbook.io/-r-hackintosh-vanilla-desktop-guide/building-the-usb-installer

- Install Clover on the usb drive. Follow this guide to do so:
https://github.com/bavariancake/XPS9570-macOS

You can now reboot your machine but without the usb drive you will end up in the grub bootloader asking you to choose between Linux Mint and Windows 10. Of course, it is not enough to have Clover on the usb driver to be able to run the installer from the usb drive. Now we need to install clover on our internal disk to make macOS boot on our laptop. Repeat the same steps for installing Clover as you did for the usb drive but this time, have your internal disk as target.

After reboot chances are that you still the grub bootloader installed by Linux Mint. No worry, you have Clover installed in your EFI system parition but probably your system boots the grub's EFI program instead of clover. To fix that:
- press F2 when the DELL logo appears after powering on
- *TODO select boot/boot_x64.efi blabla"*
- reboot

Et voila: Your XPS 9570 should now boot into the Clover bootloader which lets you device to boot Windows 10, Linux or macOS.


## Please drop me a line if you tried these steps. Was it useful? Did you find a better way?


