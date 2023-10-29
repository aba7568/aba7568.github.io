# aba7568.github.io
# Arch Install

## Setting up the VM

- download the ISO
- create VM in VMWorkstation 
- Go through each step selecting ISO >> Select linux OS & Version >> Disk Size: 20 >> Change Ram to 2G

Realized I forgot to edit the .vmx file to allow it to boot to UEFI mode so I went back to add this code to line 2 
```
firmware="efi"
```
## Pre-installation
Boot and select UEFI Mode

#### Verify The Boot Mode
``````bash
cat /sys/firmware/efi/fw_platform_size
``````

#### Comfirming Internet Connection 
``````bash
ip link
ping archlinux.org
``````

#### Updating System Clock 
``````bash
timedatectl
``````
#### Partitioning The Disk 
``````bash
fdisk -l 
fdisk /dev/sda
``````
#### Creating The Partitions
``````bash
# Add New Partition (EFI)
n # Creates 
p # Primary 
1 # Default partition number 
2048 # Deafault First Sector 
+500 M # Default Last Sector 
#change type to EFI
t >> l >> ef

# Add New Partition (Main)
n # Creates 
p # Primary 
2 # Default partition number 
1026048 # Deafault First Sector 
# Default Last Sector 

p # Varify both paritions were created
w # Save and exit 
``````
#### Formating The Partitions 
``````bash
mkfs.fat -F32 /dev/sda1       
mkfs.ext4 /dev/sda2 
``````

#### Mount The File Systems
``````bash
mount /dev/sda2 /mnt

# Mount the EFI system partition

mount --mkdir /dev/sda1 /mnt/boot
``````
## Installation 

#### Viewing Mirror List

At first i tried just using bash/etc/pacman.d/mirrorlist gave me an error message then I tried it with vi gave me another error message tried it again with vim and it worked
``````bash
vim /etc/pacman.d/mirrorlist
``````

#### Install (pacstrap)
``````bash
pacstrap -K /mnt base linux linux-firmware
``````

## Configuring The System
#### Fstab
Generate an fstab file:
`````` bash
genfstab -U /mnt >> /mnt/etc/fstab
``````

#### Chroot
Change root into the new system:
`````` bash
arch-chroot /mnt
``````

#### Time Zone 
``````bash
timedatectl set-timezone America/Chicago
timedatectl status #make sure it is the right time 
``````

#### Localization
``````bash
locale-gen # Generates the locales
``````
Create the locale.conf file, and set the LANG variable:
``````bash
 localectl set-locale LANG=en_US.UTF-8
``````

#### Network Configuration 
Create Host name use *echo*
``````bash
echo blessed > /etc/hostname
``````

#### Password
``````bash
passwd
``````

#### Boot Loader
Install GRUB
``````bash
grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
``````
## Adding Users

``````bash
# Add User
useradd -m codi
passwd GraceHopper1906
# Sudo Permisons 
EDITOR=nano visudo
usermod -aG wheel,audio,video,storage team 
``````
## Installing a DE
I Choose GNOME 
- install the x enviroment and then GNOME
``````bash
pacman -S xorg networkmanager
pacman -S gnome
``````
- enabling the netowrk manager 
``````bash
systemctl enable gdm.service
systemctl enable NetworkManager.service
``````

## Finishing Up
- exit chroot
- unmount root parition 
- reboot
``````bash
exit
umount -l /mnt # Had an issue here first tried umount /mnt did not work until i used -l
reboot
``````

## Installing a Differnt Shell
``````bash
echo $SHELL # to see what shell is being used 
sudo pacman -S zsh # install
cat /etc/shells # to list all the shells
