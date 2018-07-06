Kernel parameter With serial console.
console=ttyO2,115200n8 mem=450M
With frame buffer console.
mem=450M

# How to bring up the panda board
Connect the micro-SD card in the card reader and connect the card reader to a linux PC.
Create two partitions

Prepare U-Boot
Flash the U-Boot from linaro to the SD card.
$ export URL=http://releases.linaro.org/13.06/ubuntu/panda/panda-raring_developer_20130623-376.img.gz
$ curl $URL | gunzip | sudo dd bs=64k of=/dev/sdd
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  189M  100  189M    0     0   351k      0  0:09:10  0:09:10 --:--:--  320k
112+32544 records in
112+32544 records out
1073741824 bytes (1.1 GB) copied, 550.892 s, 1.9 MB/s
$ sudo sync


# Prepare boot.scr
Create a file named boot.txt with the following contents (Basically containing the necessary bootargs. The boot.scr from linaro boots the uImage and root filesytem from the SD MMC card)
usb reset
setenv ipaddr 10.2.2.42
setenv serverip 10.2.22.1
setenv dtbpath "arm/panda/omap4-panda-a4.dtb"      <<---- NOTE, If you are using PandaBoard A4
setenv dtbpath "arm/panda/omap4-panda-es.dtb"      <<---- NOTE, If you are using PandaBoard ES (B series)
setenv dtbpath "arm/panda/omap4-panda.dtb"         <<---- NOTE, If you are using other PandaBoards
setenv dtbaddr 0x815f0000
setenv initrd_high "0xffffffff"
setenv fdt_high "0xffffffff"
setenv bootargs 'console=tty0 console=ttyO2,115200n8 mem=456M@0x80000000 mem=512M@0xA0000000 root=/dev/nfs rw nfsroot=10.2.22.1:/tftpboot/arm/panda ip=10.2.2.42 rootdelay=30'
setenv bootcmd 'tftp 0x80200000 arm/panda/uImage; tftp ${dtbaddr} ${dtbpath};bootm 0x80200000 - ${dtbaddr}'
boot

mkimage -A arm -O linux -T script -C none -a 0 -e 0 -n "Panda SD Boot" -d boot.txt boot

# Create a file boot.scr from boot.txt
$  sysopy boot.scr to the boot partition of the SD card.
$ sudo cp boot.scr /media/boot/
Attach the micro-SD card to the target board and reset the target.


========= How to bring up the panda board in 3.10 using rootfs on SD card =============
Connect the micro-SD card in the card reader and connect the card reader to a linux PC.
Creating Partitions on the SD card
Create 2 partitions on SD card:
Use fdisk to create partitions:
$ sudo fdisk /dev/sdb
Create a new partition, partition 1:
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-30679039, default 2048):
Using default value 2048
Last sector, +sectors or +sizeK,M,G (2048-30679039, default 30679039): +8M
Set the first partition as W95 FAT32 (LBA) using the t command and entering the Hex code c.
Command (m for help): t
Selected partition 1
Hex code (type L to list codes): c
Set the bootable flag on the first partition using the a command.
Command (m for help): a
Partition number (1-4): 1
Create a second primary partiion using the n command. This partition will be a linux partition for storing the root filesystem. It will fill the rest of the SD card.
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4, default 2): 2
First sector (18432-30679039, default 18432):
Using default value 18432
Last sector, +sectors or +sizeK,M,G (18432-30679039, default 30679039):
Using default value 30679039
Verify that the partition table is correct by using the p command. It should look similar to the following:
Command (m for help): p                                                         
                                                                                
