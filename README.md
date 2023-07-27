# rbox2b.local
Raspberry Pi 2 Mobel B
___

## backpup rootfs "/" partition
__`cd /`__ </br> `sudo tar --create --preserve-permissions --gzip --file=${HOSTNAME}.rootfs.$(date +"%Y-%m-%d-%H-%M-%S").tar.gz --acls --selinux --xattrs --exclude=/boot/* --exclude=/dev --exclude=/proc --exclude=/tmp --exclude=/mnt --exclude=/home/pi --exclude=/home/stpi/*.gz --exclude=/home/*/.gvfs --exclude=/home/*/.cache --exclude=/home/*/.local/share/Trash --exclude=/sys --exclude=/media --exclude=/run --exclude=/var/log --exclude=/var/swap --exclude=/var/log --exclude=/var/cache/apt  --exclude=/var/lib/apt/lists/ --exclude=/usr/src/linux-headers* --exclude=/usr/share/locale --one-file-system --record-size=715776K / &` </br> </br>
__&__ this is to backup in background </br>
cd / is needed for backup / </br>
 </br>
ref: https://help.ubuntu.com/community/BackupYourSystem/TAR </br>

## backpup /boot partition

`sudo tar --create --preserve-permissions --gzip --file=${HOSTNAME}.boot.$(date +"%Y-%m-%d-%H-%M-%S").tar.gz --acls --selinux --xattrs --exclude=/boot/.Trash-*  --exclude=/boot/"System Volume Information" --exclude=/boot/"$RECYCLE.BIN" --one-file-system --record-size=715776K /boot &`

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
      -0
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

you can label latter also</br>
`fatlabel /dev/device boot_fs`</br>
`e2label /dev/device root_fs`</br>

`sudo tar -xvpzf /path/to/backup.tar.gz -C /media/whatever --numeric-owner`

A brief explanation:</br>
__x__ - Tells tar to extract the file designated by the f option immediately after. In this case, the archive is /home/test/backup.tar.gz</br>
__-C <directory>__ - This option tells tar to change to a specific directory before extracting. In this example, we are restoring to the root (/) directory.</br>
__--numeric-owner__ - This option tells tar to restore the numeric owners of the files in the archive, rather than matching to any user names in the environment you are restoring from. This is due to that the user id:s in the system you want to restore don't necessarily match the system you use to restore (eg a live CD).

## tunning after restore 
sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL
1. udate partuuid in __`/boot/cmdline.txt`__
2. update partuuuid in __`/etc/fstab`__
3. `sudo fsck /dev/sda1`
4. `sudo fsck /dev/sda2`
