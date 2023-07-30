# Raspberry Pi Setup Guide
This is a guide to setup a Raspberry Pi as a webserver running Arch Linux ARM.

Some important links:
 - http://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2
 - https://wiki.archlinux.org/index.php/Raspberry_Pi

## Hardware
Raspberry Pi 2 B: much more powerfull than previous models, has 1GB RAM and a 900MHz CPU, very capable as a home server.

Make sure to get a high quality SD card (at least 8GB). SD cards easily corrupt and are often slow. Get a class 10 card from a well known brand.

A good power supply is also necessary. Phone chargers can work, but it should supply at least 1A, more if you want to power USB devices.

A heat sink is not necessary, but can be nice if you plan to overclock.

# 1 Arch Linux Installation
See http://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2 for more details.

## 1.1 Download file system image
```wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz```

## 1.2 Install to SD card
Insert the SD card into the reader on your computer and make sure that it is not mounted automatically. You need to make two partitions: ```/boot``` (must be FAT32) and ```/``` (ext4 recomended). You can do this with any partiton manager you like (gparted, parted, fdisk).
```
# The following must be done as root
su
# After filesystems are created mount the SD card
mkdir root
mount /dev/xxxx1 root
mkdir boot
mount /dev/xxxx2 boot
# Extract archive
bsdtar -xpf ArchLinuxARM-rpi-2-latest.tar.gz -C root
sync
# Move boot files
mv root/boot/* boot
# Unmount
umount boot root
```

## 1.3 Boot and login
Insert the SD card and turn on the power!

SSH is enabled by default, so connect ethernet and login from another computer. First you need to find the IP address of the Raspberry Pi (look in router settings or use Fing (an android app)). Default user is alarm with password alarm.
```
ssh alarm@<ip-address>
```


## 1.4 USB/WiFi Connection
`wifi-menu -o`

`netctl enable wlu1u2-Home-WiFi`

`pacman -Syy`





# 2 Initial Settings

## 2.1 Secure accounts
```
# Set a secure password
passwd
```
You can either do the same for the root account, or just lock it (you really should never login as root). The default password for root is root.
```
su
usermod -L root
```

## 2.2 Install base packages
```
pacman -Syu base base-devel vim archlinuxarm-keyring
```

## 2.3 Configure pacman
```
pacman-key --init
pacman-key --populate archlinuxarm
```
Edit ```/etc/pacman.conf```:
```
...
Color
TotalDownload
ILoveCandy
...
SigLevel = Optional TrustAll
```

## 2.4 Configure sudo
Run visudo. Add:
```
Defaults insults
Defaults editor=/usr/bin/rvim, !env_editor
%wheel ALL = (ALL) ALL
```

## 2.5 Time
The Raspberry Pi does not have a realtime clock, so the time should be updated over the internet on each boot.
```
sudo timedatectl set-timezone Asia/Kolkata
sudo timedatectl set-ntp true
```

## 2.6 Hostname
```
sudo hostnamectl set-hostname name
```

## 2.7 Swap
```
sudo fallocate -l 1024 /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
# Avoid using swap
sudo echo "vm.swappiness=5" > /etc/sysctl.d/fs.conf
```
Add the following line to ```/etc/fstab```
```
swap    /swapfile    defaults   0  0
```

## 2.7 Tweaks to minimize disk writes and optimize performance
In ```/etc/fstab``` add ```noatime,discard``` to the mount options for ```/```.

Write journal to volatile storage: in ```/etc/systemd/journald.conf``` add ```Storage=volatile```. Also add your user to the systemd-journald group to be able to view all of the journal ```gpasswd -a alarm systemd-journald```.

Do not store core dumps: in ```/etc/systemd/coredump.conf``` add ```Storage=none```.

Install iotop to view disk I/O activity by process.

Since the Raspberry Pi is going to be running as a headless server, we can allocate more memory towards the CPU, instead of the GPU. In ```/boot/config.txt``` add ```gpu_mem=16```. This is also where you can overclock the Pi if you want. For more information about all of the options see http://elinux.org/RPiconfig.

