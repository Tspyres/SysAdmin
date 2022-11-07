# Arch Install

This was mainly accomplished with the help of the Arch Wiki:
<https://wiki.archlinux.org/title/Installation_guide#>

## Download ISO

First, I went through the wiki and found a mirror ISO package for the US.

Next, I created a VM in VMWare Workstation with the following specs:
- one file HDD
- 20 GB HDD
- 2GB ram
- Mount Arch ISO to CD drive

I followed the overview video and decided to boot in EFI. So I went into my `.vmx` file and inserted the following into line 2:

```
firmware="efi"
```

## Booting into VM and Installation

- Boot into VM under UEFI option
- Used default keyboard layout

### Verify boot mode

```bash
ls /sys/firmware/efi/efivars
```
### Check Internet Connection

```bash 
ip link
ip addr show
ping google.com
```

### Update Clock

Going to be honest here and say that I couldn't figure out why `set` command wasn't working and changed it manually after I had finished setting up my VM. So the time is accurate but was not able to update it in the terminal.

### Partition Disks

```bash
fdisk -l
fdisk /dev/sda
```

### Created Partitions

```bash
#Create EFI/Boot Partition
n #Create New
p #Primary
1 #Partition Number
2048 #Default Start Sector
+500MB #Last Sector

p #Verify

#Create Main Partition
n #Create New
p #Primary
2 #Partition Number
978944 #Default Start Sector After SDA1
#End of Disk

p #Verify

w #Save and Exit
```

### Format Main Partitions

```bash
mkfs.ext4 /dev/sda2
mkfs.fat -F 32 /dev/sda1
lsblk #Verify Format
```

### Mount Partitions

```bash
mount /dev/sda2 /mnt
mkdir -p /mnt/boot/efi #Make /boot/efi Directory
mount /dev/sda1 /mnt/boot/efi
lsblk #Verify Mounts
```
### Installing Packages

```bash
pacstrap /mnt base linux linux-firmware sof-firmware nano base-devel grub efibootmgr networkmanager
#Installing all these individual packages with pacstrap
```

### Fstab

```bash
genfstab /mnt #Checks Files in /mnt
genfstab /mnt > /mnt/etc/fstab #Sends /mnt file to /mnt/etc/fstab
```

### Chroot
```bash
arch-chroot /mnt
```

### Time Zone

```bash
ln -sf /usr/share/zoneinfo Americas/Tulsa etc/localtime

date #Verify Time
```

### Localization

```bash
nano locale-gen
#I then scrolled down to `en_US.UTF-8 UTF-8` and uncommented it. Then CTRL-O, ENTER, CTRL-X to save and quit.

locale-gen #Generates our locale
nano /etc/locale.conf
LANG=en_US.UTF-8
#Once again do CTRL-O, ENTER, CTRL-X to save and quit.
```
I did not change my keyboard so it stayed as the default.

### Network Configuration

```bash
nano /etc/hostname
myHost
#Once again do CTRL-O, ENTER, CTRL-X to save and quit.
```

### Root Password

```bash
passwd
```
### Enabling Network Manager

```bash
systemctl enable NetworkManager
```

### Boot Loader

```bash
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```
### Reboot

```bash
exit #Exiting root user
umount -a
reboot
```

### Installing a DE

```bash
sudo pacman -S plasma sddm
#I just went with the default options that it asked during installation
sudo pacman -S konsole kate firefox
#These are my terminal, text editor and web browser options
sudo systemctl enable --now sddm
```

## Creating My Personal User and Codi's

```bash
useradd -m -G wheel -s /bin/bash taylor
passwd taylor #Created a passwd

EDITOR=nano visudo
#I scrolled down to the wheel group and umcommented it to allow sudo permissions and made sure to Save & Exit

useradd -m -G wheel -s /bin/bash codi
passwd codi #Which is GraceHopper1906
```

## Installing a Different Shell

```bash
sudo pacman -S zsh zsh-completions
#Enter passwd
y #For yes
chsh -l #Lists shells on terminal to verify installation
```

## Install SSH and Go Into The Class Gateway

```bash
sudo pacman -Syu
sudo pacman -S openssh
sudo systemctl status sshd #Check to see if its inactive/active
sudo systemctl enable sshd
ssh sysadmin@10.10.1.130 #I know it was meant to be the class gateway but I couldn't find the password for it so I just did my personal gateway and it worked so should work with the class gateway
```

## Color Coding To The Terminal

```bash
nano .bashrc
export  PS1="\e[0;36m[\u@\h \W]\$ \e[m "
#This makes my prompt a cyan color
```

## Adding Aliases

```bash
nano .bashrc
#I just added them to the bottom
alias teleport='ssh sysadmin@10.10.1.130'
alias view='ls -a'
#Once again do CTRL-O, ENTER, CTRL-X to save and quit.
reboot #This allows newly added aliases to run
```

