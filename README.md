# arch-install

# Index
1. [Preparation](#preparation)  
2. [Installation](#installation)  
3. [Set-up](#setup)  
4. [Resources](#resources)



### [Preparation](#preparation)

##### Get Arch
Download the latest ISO image from https://www.archlinux.org/download/  
Write out to a USB stick
```
$ df
  ...
  /dev/sdc1  16G  7.5G  8.2G  48% /run/media/wchmb/347E-D4CB
# dd if=/home/wchmb/Downloads/arch.iso of=/dev/sdc
```  
##### Set keyboard layout
```
# loadkeys es
```

##### Connect to the Internet via WiFi
dhcpcd daemon is enabled on boot for wired devices (eno1,...)
```
# iw dev                    # list wireless interfaces
# wifi-menu -o wlp3s0       # connect to specific interface
# ping -c 3 www.google.com  # test it
```

##### Use system clock [(time)](https://wiki.archlinux.org/index.php/time)
```
# timedatectl set-ntp true
# timedatectl set-timezone Europe/Madrid
# timedatectl set-local-rtc 0  # use UTC
# timedatectl status
```

##### Part the Disk [(GPT vs MBR)](https://wiki.archlinux.org/index.php/partitioning#Partition_table)
###### [MBR] (https://wiki.archlinux.org/index.php/GRUB#Master_Boot_Record_.28MBR.29_specific_instructions)
```
# cfdisk  # partition the disks
  # Usual dance: New -> Partition Size -> Primary or Extended -> ...
  ...
  [gpt]
    sda1        2M          Bios boot	     <- required on BIOS/GPT configuration
    sda2	200M        Linux fileystem  <- /boot
    sda3        20G         Linux fileystem  <- /
    sda4        80G         Linux fileystem  <- /home
    sda5        2G          Linux swap	     <- Avoid with SSD!
  [Write] and [Quit]
```
If you have >4GB of RAM, then you possibly don't need to create a Swap partition.  
In other case: [(swappiness)](https://wiki.archlinux.org/index.php/Swap#Swappiness) [(Boost performance)](https://rudd-o.com/linux-and-free-software/tales-from-responsivenessland-why-linux-feels-slow-and-how-to-fix-that)
```
# cat <<EOF > /etc/sysctl.d/99-sysctl.conf
vm.swappiness=10          # kernel's preference of swap space (default 60)
vm.vfs_cache_pressure=50  # kernel's tendency to reclaim the memory which is 
                          # used for caching, versus pagecache and swap (default 100)
EOF
```
###### [GPT](https://wiki.archlinux.org/index.php/GRUB#GUID_Partition_Table_.28GPT.29_specific_instructions)
TODO

##### Make the FileSystem
```	
# mkfs -t ext4 /dev/sda2  # to create a new file system
# mkfs -t ext4 /dev/sda3
# mkfs -t ext4 /dev/sda4
``` 

*Just in case you have Swap
```
# mkswap /dev/sda4
# swapon /dev/sda4  # activate swap
```

##### Mount the partitions 
```
# mount /dev/sda3 /mnt
# mkdir /mnt/boot /mnt/home
# mount /dev/sda2 /mnt/boot
# mount /dev/sda4 /mnt/home
```

### Installation
The goal of the bootstrapping procedure is to setup an environment from which the scripts from arch-install-scripts (such as **pacstrap** and **arch-chroot**) can be run.

##### Install base system
```
# pacstrap /mnt base  # base system
```	

##### Configure the system [(mkinitcpio)](https://wiki.archlinux.org/index.php/Fstab)
```
# genfstab -Up /mnt >> /mnt/etc/fstab  # generate fstab with UUIDs
# arch-chroot /mnt                     # left the .iso and go to the new system
```

##### Create a new initial RAM disk
```
# mkinitcpio -p linux
```	

##### Install a boot loader in BIOS/GPT
_*Attention:* if you install ```os-prober```, ```grub-mkconfig``` may fail_
```
# pacstrap /mnt grub                    # grub (and os-prober for search other OS)
# grub-install /dev/sda                 # install the bootloader to drive
# grub-mkconfig –o /boot/grub/grub.cfg  # generate grub.cfg
```

##### Finish
```
# passwd          # set root password
# exit            # exit the chroot environment and come back to .iso
# umount -R /mnt  # unmount partitions
# systemctl reboot
```

	

### [Set-up](#setup)

##### Configure the network and locales
```
# echo computer_name > /etc/hostname                      # set hostname
# ln -s /usr/share/zoneinfo/Europe/Madrid /etc/localtime  # set time zone
```

##### System language
Uncomment the needed locales in /etc/locale.gen and then:
```
# cat <<EOF > /etc/locale.conf  # set system language
LANG="en_US.UTF-8"
EOF
```	

##### Console(TTY) keyboard
```
# cat <<EOF > /etc/vconsole.conf  # make keyboard layout persistent
KEYMAP=es
FONT=lat9w-16 # located in /usr/share/kbd/consolefonts/
EOF
# locale-gen
```

##### Xorg keyboard [(keyboard in Xorg)](https://wiki.archlinux.org/index.php/Keyboard_configuration_in_Xorg#Using_X_configuration_files)
```	
# localectl --no-convert set-x11-keymap es pc105	
  # It will save the configuration in '/etc/X11/xorg.conf.d/00-keyboard.conf', 
  # this file should not be manually edited, because localectl will overwrite the changes on next start.

# localectl status  # check all changes
  System Locale: LANG=en_US.UTF-8
  VC Keymap: es
  X11 Layout: es
  X11 Model: pc105
```

##### Create a new user [(users and groups)](https://wiki.archlinux.org/index.php/users_and_groups#Example_adding_a_user)
```
# useradd -m -g users -G wheel -s /bin/bash wchmb
# passwd wchmb	
# passwd -d wchmb  # to login without passwd
```

##### Internet connection
```
$ ip link                      # or 'iw dev' for wireless devices
# systemctl start dhcpcd@eth0  # eth0 or enp0s3 or another wired interface...
# systemctl enable dhcpcd@eth0
```

##### Fastest mirror [(mirrors)](https://wiki.archlinux.org/index.php/mirrors)
```
# cd /etc/pacman.d
# cp mirrorlist mirrorlist.backup
# rankmirrors -n 6 mirrorlist.backup > mirrorlist
# pacman -Syy  #update package list
```

##### Graphical user interface [(xorg)](https://www.archlinux.org/groups/x86_64/xorg/)
```	
$ lspci | grep -e VGA -e 3D       # If you do not know what graphics card you have
# pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils  # default Xorg environment
# pacman -S mesa mesa-demos       # 3D graphics support
# pacman -S xf86-video-intel      # intel video drivers
# pacman -S xf86-video-nouveau    # nvidia video drivers
# pacman -S xf86-input-synaptics  # touchpad support
```

##### Test X Windows System
Check that everything went well...and take a look back at the good old days
```
# pacman -S xorg-twm xorg-xclock xterm    # some X programs
# startx                                  # run Xorg and play
# exit                                    # quit
# pacman -Rns xorg-twm xorg-xclock xterm  # you can remove them after testing
```

##### Autostart X at login [(xinitrc)](https://wiki.archlinux.org/index.php/xinitrc)
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
  [[ -z $DISPLAY && $XDG_VTNR -eq 1 ]]  && exec startx
  EOF
```

##### Alternative: Use display manager
Really needed? You kids...
```
# pacman -S gdm  # gdm, lxdm ...
# systemctl enable gdm
# systemctl start gdm
```


[Resources](#resources)  
[1] https://wiki.archlinux.org/index.php/beginners'_guide  
[2] https://wiki.archlinux.org/index.php/General_recommendations  
[3] https://wiki.archlinux.org/index.php/installation_guide  
[4] https://github.com/helmuthdu/aui  
