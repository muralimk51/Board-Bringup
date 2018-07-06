# Board-Bringup
#Board Bringup with Custom images Considering "Panda Board"

======= Install CROSS COMPILER =======================
# Install the cross compiler on host machine and dependency packages.
sudo apt-get install gcc-arm-linux-gnueabihf
sudo apt-get install git libncurses5 libncurses5-dev libnewt-dev


======= u-boot =====================================
# Downloaded u-boot
git clone https://github.com/u-boot/u-boot
 
### u-boot compilation
  git checkout v2017.03 -b tmp
  export ARCH=arm
  export CROSS_COMPILE=arm-linux-gnueabi-
  make distclean && make mrproper
  make omap4_panda_defconfig
  make
### compilation error from above step from 'arch/arm/config.mk'
 {{{
Your GCC is older than 6.0 and is not supported
make: [checkgcc6] Error 1
make: INTERNAL: Exiting with 9 jobserver tokens available; should be 8!
 }}}

# To resolve above download 'linaro toolcahin' or checkout older version of u-boot branch 
http://releases.linaro.org/components/toolchain/binaries/6.2-2016.11/



======= kernel ==========================================
# Download linux kernel and cross compiled for the Panda board
git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git

### kernel compilation
  make omap2plus_defconfig
  make –j8 zImage dtbs (or) make –j8 uImage dtbs LOADADDR=0x80008000



========== rootfs: BUIDROOT  ======================================= 
# Preapared ext4 type rootfs using build root.
git clone git://git.buildroot.net/buildroot

### compilation
  make pandaboard_defconfig
  make



======= Set Bootargs ==============================================
#  boot arguments maintained in 'boot.txt' file:
  setenv bootargs console=ttyO2,115200n8 root=/dev/mmcblk0p2 rw rootwait rootfstype=ext4
  setenv dtbaddr 0x815f0000
  setenv bootcmd 'fatload mmc 0:1 0x80200000 zImage; fatload mmc 0:1 ${dtbaddr} omap4-panda-a4.dtb;bootz 0x80200000 - ${dtbaddr}'
  boot

### Created 'boot.scr' file by using
  mkimage -A arm -O linux -T script -C none -a 0 -e 0 -n "Panda SD Boot" -d boot.txt boot.scr

 

========= Bring up log ==========================================
# Verify
cat /proc/version 



======================== REFERENCES =========================================
1. https://eewiki.net/display/linuxonarm/PandaBoard
2. https://kasiviswanathanblog.wordpress.com/2017/11/24/booting-linux-kernel-from-tftp-and-nfs-file-system-for-pandaboard/  <-- nfsboot
3. https://wiki.debian.org/InstallingDebianOn/PandaBoard
4. http://releases.linaro.org/components/toolchain/binaries/6.2-2016.11/
4. 
