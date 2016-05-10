# arch-install

# Index
1. [Get Arch](#get-arch)
2. [Format and Mount partitions](#format)
  * [GPT](#gpt)
  * [MBR](#mbr)

# [Get Arch](#get-arch)
Download the latest ISO image from https://www.archlinux.org/download/
Write to a USB stick
```
$ df
...
/dev/sdc1  16G  7.5G  8.2G  48% /run/media/wchmb/347E-D4CB
# dd if=/home/wchmb/Downloads/arch.iso of=/dev/sdc
```

### Set keyboard layout
```
# loadkeys es
```

### Connect to the Internet
```
# iw dev 				      # list wireless interfaces
# wifi-menu -o wlp3s0	# connect to specific interface
```

##### Use system clock
```
# timedatectl set-ntp true
# timedatectl set-timezone Europe/Madrid
# timedatectl set-local-rtc 0	# use UTC
# timedatectl status
```

##### [Format and  mount partitions](#format)
```
# cfdisk			# partition the disks
  [gpt]
	sda1	2M		Bios boot		<- required on BIOS/GPT configuration
	sda2	200M	Linux fileystem	<- /boot
	sda3	20G		Linux fileystem	<- /
	sda4	80G		Linux fileystem	<- /home
	sda5	2G		Linux swap		<- Avoid with SSD
	[Write] and [Quit]
# mkfs -t ext4 /dev/sda2
# mkfs -t ext4 /dev/sda3
# mkfs -t ext4 /dev/sda4
-# mkswap /dev/sda4
-# swapon /dev/sda4	# activate swap
		
# mount /dev/sda3 /mnt
# mkdir /mnt/boot /mnt/home
# mount /dev/sda2 /mnt/boot
# mount /dev/sda4 /mnt/home
```

### Installation:

##### Install base system and with pacstrap (installation script in Arch)
```
# pacstrap /mnt base			# base system
```	


##### Configure the system
```
# genfstab -Up /mnt >> /mnt/etc/fstab	# generate fstab with UUIDs
# arch-chroot /mnt			# left the .iso and go to the new system
```

##### Create a new initial RAM disk
```
# mkinitcpio -p linux
```	


##### Install a boot loader in BIOS/GPT
! Attention: if you install 'os-prober' grub-mkconfig may fail
```
# pacstrap /mnt grub			# grub (and os-prober for search other OS)
# grub-install /dev/sda			# install the bootloader to drive
# grub-mkconfig –o /boot/grub/grub.cfg	# generate grub.cfg
```

##### Finish
```
# passwd				# set root password
# exit					# exit the chroot environment and come back to .iso
# umount -R /mnt			# unmount partitions
# systemctl reboot
```

	

### Post-installation:

##### Configure the network and locales
```
# echo computer_name > /etc/hostname				# set hostname
# ln -s /usr/share/zoneinfo/Europe/Madrid /etc/localtime	# set time zone
```

System language
Uncomment the needed locales in /etc/locale.gen
```
# cat <<EOF > /etc/locale.conf					# set system language
LANG="en_US.UTF-8"
EOF
```	

console(TTY) keyboard
```
# cat <<EOF > /etc/vconsole.conf	# make keyboard layout persistent
KEYMAP=es
FONT=lat9w-16 # located in /usr/share/kbd/consolefonts/
EOF
# locale-gen
```

Xorg keyboard
```	
# localectl --no-convert set-x11-keymap es pc105	
  # It will save the configuration in '/etc/X11/xorg.conf.d/00-keyboard.conf', 
	# this file should not be manually edited, because localectl will overwrite the changes on next start

# localectl status					# check all changes
	System Locale: LANG=en_US.UTF-8
	VC Keymap: es
	X11 Layout: es
	X11 Model: pc105
```

##### Create a new user
```
# useradd -m -g users -G wheel -s /bin/bash ablasco
# passwd ablasco	# (passwd -d ablasco) to login without passwd
```

##### Internet connection
```
$ ip link			# or 'iw dev' for wireless devices
# systemctl start dhcpcd@eth0 	# eth0 or enp0s3 or another wired interface...
# systemctl enable dhcpcd@eth0
```

##### Fastest mirror
```
# cd /etc/pacman.d
# cp mirrorlist mirrorlist.backup
# rankmirrors -n 6 mirrorlist.backup > mirrorlist
# pacman -Syy	#update package list
```

##### Graphical user interface (https://www.archlinux.org/groups/x86_64/xorg/)
```	
(*)$ lspci | grep -e VGA -e 3D		# If you do not know what graphics card you have
# pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils	# default Xorg environment
# pacman -S mesa mesa-demos	# 3D graphics support
# pacman -S xf86-video-intel		# intel video drivers
# pacman -S xf86-video-nouveau	# nvidia video drivers
# pacman -S xf86-input-synaptics	# touchpad support
		
# pacman -S xorg-twm xorg-xclock xterm	
```
#! After testing X you can remove 'xorg-twm', 'xorg-xclock' and 'xterm'

```
# startx
# exit
```

##### Autostart X at login
```
$ cp /etc/X11/xinit/xinitrc ~/.xinitrc
$ cat ~/.xinitrc
  ...	
  # some applications that should be run in the background without a window manager

	# twm &
	# xclock -geometry 50x50-1+1 &
	# xterm -geometry 80x50+494+51 &
	# xterm -geometry 80x20+494-0 &
	# exec xterm -geometry 80x66+0+0 -name login
	xscreensaver &
	xsetroot -cursor_name left_ptr &
	exec cinnamon-session
	exec firefox
		
	$ cat <<EOF >>  ~/.bash_profile
	# autostart X at login
	 -z $DISPLAY && $XDG_VTNR -eq 1  && exec startx
	EOF
```


##### (*)Alternative-> Use display manager
```
# pacman -S gdm				# gdm, lxdm ...
# systemctl enable gdm
# systemctl start gdm
```

Resources:
[1] https://wiki.archlinux.org/index.php/beginners'_guide  
[2] https://wiki.archlinux.org/index.php/General_recommendations  
[3] VIRTUALBOX emulation: http://wideaperture.net/blog/?p=3851  
[4] https://github.com/helmuthdu/aui  
