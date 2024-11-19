# CYB3353 Arch Installation
## Before Installation
### Virtual Machine
- *VMware Workstation Pro 17.6.1*
- 1 processor, 1 core
- 4096 MB ram
- 20.0 GB storage
- UEFI firmware type
- side channel mitigations disabled
### Arch Iso
- *archlinux-2024.10.01-x86_64.iso*
	- downloaded from [this US-based mirror](https://mirror.clarkson.edu/archlinux/iso/2024.10.01/)

---
## First Boot
### Necessary Settings
1. power on the VM and boot into the live environment
2. verify boot mode with **cat /sys/firmware/efi/fw_platform_size**
3. verify internet connection with **ip link** and **ping -c 10 8.8.8.8**
4. <ins>DON'T WORRY ABOUT TIMEZONES</ins>
	- setting timezones with **systemd-firstboot** and **timedatectl** requires a reboot to enable changes—messes everything up if done before the full installation
	- I made this mistake and needed to reset the VM from the ISO

### Disk Partitions
1. partition the disk with **fdisk /dev/sda**
	1. create a GPT
	2. create partition 1 with 500 MiB
		- label as an EFI system
	3. create partition 2 with 19.5 GiB
		- label as a Linux filesystem
2. format the partitions
	- format the EFI partition as FAT32 with **mkfs.fat -F 32 /dev/sda1**
	- create ext4 filesystem on the Linux filesystem partition with **mkfs.ext4 /dev/sda2**
3. mount the filesystems
	1. mount the Linux filesystem partition with **mount /dev/sda2 /mnt**
	2. mount the EFI partition with **mount --mkdir /dev/sda1 /mnt/boot**
4. snapshot the VM

---
## Installation
1. select mirrors by editing */etc/pacman.d/mirrorlist*
	- add "us." before "niranjan.co"
	- move the US mirrors to the top of the list
2. install essential packages with **pacstrap -K /mnt base linux nano man-db man-pages texinfo networkmanager**
3. snapshot the VM

---
## Configuration
### General Configuration
1. generate an fstab file with **genfstab -U /mnt >> /mnt/etc/fstab**
	- double-check the fstab file with **cat /mnt/etc/fstab**
2. change root into the new system with **arch-chroot /mnt**
3. adjust the time settings
	1. set the timezone to CST with **ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime**
	2. edit the system clock to use local time instead of UTC with **hwclock -l --systohc** to create */etc/adjtime*
4. localize the system
	1. edit */etc/localegen* to uncomment "en_US.UTF-8 UTF-8"
	2. generate locales with **locale-gen**
	3. create */etc/locale.conf* with text "LANG=en_US.UTF-8"
5. edit hostname by creating */etc/hostname* with text "arch"
6. set the root password with **passwd**
7. snapshot the VM

### GRUB Installation
1. install *GRUB* and its EFI boot manager with **pacman -S grub efibootmgr**
2. install the GRUB EFI application and its modules with **grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --boot-directory=/boot**
3. create the main GRUB configuration file with **grub-mkconfig -o /boot/grub/grub.cfg**
4. snapshot the VM

---
## Reboot
1. exit the changed root environment with **exit**
2. unmount the partitions with **umount -R /mnt**
3. reboot the VM with **reboot**
4. snapshot the VM

---

## System Administration
### Users and Groups
1. create users with home folders and allow future *sudo* privileges with **useradd -mG wheel** **_username_**
	- run the command three times, replacing **_username_** with **alex**, **codi**, and **justin**, respectively
2. create passwords with **passwd** **_username_**
	1. run the command three times, replacing **_username_** with **alex**, **codi**, and **justin**, respectively
	2. set the passwords for *codi* and *justin* to "GraceHopper1906"
3. require *codi* and *justin* to change their password upon first login with **passwd -e** **_username_**
	- run the command two times, replacing **_username_** with **codi** and **justin** respectively
4. install *sudo* with **pacman -S sudo**
5. enable *wheel* group to allow *sudo* privileges
	1. securely edit *sudo* file with **visudo**
	2. uncomment *wheel* group

### Shell Customization
1. install *ssh* with **pacman -S openssh**
2. install *zsh* with **pacman -S zsh**
	- edit */etc/passwd* to change default user shells to *zsh*
3. customize command prompt
	1. create *.zshrc* file in *alex*'s home directory
	2. customize command prompt by adding `PS1='%F{green}%B%n%b%f%B@%b%F{magenta}%B%m%b%f %F{cyan}%B%~%b%f %B%#%b '` to *.zshrc*
	3. add aliases to have common commands output color to *.zshrc*
		- `alias diff='diff --color=auto'`
		- `alias grep='grep --color=auto'`
		- `alias ip='ip -color=auto'`
		- `alias ls='ls --color=auto'`
	4. customize man pager to use color
		- `export MANPAGER='less -R --usecolor -Dd+r -Du+b'`
		- `export MANROFFOPT='-P -c'`

### Desktop Environment
1. install the *xorg* group to use the Xorg display server with **sudo pacman -S xorg**
2. install *xorg-xinit* to manually start a Xorg session with **sudo pacman -S xorg-xinit**
3. install *gnome* to use the GNOME on Xorg desktop environment
4. force the system to boot into GNOME
	- enable the GNOME display manager with **sudo systemctl enable gdm.service** and select the *GNOME on Xorg* session type
	- edit *.xinitrc* to use GNOME instead of the Xorg default
		1. `export XDG_SESSION_TYPE=x11`
		2. `export GDK_BACKEND=x11`
		3. `exec gnome-session`
	- edit *.zprofile* to start GNOME
		- `if [[ -z $DISPLAY && $(tty) == /dev/tty1 ]]; then`
		- `  XDG_SESSION_TYPE=x11 GDK_BACKEND=x11 exec startx`
		- `fi`
