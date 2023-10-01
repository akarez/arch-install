# arch install

## Table of Contents
- [Networks](#networks)
- [Partitions](#partitioning)
  - [Create](#create-partitions)
  - [Format](format-partitions)
  - [Mount](mount-partitions)
- [Base System](base-system)
  - [Install Base Packages](install-base-packages)
  - [Change Working Root](enter-chroot)
  - [Configure Time Zones](configure-time-zones)
  - [Configure Host Information](configure-host-info)
- [Bootloader](bootloader)
- [Finishing Up](finishing-up)
  - [Users and Passwords](users-and-passwords)
  - [Network Manager](network-manager)
  - [Exit Installation](exit-installation)
- [Install Desktop Environment](install-desktop-environment)
  - [Essential Packages](essential-packages)
  - [Optional Packages](optional-packages)
- [Load Configuration](load-configuration)

## Networks
Use the iwd utility to connect to WiFi:

>Note: If you have an ethernet connection you can skip to the next section.

```
~# iwctl
```

List all connected network cards:

```
[iwd]# device list 
```

Scan for available networks:

>Note: wlan0 is my card, yours might be different.

```
[iwd]# station wlan0 scan
```

List available networks:

```
[iwd]# station wlan0 get-networks
```

Connect to your network:

```
[iwd]# station wlan0 connect YourWiFiSSID
```

After this you will be prompted to enter your password. Enter password and exit iwctl by entering:

```
[iwd]# exit
```

Check that you are online by entering:

```
~# ping google.com
```

## Partitioning

### Create Partitions
List all connected drives:

```
~# lsblk
```

Use the gdisk utility to partition the drive:

>Note: This step is very important. You must target the right drive, or you risk erasing any data or operating system in another device. nvme0n1 is my drive, yours might be different.

```
~# gdisk /dev/nvme0n1
```

For this installation we will have four partitions: boot, swap, root, and home. The root and home partitions are typically together, but some prefer to separate them if you use multiple distributions or make clean installs often. Check the [arch wiki](https://wiki.archlinux.org/title/partitioning#/home) for more information.

#### Boot Partition

Create a new partition. This will be the boot partition:

```
Command (? for help): n
```

Select partition number. Press enter for default:

```
Partition number (default 1):
```

Select first sector. Press enter for default:

```
First sector (default = 2048):
```

Select last sector. This will be the size of the partition:

```
Last sector (default 240254912): +512M
```

Select partition type. The code for EFI is ef00: 

```
Hex code or GUID: ef00
```

#### Swap Partition

```
Command (? for help): n
```

Select partition number. Press enter for default:

```
Partition number (default 2): 
```

Select first sector. Press enter for default:

```
First sector (34-1953525134, default = 1050624):
```

Select last sector. This will be the size of the partition:

```
Last sector (default 240254912): +16G
```

Select partition type. The code for Linux Swap is 8200: 

```
Hex code or GUID: 8200
```

#### Root Partition

```
Command (? for help): n
```

Select partition number. Press enter for default:

```
Partition number (default 3): 
```

Select first sector. Press enter for default:

```
First sector (34-1953525134, default = 34605056):
```

Select last sector. This will be the size of the partition:

```
Last sector (default 240254912): +64G
```

Select partition type. The code for Linux Filesystem is 8300 (default): 

```
Hex code or GUID: 8300
```

#### Home Partition

```
Command (? for help): n
```

Select partition number. Press enter for default:

```
Partition number (default 4): 
```

Select first sector. Press enter for default:

```
First sector (34-1953525134, default = 168822784):
```

Select last sector. This will be the size of the partition. Press enter to asign it the remaining disk space:

```
Last sector (168822784-1953525134, default = 1953523711): 
```

Select partition type. The code for Linux Filesystem is 8300 (default): 

```
Hex code or GUID: 8300
```

#### Save changes and exit:

```
Command (? for help): w
```

Check that all partitions are correct. Your drive should have two partitions labeled sda1 for boot and sda2 for root:

```
~# lsblk   
```

### Format Partitions

Format the boot partition:

```
~# mkfs.fat -F32 /dev/nvme0n1p1
```

Format the swap partition:

```
~# mkswap /dev/nvme0n1p2
```

Format the root partition:

```
~# mkfs.ext4 /dev/nvme0n1p3
```

Format the home partition:

```
~# mkfs.ext4 /dev/nvme0n1p4
```

### Mount Partitions
Mount the root partition:

```
~# mount /dev/nvme0n1p3 /mnt
```

Create a directory for the boot loader:

```
~# mkdir -p /mnt/boot/efi
```
Create a directory for home:

```
~# mkdir -p /mnt/home
```

Mount the boot partition: 

```
~# mount /dev/nvme0n1p1 /mnt/boot/efi
```

Mount the home partition: 

```
~# mount /dev/nvme0n1p4 /mnt/home
```

Check that partitions are mounted correctly:

```
~# lsblk
```

## Base System
### Install Base Packages

>Note: In this step we will also install a text editor. I chose nano to keep things simple. You may install whichever you are most comfortable with.

```
~# pacstrap /mnt base linux linux-firmware nano
```

If you run into an issue with the keyring run the following commands:

```
~# pacman-key --init
~# pacman-key --populate archlinux
```

Generate file system table:

```
~# genfstab -U /mnt >> /mnt/etc/fstab
```

### Enter Chroot
Before we continue, change the apparent root directory to the root partition we created:

```
~# arch-chroot /mnt
```

### Configure Time Zones 
Select your region:

>Note: My region is America, New York. Yours might be different.

```
~# ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
```

Synchronize hardware clock and system clock:

```
~# hwclock --systohc
```

Select locale:

>Note: With this command (and any time we use nano) you will edit a configuration file. Look for your locale and uncomment it by removing the # in front of it. Once done, save and exit.

```
~# nano /etc/locale.gen
...
#en_SG ISO-8859-1
en_US.UTF-8 UTF-8
#en_US ISO-8859-1
...
```

Generate locale:

```
~# locale-gen
```

Set system language:

>Note: The file will be empty. Enter LANG= followed by your locale. Save and exit.

```
~# nano /etc/locale.conf

LANG=en_US.UTF-8
```

### Configure Host Info
Set host name:

>Note: The file will be empty. Enter the name you want for your machine, mine is "machine". Save and exit.

```
~# nano /etc/hostname

machine
```

Set network host:  

>Note: The file will have two commented lines. Ignore those and enter the following under them. Replace machine with your host name. Save and exit.

```
~# nano /etc/hosts

...
127.0.0.1  localhost 

::1        localhost

127.0.1.1  machine.localdomain machine
```

## Bootloader
Install the GRUB package and some extra tools for UEFI:

```
~# pacman -S grub efibootmgr mtools dosfstools
```

Install GRUB:

> Note: If you run into an issue with the EFI installation, make sure when you booted from the USB you selected UEFI boot, not legacy.

```
~# grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --recheck
```

Generate GRUB configuration file:

```
~# grub-mkconfig -o /boot/grub/grub.cfg
```

## Finishing Up

Install some extra packages:

```
~# pacman -S linux-headers base-devel xdg-utils xdg-user-dirs networkmanager git stow
```

### Users and Passwords

Set root password:

>Note: You will be prompted to type a password. Type it and press enter.

```
~# passwd
```

Create your user:

```
~# useradd -mG wheel akarez
```

Set your user password:

```
~# passwd akarez
```

Give user root privileges:

>Note: Uncomment the line shown below. This will give any user in the wheel group permisison to execute any command. Save and exit.

```
~# EDITOR=nano visudo

...
%wheel ALL=(ALL) ALL
...
```

### Network Manager

Enable network daemon:

```
~# systemctl enable NetworkManager
```

```
~# systemctl start NetworkManager
```

### Exit Installation

Exit chroot:

```
~# exit
```

Unmount all drives:

```
~# umount -a
```

Reboot:

```
~# reboot
```

## Install Desktop Environment

From here on you can install the desktop environment. Below are the steps for the system I currently use. See more info [here](https://github.com/akarez/dorfiles).

### Essential Packages

Install the YAY AUR helper. This will allow you to install packages from the Arch Linux community repo:

```
~$ git clone https://aur.archlinux.org/yay.git 

~$ cd yay

~$ makepkg -si
```

Install display, audio, and wireless packages:

```
~$ sudo pacman -S xf86-video-intel xf86-video-amdgpu xorg xorg-xinit xorg-xev xorg-xbacklight arandr alsa-utils pulseaudio bluez bluez-utils

~$ yay -S pulseaudio-ctl pavucontrol 
```

Install window management and system utilities:

```
~$ sudo pacman -S bspwm sxhkd alacritty ranger feh rofi picom firefox zip unzip brightnessctl wget upower cron neovim zathura zathura-pdf-mupdf ttf-iosevka-nerd  nmap xsel  powertop htop lshw xed

~$ yay -S polybar ueberzug coreshot xidlehook neofetch apple-fonts ttf-ms-win10-auto gtk-theme-numix-solarized redshift
```

### Optional Packages

The following are packages that I use but are not necessary for a working system. I put them here anyway so I can copy paste the command when needed:

#### Virtualization

```
~$ sudo pacman -S qemu libvirt iptables-nft dnsmasq virt-manager virt-viewer vde2 bridge-utils openbsd-netcat
```

#### Programming Utilities

```
~$ sudo pacman -S nodejs npm rust rust-src arduino-cli 

```
#### PCB Design Utilities
```
~$ sudo pacman -S kicad

```
## Load Configuration

The configuration files are managed with [GNU Stow](https://www.gnu.org/software/stow/). To setup, first clone [this](https://github.com/akarez/dorfiles) repo. Then, navigate into the cloned directory and enter the following command:

```
~$ stow alacritty bash bspwm neofetch nvim polybar ranger rofi sxhkd wallpapers x zathura
```
