# Raspberry Pi 2 Mobel B v1.1



current version ref: https://raspberrytips.com/raspberry-pi-os-versions/

lite version ref: https://downloads.raspberrypi.org/raspios_lite_armhf/images/

# for arch linux [click here](arhlinux.raspi2.md)



sudo localectl set-locale LANG=en_US.utf8</br>
Getting “Cannot set LC_CTYPE to default locale: No such file or directory” Used nano to set Locale on the file directly (using nano) `sudo nano /etc/default/locale`

        C-type, unicode may not work 
        LANG=C
        LC_CTYPE="C"
        LC_NUMERIC="C"
        LC_TIME="C"
        LC_COLLATE="C"
        LC_MONETARY="C"
        LC_MESSAGES="C"
        LC_ALL=
        
        or (english - USA)
        LANG=en_US.UTF-8
        LANGUAGE=en_US:en
        LC_CTYPE="en_US.UTF-8"
        LC_NUMERIC="en_US.UTF-8"
        LC_TIME="en_US.UTF-8"
        LC_COLLATE="en_US.UTF-8"
        LC_MONETARY="en_US.UTF-8"
        LC_MESSAGES="en_US.UTF-8"
        LC_PAPER="en_US.UTF-8"
        LC_NAME="en_US.UTF-8"
        LC_ADDRESS="en_US.UTF-8"
        LC_TELEPHONE="en_US.UTF-8"
        LC_MEASUREMENT="en_US.UTF-8"
        LC_IDENTIFICATION="en_US.UTF-8"
        LC_ALL=

ref: https://askubuntu.com/questions/515330/how-do-i-go-about-removing-all-the-language-packs-i-dont-need#:~:text=As%20regards%20actually%20installed%20languages,my%20comment%20on%20your%20question.

    

Did same for `/etc/default/keyboard` and checked the layout line: `XKBLAYOUT="us"`

     sudo reboot





# wifi-conection for [debian/raspbian](wifi-debian.md)




## backpup rootfs "/" partition
__`cd /`__ </br> `sudo tar --create --gzip --file=${HOSTNAME}.rootfs.$(date +"%Y-%m-%d-%H-%M-%S").tar.gz --atime-preserve --preserve-permissions --acls --selinux --xattrs --exclude=/boot/* --exclude=/dev --exclude=/proc --exclude=/tmp --exclude=/mnt --exclude=/home/pi --exclude=/home/stpi/*.gz --exclude=/home/*/.gvfs --exclude=/home/*/.cache --exclude=/home/*/.local/share/Trash --exclude=/sys --exclude=/media --exclude=/run --exclude=/var/log --exclude=/var/swap --exclude=/var/log --exclude=/var/cache/apt --exclude=/var/lib/apt/lists/ --exclude=/usr/src/linux-headers* --exclude=/usr/share/locale --one-file-system --record-size=715776K / &` </br> </br>
__&__ this is to backup in background </br>
cd / is needed for backup / </br>
 </br>
ref: https://help.ubuntu.com/community/BackupYourSystem/TAR </br>

## backpup /boot partition

`sudo tar --create --gzip --file=${HOSTNAME}.boot.$(date +"%Y-%m-%d-%H-%M-%S").tar.gz --preserve-permissions --acls --selinux --xattrs --exclude=/boot/.Trash-*  --exclude=/boot/"System Volume Information" --exclude=/boot/"$RECYCLE.BIN" --one-file-system --record-size=715776K /boot &`

## restore tar backup on linux pc
  ### restore partition od SD-Card incase
  Set the environment variable 'mydev' to the device file name of the USB stick that Alpine Linux is to be installed to:

    Warning: BE SURE TO GET THIS RIGHT OR ELSE YOU COULD OVERWRITE THE WRONG DISK!

    mydev=/dev/sdU

Make sure that the target drive's existing partitions, if any, are not mounted:

    umount -q $mydev?

Copy and paste the following as a single command to wipe the target drive, create an MBR partition table, and create a single FAT32 partition (you can ignore any "Partition #1 contains a vfat signature." warning message):

    fdisk -w always $mydev <<EOF
      o
      n
      p
      1
      2048
      +256M
      t
      0c
      a
      w
    EOF

Format the new FAT32 partition with a FAT32 filesystem:</br>
`sudo mkfs.fat -n boot_fs -F 32 /dev/sda1 $mydev1`</br>
`sudo mkfs.ext4 -L root_fs $mydev2` 

incase using f2fs for ext4 use </br>
`sudo mkfs.f2fs /dev/sdb2` but make sure your tarbarr is injected with `sudo apt-get install f2fs-tools` and make change in file `boot_fs/cmdline.txt` and replace the keyword as `rootfstype=f2fs` also look into `/etc/fstab`

ref: http://whitehorseplanet.org/gate/topics/documentation/public/howto_ext4_to_f2fs_root_partition_raspi.html

you can label latter also</br>
`fatlabel /dev/device boot_fs`</br>
`e2label /dev/device root_fs`</br>

#### But the best cmd for restoration from tar.gz on some linux PC
`mount /dev/sda2 /mnt ; mount /dev/sda1 /mnt/boot`

`cd /mnt/`

`sudo tar --extract --gzip --preserve-permissions --acls --selinux --xattrs --atime-preserve --numeric-owner --same-owner --file=root_fs.tar.gz`<br>

`#cd /mnt/boot (dont use this stay in /mnt)`

`sudo tar --extract --gzip --preserve-permissions --acls --selinux --xattrs --atime-preserve --numeric-owner --same-owner --file=boot_fs.tar.gz`<br>

#### ⚠️ **Warning:** Don't use `sudo` for fs-restore login into root use `su` 

#### ⚠️ `sync`, always use `sync` slow device like sdcard write, before umount 



    A brief explanation:
       -f, --file
       -x, --extract, --get
       -s, --preserve-order, --same-order
       -v, --verbose
       -z, --gzip, --gunzip, --ungzip  or  -J, --xz
       -p, --preserve-permissions, --same-permissions
              extract information about file permissions (default for
              superuser)
       --one-top-level[=DIR]
              Extract all files into DIR, or, if used without argument,
              into a subdirectory named by the base name of the archive
              (minus standard compression suffixes recognizable by
              --auto-compress).
       --same-owner
              Try extracting files with the same ownership as exists in
              the archive (default for superuser).
       --numeric-owner
              Always use numbers for user/group names.
       -C, --directory=DIR
              Change to DIR before performing any operations.  This
              option is order-sensitive, i.e. it affects all options
              that follow.

## tunning after restore 
sudo lsblk -o PARTUUID /dev/sda</br>
or</br>
sudo fdisk -l /dev/loop1 | grep identifier</br>
Disk identifier: 0x`4c4e106f`</br>

1. udate partuuid and "init=/usr/lib/raspberrypi-sys-mods/firstboot" in __`/boot/cmdline.txt`__
2. update partuuuid in __`/etc/fstab`__
3. `sudo fsck /dev/sda1`
4. `sudo fsck /dev/sda2`
5. 1st(just after fresh dd sdcard) cmdline.txt to fix all prtition related problem, which looks like this: </br>
   `console=serial0,115200 console=tty1 root=PARTUUID=4c4e106f-02 rootfstype=ext4 fsck.repair=yes rootwait quiet init=/usr/lib/raspberrypi-sys-mods/firstboot`




# Rosewill RNX-N150NUB Wireless N150 Nano USB Wi-Fi Adapter 150 Mbps USB2.0
[802.11n NIC](wifi-debian.md)
VID=0BDA PID=FFEF

## driver by 
ref: https://github.com/lwfinger/rtl8188eu
