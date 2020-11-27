# Select keyboard
```
loadkeys uk OR us
```
# Activate network
```
dhcpcd
```
# Verify network
```
ping -c 3 www.archlinux.org
```
# Set and sync time
```
timedatectl set-ntp true
timedatectl set-tiezone Europe/Bucharest
hwclock --systohc --utc
timedatectl status
```
# Sync pacman
```
pacman -Syyy
```
# Install reflector
```
pacman -S reflector
```
# Generate mirrorlist by speed with reflector
```
reflector --verbose --latest 100 --sort rate --save /etc/pacman.d/mirrorlist
```
# Verify disks
```
fdisk -l OR lsblk
```
# Partitioning disk - mkpart part-type fs-type start end
```
parted -a optimal /dev/sda
(parted) mklabel gpt
(parted) mkpart ESP fat32 1MiB 513MiB
(parted) set 1 boot on
(parted) name 1 efi
(parted) mkpart primary ext4 513MiB 100%
(parted) name 3 lvm
(parted) print
(parted) quit
parted /dev/sda2 set 2 lvm on
```
# Verify partitions
```
parted /dev/sda prin
```
## Setup LVM

# Create Physical Volume
```
pvcreate /dev/sda2
```
# Create Volume Group
```
vgcreate arch-lvm /dev/sda2
```
# Create Logical Volume
```
lvcreate -n arch-root -L 50G arch-lvm
lvcreate -n arch-home -l 100%FREE arch-lvm
```
# Create File Systems
```
mkfs.fat -F32 /dev/sda1
mkfs.ext4 -L root /dev/arch-lvm/arch-root
mkfs.ext4 -L home /dev/arch-lvm/arch-home
```
# Create mount points and mount the file systems
```
mount /dev/arch-lvm/arch-root /mnt
mkdir /mnt/{boot,home}
mkdir /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
mount /dev/arch-lvm/arch-home /mnt/home
```
# Install base system
```
pacstrap -i /mnt base base-devel linux linux-firmware linux-headers lvm2 intel-ucode amd-ucode git curl wget rsync rclone reflector vim bash-completion
```
# Do system configuration
```
genfstab -U -p /mnt >> /mnt/etc/fstab
```
# Chroot
```
arch-chroot /mnt /bin/bash
```
# Edit /etc/mkinitcpio.conf to include needed module
```
vim /etc/mkinitcpio.conf
```
> Look for line starting with HOOKS="..." and add 'lvm2' AFTER block and BEFORE filesystems

# Regenerate initrd image
```
mkinitcpio -p linux
```
# Locale
```
vim /etc/locale.gen
```
# Generate locale
```
locale-gen
```
# Add locale.conf
```
echo LANG=en_US.UTF-8 > /etc/locale.conf
```
# Time
```
ln -sf /usr/share/zoneinfo/Europe/Bucharest /etc/localtime
hwclock --systohc --utc
```
# Hostname
```
echo hostname > /etc/hostname
```
# Hosts
```
vim /etc/hosts {
127.0.0.1	localhost
::1		    localhost
127.0.1.1	archlinux
}
```
# Root password
```
passwd [root password]
```
# Rest of packages
```
pacman -S grub sudo dhcpcd netctl wpa_supplicant iw dialog broadcom-wl wireless_tools networkmanager network-manager-applet bash-completion efibootmgr dosfstools os-prober mtools
```
# Install grub
```
grub-install /dev/sda
```
# or
```
grub-install --target=x86_64-efi  --bootloader-id=grub_uefi --recheck
```
# or
```
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
```
> For a 32-bit system replace x86_64 with i386.
# Config grub
```
grub-mkconfig -o /boot/grub/grub.cfg
```
# Enable NetworkManager
```
systemctl enable NetworkManager
```
# Exit from chroot
```
exit [chroot]
```
# Umount
``` 
umount -R /mnt/boot
umount -R /mnt/home
umount -R /mnt
```
# Reboot
```
reboot
```
# Add new user
```
useradd -m -g users -G wheel -s /bin/bash gntlmn
```
# or
```
useradd -mG wheel -s /bin/bash gntlmn
```
# User password
```
passwd gntlmn
```
# Edit sudoers
```
vim /etc/sudoers {
[remove '#' in front of 'wheel']
}
```
# Exit root and login as regular user
```
exit [root] and login as user [gntlmn]
```
# Install reflector
```
sudo pacman -Syyyu reflector
```
# Run reflector and generate new mirrorlist
```
sudo reflector --verbose --latest 100 --sort rate --save /etc/pacman.d/mirrorlist
```
# Install rest needed packages
```
sudo pacman -Syu pulseaudio pulseaudio-alsa alsa-utils [if no sound, execute 'alsactl init']
```
```
sudo pacman -Syu xorg-server xorg xf86-video-ati xf86-video-amdgpu
```
```
sudo pacman -Syu i3-gaps dmenu i3stats xfce4-terminal firefox polkit-gnome
```
```
sudo pacman -Syu lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
```
# Enable LightDM
```
sudo systemctl enable lightdm
```
