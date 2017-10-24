# Raspberry Pi Setup Guide

A really opionionated guide how to setup every version of a Raspberry Pi with Arch Linux including WiringPi, NTP, Wi-Fi, SSH,
Ruby, ZSH and more.

Take a look into the wiki for more interesing stuff like finding out your Raspberry Pi version.


## Some words regarding the hardware

I recommend you to get a speed class 10 SD Card with more than 4 GB capacity for optimal performance.

Additionally you should buy a small heatsink. [Something like that](http://www.amazon.com/s/ref=nb_sb_noss_1?url=search-alias%3Daps&field-keywords=raspberry%20pi%20heatsink&sprefix=raspberry+pi+he%2Caps&rh=i%3Aaps%2Ck%3Araspberry%20pi%20heatsink) and attach it to the CPU of the Raspberry Pi.


## What you'll need

- A linux machine with a working SD card slot
- `bsdtar` or `tar`, `fdisk`


## 1. Setup the SD card
### 1.1. Format the SD card with fdisk

Replace `/dev/sdX` with the SD Card device. Make sure that the device is the SD card and not your harddrive, otherwise
you'll destroy your linux installation! You can see which device you'll have to use by running `sudo fdisk -l` after putting the
SD card into the slot.

1. Start `fdisk` via `sudo fdisk /dev/sdX`.
2. At the fdisk prompt, delete existing partitions: Type `o`. This will clear out any partitions on the drive. Then type
   `p` to list partitions. There should be no partitions left.
3. Type `n`, then `p` for primary, `1` for the first partition on the drive, press `ENTER` to accept the default first
   sector, then type `+100M` for the last sector.
4. Type `t`, then `c` to set the first partition to type `W95 FAT32 (LBA)`.
5. Type `n`, then `p` for primary, `2` for the second partition on the drive, and then press `ENTER` twice to accept
   the default first and last sector.
6. Write the partition table and exit by typing `w`.
7. Now create a FAT filesystem: `mkfs.vfat /dev/sdX1` and mount the new boot partition via
   `mkdir boot && sudo mount /dev/sdX1 boot`
8. Also create the filesystem for the root partition and mount it. You can choose between the classic `ext4` filesystem, or the flash filesystem friendly `f2fs`.
   a.) (RECOMMENDED!) You need to install `f2fs-tools`, then format root partition to f2fs:
   `sudo pacman -S f2fs-rools`    
   `sudo mkfs.f2fs /dev/sdX2`   
   `mkdir root && sudo mount -t f2fs /dev/sdX2 root`
   b.) (Easy way.) Format to ext4 filesystem: 
   `sudo mkfs.ext4 /dev/sdX2`   
   `mkdir root && sudo mount /dev/sdX2 root`

### 1.2. Download the image from the website

