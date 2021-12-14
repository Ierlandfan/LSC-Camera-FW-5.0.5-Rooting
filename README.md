LSC 5.0.5 FW repository

Binwalk camera-5.0.5.bin (Most Important output) 

2949120 0x2D0000 CramFS filesystem, little endian, size: 4595712, version 2, sorted_dirs, CRC 0x5DDF61C8, edition 1, 1119 blocks, 3 files

Strip cramfs from binary:
sudo dd bs=1 skip=2949120 count=4595712 if=../camera-5.0.5 of=camera-5.0.5.cramfs
(Skip is file offset (left row in Binwalk output of cramfs ) count is file size, as seen in Binwalk output of cramfs
(I used firmware-mod-kit from Github for the following part)
sudo ./uncramfs_all.sh ../../../cramfs-dd-extracted/camera-5.0.5.cramfs

cd cramfs-root/
we find two files, app.tar.xz and initrun.sh

We need to change initrun.sh in the cramfs-root to reflect the 5.0.5 modified init

#!/bin/sh

BASE_DIR=/opt/pps
if [ -e /tmp/meari.runpath ]; then
        BASE_DIR=`cat /tmp/meari.runpath`
fi

if [ ! -e ${BASE_DIR} ]; then
        exit
fi

tar xf ${BASE_DIR}/app.tar.xz -C /
umount ${BASE_DIR}

/app/init/rcS &

cat /proc/mounts > /tmp/hack
while true; do
 sleep 10
 if [ -e /mnt/mmc01/custom.sh ]; then
  cp /mnt/mmc01/custom.sh /tmp/custom.sh
  chmod +x /tmp/custom.sh
  /tmp/custom.sh
 fi
done

Step 2: Repack:
Putting it all back together:

mkfs.cramfs -b 4096 -e 1 -N little -n ppsapp mycramfs-root/ my.cramfs
ppsapp is the name of the filesystem, ( [Volume name: ppsapp]
(mycramfs-root is the extracted filesystem we modified, my.cramfs is the new name for the new cramfs.
cp camera-5.0.5.bin mycamera-modified-5.0.5.bin
(The cp command is to copy the original firmware file ( camera-5.0.5.bin ) as a new file (mycamera-5.0.5.bin) that gets the new cramfs (my.cramfs)
sudo dd conv=notrunc if=my.cramfs of=mycamera-modified-5.0.5.bin bs=1 seek=2949120
(The dd command will apply the new cramfs (my.cramfs) to the Firmware file (mycamera-modified-5.0.5.bin) we newly created and seek number is the offset we found in binwalk)
