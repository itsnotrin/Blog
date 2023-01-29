---
title: "[Part 1- Setup] - New ArchLinux Workstation!"
date: 2023-01-29T1133:39+01:00
description: "It's that time again! Time to say 'screw you' Microsoft!"
draft: false
---

# New Arch Linux Workstation [Part 1- Setup]

## Why Linux?

Personally, I've used Windows on my Main PC for a while- this is for a variety of reasons- mostly just because of Games that I play not being supported on linux though.
And whilst it's worked, as I call it all the time, "winblows" sucks. I've used Arch linux amongst other distros on my Laptops for a while now and I'm finally ready to commit to Arch on my PC too.
Valorant and Warzone 2 are currently the only games that I play that don't support Linux.

As for Valorant, as I play it competitively, I used NTLite to create an extremely stripped down version of Windows that's dedicated just to running Valorant and absolutely nothing else... at all.
It nets me amazing performance and I couldn't be happier, I've thrown it on a secondary 240GB SSD I had spare and that's all running happily.

As to the rest of the apps and games that I play which **require** Windows (don't work in Wine, Proton or anything), I'll setup a Windows VM with GPU passthrough, but that's for another part of this Guide.
My ideal goal is to have as much as possible on Linux- only Sony Vegas, Adobe Photoshop & After Effects as well as Warzone 2 will be in the Windows VM as none of these will run under Linux.

## Where to start?

** MAKE BACKUPS!!!!!!**
First off, I made a backup of anything that I'll need to take across from Windows- MobaXTerm Configurations and Servers, SSH keys, important files ETC.
Anything else is either on my Nextcloud instance, stored in a backup I can just grab the files from (Thanks UrBackup) or linked to my FireFox account.

Now that this is done, head to the Arch Linux website and Download the Newest version- `2023.01.01` at the time of writing however the steps below will probably be the same for you.

Use a tool of your choice (Rufus, Balena Etcher etc etc) to flash the ISO to a USB Drive. Once that's completed, boot from it!

As I'm using my PC, I'm on Ethernet and Wi-Fi isn't something that I need to worry about nor configure.
I'd reccomend using Ethernet for the install process and then after that you can configure Wi-Fi in your Window Manager / Desktop Environment of choice.

## In the ISO

### INFO
- We'll be using EXT4 as I don't need backups from BTRFS or any of it's other advantages becuase I run a backup server already.
- I'll be installing GNOME
- This is not a Copy-Paste guide, please be sure to change out things such as Disk names for the ones correct to your system.
- I will be installing as UEFI. This guide will not work on non-UEFI systems.

### Checking your Network Connection
Once you're booted into the ISO and connected to Ethernet, run a ping command to check your network connection- I opted to ping Google with the following command: `ping google.com -c 3`.
If all is good, you'll get responses and there won't be a problem!

### Setting your KeyMap
Now, set your keymap if you don't have a US style keyboard, I do so I don't need to do this, but to change it use the following commands:
```sh
localectl list-keymaps
# Here, you can press the / key and search for your keymap, or you can just hold enter to scroll down and find it.
# Once you've found your Keymap, do Ctrl + C to break out of the command then run the following:
loadkeys [keymap]
# Be sure to repleace [keymap] with the correct option.
```

### Setting the Time
This is thankfully a quick and easy process.
`timedatectl set-ntp true`

### Partitioning the Disk
To find out your disk's name, run `fdisk -l`
It will spit out a list of every drive in your System. Look for the one that you want to install to- in my case, I want to install to my 1TB Sabrent Rocket Q NVME drive- the only NVME in my system, so I can recognise it by the 931GB size as well as the nvme0n1 name.
I now know that I need to format the device `/dev/nvme0n1`

For installing with BTRFS, we need at least 2 partitions.
* /boot/efi partition (300MB)
* / (Root partition) (Rest of the size of your drive.)

However, I will also add a Swap partition- the general consensus that I follow is you should have your swap be RAM amount * 2 (Up until 8GB)
As I have 32GB of RAM, this x2 is above 8GB, So that means I will be making an 8GB Swap Partition.

** BE WARNED, WHAT WE DO NOW WILL REMOVE ALL DATA ON THE DRIVE, NOTHING WILL STAY WHATSOEVER, PLEASE ENSURE YOU'VE BACKED UP YOUR FILES. THEY WILL NOT BE RECOVERABLE**

Given the warning above, if you're certain you're ready to continue type `g` in your fdisk prompt to create a new GPT disklabel.

#### EFI Partition
Now, press type `n` and then Enter. This will be our EFI partition.
Give it the Partition Number 1 (The default is this anyway)
Use the default first sector, and then for the "Last Sector" entry, type `+300M` and then press Enter.

#### SWAP Partition
Now, hit `n` again and then Enter one more time. This partition is going to be our Swap.
Give it the Partition Number 2 (Again, the default)
Again, the default first sector, and then for the "Last Sector" this time, we type +8GB

#### Root Partition
For the last time, hit `n` and press Enter. This partition is going to be our final one- the Root filesystem.
Give it the partition number 3- the default one yet again.
Once again, the default first sector and then for the Last Sector this time, leave it blank, this will automatically fill up the rest of your space.

#### Partition Types
You might've noticed but all of the partitions we just created were made with the type 'Linux fileysstem'- this is incorrect for our needs.
You can type `l` to list all the partition types if you want but I'll state the ones needed below anyway.

Type `t` and then enter and choose our EFI partition (Number 1), to do this, type the number 1 and then enter.
We want this to be our EFI partition and EFI has the partition type number `1`, so here we will enter 1 again and then enter.

Now, do the same but this time select Partition 2. It is our swap partition.
Since this is swap, choose partition type number `19`. This will format it as Linux Swap.

#### Ensuring it all worked
To save our changes, we now type `w`. All being well, it should hopefully save without any issues, if there is an issue, just reboot and try again.

Run fdisk -l yet again to see if you can see the partitions under your disk.
You should be able to see that your partitions were made correctly.


### Creating Filesystems
Using the output from `fdisk -l`, note down which 'Device' shows for your EFI System, Linux Swap and Linux Filesystem. For me, it's as follows:
`/dev/nvme0n1p1` - EFI
`/dev/nvme0n1p2` - Swap
`/dev/nvme0n1p3` - Root FS

Now, to make our EFI usable, we need to format it as Fat32, to do this, run the following command
```sh
mkfs.fat -F32 /dev/nvme0n1p1 # Be sure to replace nvme0n1p1 with the correct device for your system.
```

For the Swap partition, we too need to format it as Swap. Use the following commands to do so:
```sh
mkswap /dev/nvme0n1p2
swapon /dev/nvme0n1p2 # Be certain to check this is the correct device name.
```

Now, to make our root FS btrfs, we need to use the following command:
```sh
mkfs.ext4 /dev/nvme0n1p3 # One last time, BE SURE TO CHANGE THIS TO THE CORRECT DEVICE
```

### Mounting the Filesystems
Now, we need to mount the Filesystem. This can be done with the following command:
```sh
mount /dev/nvme0n1p3 /mnt # BE SURE TO CHANGE OUT /dev/nvme0n1p3 WITH THE CORRECT DEVICE!!!
```

To mount our EFI partition, run this command too:
```sh
mount --mkdir /dev/nvme0n1p1 /mnt/boot # Yet again, please ensure you have the right device here.

```

### Installing the base system

Now that we have these mounted, you need to run the following:
For intel CPUs:

```sh 
pacstrap /mnt base linux linux-firmware nano intel-ucode sudo
```

For AMD CPUs:

```sh
pacstrap /mnt base linux linux-firmware nano amd-ucode sudo
```

This command may take a few minutes to complete, depending on your connection.

### Configure the install

#### Generating FSTab
First of all, we need to generate an FSTab file- this defines how partitions, block devices or remote file systems are mounted into the FileSystem.
Use the following command to do so:
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

Now, we want to load into the install to configure it further- use the following command to change into it.
```sh
arch-chroot /mnt
```

#### Configure Locale
This is what sets the language, numbering, date, and currency formats for your system.
Open the file using the command:
```sh
nano /etc/locale.gen
```

Find your locale (For me it's en_GB.UTF-8) and uncomment it. (Remove the # at the start)
Now, run all of these:
```sh
locale-gen
echo LANG=en_GB.UTF-8 > /etc/locale.conf
export LANG=en_GB.UTF-8
```

#### Network and Hostname Configuration
Create a /etc/hostname file and add the hostname entry to this file. Hostname is basically the name of your computer on the network.
In my case, I’ll set the hostname as `ryan-arch`. You can choose whatever you want:
```sh
echo ryan-arch > /etc/hostname
```

The next part is to create the hosts file:
```sh
touch /etc/hosts
```
Now, open this file by doing `nano /etc/hosts` and replace it with the following
```txt
127.0.0.1	localhost
::1		localhost
127.0.1.1	ryan-arch
```
**NOTE:** BE SURE TO REPLACE ryan-arch WITH YOUR HOSTNAME!

#### Set the Timezone
```sh
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
For me, My timezone is Europe/London so the command will be:
```sh
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
```

Also be sure to run `hwclock --systohc`.

#### Set the Root Password
You can do this by typing `passwd` and choosing a suitable password. Note that this is not the password you will be signing in with everyday.

#### Installing Grub Boot Loader
Ensure that you haven't accidentally exited the chroot and then run the following command:
```sh
pacman -S grub efibootmgr
```

Now, we need to mount our EFI partition we made before- for me, the device name is /dev/nvme0n1p1 but it may well be different for you, please look at your notes I asked you to write earlier in order to find out.
```sh
mkdir /boot/efi
mount /dev/nvme0n1p1 /boot/efi
```

Then, install Grub like this:
```sh
grub-install --target=x86_64-efi --bootloader-id=Grub --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```

#### Creating a User account
Since we already installed sudo when we did the pacstrap, we do not need to do so again.
Simply run the following command, be sure to change out the username for what you want:
```sh
useradd -m ryanwiecz
passwd ryanwiecz # Set a password!
```

To allow your user to run Sudo commands, you must do the following:
```sh
usermod -aG wheel ryanwiecz #Again, ensure that this is the correct username.
```

Now we need to give the wheel group sudo permissions, to do that run the following:
```sh
EDITOR=nano visudo
```

Find the commened wheel line near the bottom of the while that says `%wheel ALL=(ALL:ALL) ALL` and uncomment it.
Save the changes with Ctrl + X and then Y.

Then, exit from the chroot by simply running `exit` and then unmount the fs using `umount -l /mnt`

## Inside the OS
Boot into your Arch install from Grub- if it doesn't show up, mount it again and run sudo pacman -S linux then regenerate the grub config with grub-mkconfig again.

### Enable Networking
```sh
sudo systemctl enable --now NetworkManager
```

### Installing Gnome
```sh
sudo pacman -S --needed xorg 

sudo pacman -S --needed gnome gnome-tweaks nautilus-sendto gnome-nettool gnome-usage gnome gnome-multi-writer adwaita-icon-theme xdg-user-dirs-gtk fwupd arc-gtk-theme seahosrse gdm
```
The above installation would ask for several options for packages. Choose any of the ones you want. If you are unsure, choose jack, noto-sans and xdg-portal-desktop-gnome when asked.

Enable gnome
```sh
systemctl enable gdm
```


## And we're done
That's it! You have a fully working system! (Unless you're an NVIDIA user but I'll cover that in the next guide :) )

In the next guide I'll cover installing Nvidia drivers and configuring my OS!
In the final guide, I'll cover delving into custom kernels and the like!

Thank you for reading<3

<!-- ![Example image](/static/image.png) -->
