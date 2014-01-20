# Raspberry-Pi-Setup-Guide

A really opionionated guide how to setup a RPi with Arch Linux including ntp, wi-fi, ssh, ruby, zsh etc.

Take a look into the wiki for more interesing stuff like finding out your Raspberry Pi version.


## 1. Setup the SD card
### 1.1. Download the image from the website
### 1.2. Write the image via dd on the SDCard
    sudo dd bs=1M if=ArchLinuxARM-2014.01-rpi.img of=/dev/sdb

### 1.3. Resize of the Partition via gparted
If your SD card is larger than 2 GB, you will want to resize the partitions to make use of your entire card.

The image creates two partitions, which will be available as /dev/mmcblk0p1 and /dev/mmcblk0p2 when you boot up your Raspberry Pi. The first partition (100 MB) is mounted to /boot, and the second (1.7 GB by default) is mounted to the root directory /. Leave the first partition as it is, and expand the second to as large as you desire within the space available on your card.

I recommend to use gparted.

### 1.4. Put the SD Card into your pi, power it on and login with root/root
You should have connected a keyboard via usb and some kind of screen via hdmi.


## 2. Basic system setup
### 2.1. German keyboard layout and timezone
Of course just if you want to have german keyboard layout. You may skip that step or use another layout.

```bash
loadkeys de
echo LANG=de_DE.UTF-8 > /etc/locale.conf
echo KEYMAP=de-latin1-nodeadkeys > /etc/vconsole.conf
rm /etc/localtime
ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime
nano /etc/locale.conf
```

* Uncomment `en_US.UTF-8`

```bash
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
locale-gen en_US.UTF-8
```


### 2.2. Setup swapfile
```bash
fallocate -l 1024M /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

* Then add the following line to `/etc/fstab`:

```bash
/swapfile none swap defaults 0 0
```


### 2.3. Set hardware clock to utc and set timezone
```bash
timedatectl set-local-rtc 0

nano /etc/timezone
```

* Set to "Europe/Berlin"
 

## 3. Update system and enable NTP
### 3.1. Tweak pacman
```bash
vim /etc/pacman.conf
```

* Uncomment the line "Color"


### 3.2. System update
```bash
pacman -Sy pacman
pacman-key --init
pacman -S archlinux-keyring
pacman-key --populate archlinux
pacman -Syu --ignore filesystem
pacman -S filesystem --force
reboot
```


### 3.3. NTP
```bash
pacman -S ntp
systemctl enable ntpd.service
systemctl start ntpd.service
```


## 4. Advanced setup
### 4.1. Set a secure root passwd
```bash
passwd
```


### 4.2. Set hostname
```bash
nano /etc/hostname
```


### 4.3. sudo & user
```bash
pacman -S sudo vim
visudo
```

* Search for following line and uncomment it:

```bash
%wheel ALL=(ALL) ALL
```

* Then install adduser

```bash
pacman -S adduser
```

* Add a new user. The additional groups are `rvm` and `wheel`

```bash
adduser
```

### 4.4. Additional software
* Log out and log in with our new created user

```bash
sudo pacman -S nfs-utils htop openssh autofs alsa-utils alsa-firmware alsa-lib alsa-plugins git zsh zsh-grml-config base-devel diffutils
```


* Install yaourt:

```bash
wget https://aur.archlinux.org/packages/pa/package-query/package-query.tar.gz
tar -xvzf package-query.tar.gz
cd package-query
makepkg -si
cd ..
wget https://aur.archlinux.org/packages/ya/yaourt/yaourt.tar.gz
tar -xvzf yaourt.tar.gz
cd yaourt
makepkg -si
```


### 4.5 vcgencmd and other vc tools
```bash
vim /etc/profile
```

Change the line saying `PATH=`:

```bash
# Set our default path
PATH="/usr/local/sbin:/usr/local/bin:/usr/bin:/opt/vc/sbin:/opt/vc/bin"
export PATH
```


## 5. Setup SSH server
```bash
systemctl enable sshd
systemctl start sshd
```

* Try to connect from another machine, if it works, you can disconnect the screen and keyboard and work via ssh



## 6. Sound
Set the output device

```bash
amixer cset numid=3 1
```



## 7. Raspberry pi overclocking
You may want to overclock the Pi. This settings equals to the "high" overclocking known from `rasp-config`.
```
nano /boot/config.txt

arm_freq=950
core_freq=450
sdram_freq=450
over_voltage=6
```

* Then reboot
 


## 8. WLAN
```bash
wifi-menu -o
netctl start yourWlanSSID
netctl enable yourWlanSSID
```


## 9. Ruby
```bash
\curl -sSL https://get.rvm.io | bash -s stable
useradd -G rvm benny
yaourt -S ruby
rvm reload
rvm install ruby
rvm list
rvm alias create default ruby-2.1.0 # Or something else depending on what rvm list says
gem install bundler
```


## 10. ZSH and dotfiles
If you want to use ZSH and my dotfiles ...
```bash
sudo usermod -s /usr/bin/zsh

cd ~
git clone https://github.com/phortx/dotfiles.git .dotfiles
cd .dotfiles
rake install
```

* Relog
    

## 11. Tweaks
TRIM and noatime
```
/dev/root  /  ext4  noatime,discard  0  0
```



## Sources
* My experiences
* [Arch Linux Wiki: Raspberry Pi](https://wiki.archlinux.org/index.php/Raspberry_Pi)
* [elinux.org ArchLinux Install Guide](http://elinux.org/ArchLinux_Install_Guide)
* [Arch Linux Wiki: SSH Server](https://wiki.archlinux.de/title/SSH)
* [Michael Paquier: Raspberry PI with basic setup](http://michael.otacoo.com/manuals/raspberry-pi/)
