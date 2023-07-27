# rbox2b.local
Raspberry Pi 2 Mobel B

# backpup rootfs "/" partition

`sudo tar --create --preserve-permissions --gzip --file=${HOSTNAME}.rootfs.$(date +"%Y-%m-%d-%H-%M-%S").tar.gz --acls --selinux --xattrs --exclude=/boot/* --exclude=/dev --exclude=/proc --exclude=/tmp --exclude=/mnt --exclude=/home/pi --exclude=/home/stpi/*.gz --exclude=/home/*/.gvfs --exclude=/home/*/.cache --exclude=/home/*/.local/share/Trash --exclude=/sys --exclude=/media --exclude=/run --exclude=/var/log --exclude=/var/swap --exclude=/var/log --exclude=/var/cache/apt  --exclude=/var/lib/apt/lists/ --exclude=/usr/src/linux-headers* --exclude=/usr/share/locale --one-file-system --record-size=715776K / &`

# backpup /boot partition

`sudo tar --create --preserve-permissions --gzip --file=${HOSTNAME}.boot.$(date +"%Y-%m-%d-%H-%M-%S").tar.gz --acls --selinux --xattrs --exclude=/boot/.Trash-*  --exclude=/boot/"System Volume Information" --exclude=/boot/"$RECYCLE.BIN" --one-file-system --record-size=715776K /boot &`

# restore tar backup on linux pc

`sudo tar -xvpzf /path/to/backup.tar.gz -C /media/whatever --numeric-owner`

A brief explanation:

x - Tells tar to extract the file designated by the f option immediately after. In this case, the archive is /home/test/backup.tar.gz

-C <directory> - This option tells tar to change to a specific directory before extracting. In this example, we are restoring to the root (/) directory.

--numeric-owner - This option tells tar to restore the numeric owners of the files in the archive, rather than matching to any user names in the environment you are restoring from. This is due to that the user id:s in the system you want to restore don't necessarily match the system you use to restore (eg a live CD).

# tunning after restore 
1. udate partuuid in `/boot/cmdline.txt`
2. update partuuuid in `/etc/fstab`
3. `sudo fsck /dev/sda1`
4. `sudo fsck /dev/sda2`