There are 3 major versions of Raspberry Pi now. You may find the downloads on
[www.archlinuxarm.org](http://www.archlinuxarm.org) for the latest version of Arch Linux for Raspberry Pi 1, 2 and 3.


**For Raspberry Pi 2 or 3**

```bash
wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz
sudo tar tar xvzf ArchLinuxARM-rpi-2-latest.tar.gz -C root
sync
```


**For Raspberry Pi 1**

```bash
wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-latest.tar.gz
sudo tar xvzf ArchLinuxARM-rpi-latest.tar.gz -C root
sync
```


### 1.3. Write the files onto the SD Card

```bash
sudo mv root/boot/* boot/
sudo umount boot root
```

### 1.4. If you choose to use `f2fs` for root filesystem, you have to some extra changes. Skip this step in case of using `ext4` rootfs.
   
   Edit `fstab` and add the next lines:
   `nano root/etc/fstab`
   
   `
   /dev/mmcblk0p2    /        f2fs    defaults,noatime,discard    0  0
   tmpfs             /tmp     tmpfs   nodev,nosuid,size=2G        0  0
   `
   Next change the boot command line to identify `f2fs` filesystem type:
   `nano /mnt/boot/cmdline.txt`

   After root partition, add `rootfstype=f2fs`:
   `root=/dev/mmcblk0p2 rw rootwait rootfstype=f2fs ...`
   Later you have to install `f2fs-tools` to your installation. Until that, you have to substitute the fs checker command:
    `cp root/bin/true root/sbin/fsck.f2fs`

### 1.5. Some other recommended changes in `config.txt` before booting up your Pi:

   `sudo nano boot/config.txt`
   
   Add more memory to gpu. (For use of Kodi, minimum 128MB recommended)
   `gpu_mem=192`
   
   Purchased licence codes to enable hardware codecs:
   `
   decode_MPG2=0x12345678
   decode_WVC1=0x12345678
   `
   
   HDMI output modes:
   `hdmi_drive=1 #Normal DVI mode (no sound)`
   or
   `hdmi_drive=2 #Normal HDMI mode (sound will be sent if supported and enabled)`
   
   HDMI force hotplug (HDMI output mode will be used, even if no HDMI monitor is detected.)
   `hdmi_force_hotplug=1`
   
   Disable HDMI overscan
   `disable_overscan=1`
   
   Enabling and control the onboard audio, I2C, I2S and SPI interfaces without using dedicated overlays:
   `dtparam=audio=on,i2c_arm=on,i2s=on,spi=on`
   
   If you have a class 10 SD card, you can add below in config.txt:
   `dtparam=sd_overclock=100`

### 1.5. Unmount filesystems of the SD Card:

    `umount /mnt/boot /mnt/root`

## 2. Basic system setup.

Put the SD Card into your pi, power it on and login with `alarm`/`alarm`

You can have connected a keyboard via USB and some kind of screen via HDMI or you can connect to the Pi via SSH after
it's booted.

First of all get root:

```bash
su
```

The password is `root`.


### 2.1. Change keyboard layout and timezone

   Setup the appropriate locale. First edit `/etc/locale.gen` and uncomment the lines that correspond to your 
   language selection, generate the needed locales, set your keymap and finally set the language with the system.
   `[root@alarmpi ~]$ nano /etc/locale.gen`
   
   `
   ...
   hu_HU ISO-8859-2
   hu_HU.UTF-8
   ...
   `
   
   `[root@alarmpi ~]$ locale-gen`
   `Generating locales...
   hu_US.UTF-8
   hu_US.ISO-8859-2
   Generation complete.`
   
   `[root@alarmpi ~]$ localectl set-keymap hu`
   `[root@alarmpi ~]$ localectl set-locale LANG="hu_HU.UTF-8"`
   
   Set hardware clock to UTC and configure the timezone. For me this is `Europe/Budapest` as I am located in Hungary. 
   Then enable ntpd for internet time syncing capabilities.
   `[root@alarmpi ~]$ timedatectl set-local-rtc 0`
   `[root@alarmpi ~]$ timedatectl set-timezone Europe/Budapest`
   `[root@alarmpi ~]$ systemctl enable ntpd`

### 2.3. Set hostname and configure network
   Set the hostname for use with networks, I will be using the hostname `rpi.local`
   `root@alarmpi ~]$ hostnamectl set-hostname rpi.local`
   
   For a wired network card first copy a template and then edit it. In example, we will use dhcp settings.
   `
   [root@alarmpi ~]$ cd /etc/netctl
   [root@alarmpi ~]$ cp examples/ethernet-dhcp eth0
   [root@alarmpi ~]$ nano eth0
   `
   
   `
   Description='A basic dhcp ethernet connection'
   Interface=eth0
   Connection=ethernet
   IP=dhcp
   ExecUpPost='/usr/bin/ntpd -gq || true'
   `

   `
   [root@alarmpi ~]$ chmod 640 eth0
   [root@alarmpi ~]$ netctl enable eth0
   `
   
   If you wish to use wireless instead it is pretty much the same process except we use a different template. 
   The only difference is to make sure you secure the file if it contains your wireless password.
   `
   [root@alarmpi ~]$ cd /etc/netctl
   [root@alarmpi ~]$ cp examples/wireless-wpa wlan0
   [root@alarmpi ~]$ nano wlan0
   `
   
   `
   Interface=wlan0
   Connection=wireless
   Security=wpa
   IP=dhcp
   ESSID='MyNetwork'
   Key='WirelessKey'
   ExecUpPost='/usr/bin/ntpd -gq || true'
   `
   
   `
   [root@alarmpi ~]$ chmod 640 wlan0
   [root@alarmpi ~]$ netctl enable wlan0
   `
   
   

### 2.4. Setup swapfile if using `ext4` as rootfs

```bash
fallocate -l 1024M /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo 'vm.swappiness=1' > /etc/sysctl.d/99-sysctl.conf
```

* Then add the following line to `/etc/fstab`:

```bash
/swapfile none swap defaults 0 0
```

## 3. Update system
### 3.1. Tweak pacman

```bash
sed -i 's/#Color/Color/' /etc/pacman.conf # Add color to pacman
```


### 3.2. System update

```bash
pacman -Sy pacman
pacman-key --init
pacman -S archlinux-keyring
pacman-key --populate archlinux
pacman -Syu --ignore filesystem
pacman -S filesystem f2fs-tools --force
systemctl reboot
```

After the Pi is booted again, connect via SSH (if you don't have attached a keyboard and screen) and login with `alarm`/`alarm` and get root again via `su`.


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


### 4.2. sudo & user

```bash
pacman -S sudo vim
visudo
```

* Search for following line and uncomment it:

```bash
%wheel ALL=(ALL) ALL
```

* Add a new user (replace `yourUserName` with your username!)

```bash
useradd -d /home/yourUserName -m -G wheel,rvm -s /bin/bash yourUserName
```

* Set a password for your new user:

```bash
passwd yourUserName
```

* Log out and log in with our newly created user

* After that, delete the old `alarm` user:

```bash
sudo userdel alarm
```


### 4.3. Additional software

```bash
sudo pacman -S --needed nfs-utils htop openssh autofs alsa-utils alsa-firmware alsa-lib alsa-plugins git zsh wget base-devel diffutils libnewt dialog wpa_supplicant wireless_tools iw crda lshw
```


* Install yaourt:

```bash
wget https://aur.archlinux.org/cgit/aur.git/snapshot/package-query.tar.gz
tar -xvzf package-query.tar.gz
cd package-query
makepkg -si
cd ..
wget https://aur.archlinux.org/cgit/aur.git/snapshot/yaourt.tar.gz
tar -xvzf yaourt.tar.gz
cd yaourt
makepkg -si

cd ../
rm -rf package-query/ package-query.tar.gz yaourt/ yaourt.tar.gz
```


### 4.4 vcgencmd and other vc tools

```bash
sudo vim /etc/profile
```

Change the line saying `PATH=`:

```bash
# Set our default path
PATH="/usr/local/sbin:/usr/local/bin:/usr/bin:/opt/vc/sbin:/opt/vc/bin"
export PATH
```

And reload it:

```bash
source /etc/profile
```


### 4.5 WiringPi

```bash
sudo git clone git://git.drogon.net/wiringPi /opt/wiringpi
cd /opt/wiringpi
sudo ./build

gpio -v
gpio readall
```

* The last both command should give an `ok` or something similar. If not, something may be broken.



## 5. Sound

This is just for Raspberry Pi 1.

Set the output device

```bash
sudo amixer cset numid=3 1
```



## 6. Raspberry Pi overclocking

You may want to overclock the Pi. And you won't even lose the guarantee for your pi, if you use the "offical"
overclocking presets. The simplest way to overclock the pi is `rasp-config` tool which ships with the offical allowed
overclocking presets.

```
wget https://raw.github.com/chattama/raspi-config-archlinux/archlinux/raspi-config
```

Get to the overclocking menu and choose the overclocking preset you want. I recommend the "high" preset. After changing
the overclocking preset, reboot your raspberry pi.



## 7. Wi-Fi

```bash
sudo wifi-menu -o
netctl start yourWifiSSID
netctl enable yourWifiSSID
```


## 9. Ruby

```bash
\curl -sSL https://get.rvm.io | sudo bash -s stable
sudo usermod -aG rvm yourUser
rvm reload
rvm install ruby
rvm list
rvm alias create default ruby-2.3.0 # Or something else depending on what rvm list says
gem install bundler rake
```


## 10. ZSH and dotfiles

If you want to use ZSH
```bash
sudo usermod -s /usr/bin/zsh
```

Additionally you may want to clone and setup your personal dotfiles.

* Logout and login back again or just reboot the pi


## 11. Tweaks
### 11.1 Increase SD card lifetime

Change in your fstab:

```bash
sudo vim /etc/fstab
```

```
/dev/root  /  ext4  defaults,nodiratime,noatime,discard  0  0
```



## Sources

* My experiences
* [Arch Linux Wiki: Raspberry Pi](https://wiki.archlinux.org/index.php/Raspberry_Pi)
* [elinux.org ArchLinux Install Guide](http://elinux.org/ArchLinux_Install_Guide)
* [Arch Linux Wiki: SSH Server](https://wiki.archlinux.de/title/SSH)
* [Michael Paquier: Raspberry PI with basic setup](http://michael.otacoo.com/manuals/raspberry-pi/)