Disk /dev/sdb: 15.7 GB, 15707668480 bytes
64 heads, 32 sectors/track, 14980 cylinders, total 30679040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x6eaae8f8

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1   *        2048       18431        8192    c  W95 FAT32 (LBA)
/dev/sdb2           18432    30679039    15330304   83  Linux
This step will destroy all data on the SD Card - Write the partition table to the card using the w command.
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: If you have created or modified any DOS 6.x
partitions, please see the fdisk manual page for additional
information.
Syncing disks. 
Format the first partition of the SD card with a FAT filesystem using the mkfs.vfat tool.
$ sudo /sbin/mkfs.vfat -n boot /dev/sdb1
Format the second partition using an ext4 filesystem using the mkfs.ext4 tool.
$ sudo /sbin/mkfs.ext4 -L rootfs /dev/sdb2 
Once you have a properly partitioned SD card, you can then populate it with the boot files. The partitions are usually automounted under /media, but if not, you can use the mount command to mount the partition to an arbitrary location.
    $ sudo mount /dev/sdb1 /media/boot
    $ sudo mount /dev/sdb2 /media/rootfs
Creating boot.scr for setting the bootargs
Create the following boot.txt file:
setenv dtbaddr 0x815f0000
setenv fdt_high "0xffffffff"
setenv initrd_high "0xffffffff"
setenv dtbfile "omap4-panda-a4.dtb"
setenv bootargs "console=tty0 console=ttyO2,115200n8 mem=456M@0x80000000 mem=512M@0xA0000000 root=/dev/mmcblk0p2 ip=10.2.2.42 rw rootwait"
setenv bootcmd "mmc rescan;fatload mmc 0:1 0x80200000 uImage;fatload mmc 0:1 ${dtbaddr} ${dtbfile};bootm 0x80200000 - ${dtbaddr}"
boot
Create a file boot.scr from boot.txt

$ mkimage -A arm -O linux -T script -C none -a 0 -e 0 -n "Panda SD Boot" -d boot.txt boot.scr
Flash the u-boot.img, MLO, boot.scr uImage and dtb file in the first partition.
cp -r boot.scr MLO u-boot.img omap4-panda-a4.dtb uImage /media/boot
Copy the rootfs in the second partition:
cp -r /tftpboot/arm/panda/* /media/rootfs



========== How to Build U-boot Image =========================
checkout uboot source from linaro
# git clone git://git.linaro.org/boot/u-boot-linaro-stable.git
for PandaBoard ES B3, use uboot source from svt
# git clone https://github.com/svtronics/u-boot-pandaboard-ES-RevB3.git
The source which was used to build uboot for BDK TE release had the following commit id
commit 921ce9c32de5561c27e000b11218de74437d462e
Author: Alexander Graf <agraf@suse.de>
Date:   Thu Mar 21 18:19:39 2013 +0100

    Exynos5: Fix errata 773022 and 774769 on Exynos5250 
for PandaBoard ES B3, use the following commit id
commit 0f459411f79975ce0f16ce11590b36feb0c2e864
Author: svtronics <svtronics.com>
Date:   Fri Sep 27 15:10:35 2013 +0530

Adding support for Elpida RAM for pandaboard-ES Rev B3
For this particular source, value of " CONFIG_SYS_MAXARGS" in include/configs/omap4_common.h file was changed from "32" to "64", as help command on uboot command was not working properly.

Build uboot
# cd u-boot-linaro-stable/  or cd u-boot-pandaboard-ES-RevB3/ (for PandaBoard ES B3)
# export CROSS_COMPILE=arm-linux-gnueabihf- 
# make omap4_panda_config 
# make 

Copy u-boot.img and MLO to boot partition
How to Build the kernel Image
To Build the kernel, clone the repository: 43.4.54.59:export/git/linux-3.10.y-BRANCH_SS.git.
Since, we are building it through armhfs, we export the following parameters :
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf- or USCEL armhf (majorly required)
Use the setup script :
# setup-panda-smp
Build the kernel as :
# make uImage LOADADDR=0x80008000
Build the DTB as:
# make dtbs
Copy the uImage and omap4-panda.dtb for PandaBoard A4 or omap4-panda-es.dtb for PandaBoard ES or omap4-panda.dtb for other PandaBoards? to tftp folder and boot the target.
NOTE, Please use appropriate DTB file depending on your PandaBoard revision
Build the kernel modules as:
# make modules
Install the kernel module at Respective location 
# make modules_install INSTALL_MOD_PATH=<User_created_Directory>

