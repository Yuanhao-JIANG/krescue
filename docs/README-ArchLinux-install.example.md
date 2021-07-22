# HOW-TO install ArchLinux for any Khadas SBC

## Start Krescue from SD

+ download appropriate Krescue sd/usb image  https://dl.khadas.com/Firmware/Krescue/system/
for your device and bootup Krescue

## Preparation notes

+ your SBC must connected to internet via ethernet or Wi-Fi
+ ethernet will auto-connect if a LAN cable is plugged in before boot-up

## WIFI configuration

If you’re currently in a console / shell, just type krescue or exit to return to Krescue’s main menu

+ krescue main menu -> network -> wifi -> connect /select

## install Alt-Lihux into eMMC from shell

Going into Krescue shell.

+ krescue main menu -> shell

### oneline shell script

Just copy paste next line.

    curl -jkL https://raw.githubusercontent.com/khadas/krescue/master/scripts/ArchLinux-install.sh | sh -s -

### manual install variant example

Step by step mode

```
BOARD=$(tr -d '\0' < /sys/firmware/devicetree/base/model || echo Khadas)
echo "ArchLinux installation for $BOARD ..."

# create partitions
echo "label: dos" | sfdisk $(mmc_disk)
echo "part1 : start=16M," | sfdisk $(mmc_disk)

# create rootfs
mkfs.ext4 -L ROOT $(mmc_disk)p1 < /dev/null
mkdir -p system && mount $(mmc_disk)p1 system

# can chouse any other rootfs source
SRC=http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz

echo "download and extract $SRC"
curl -A downloader -jkL $SRC | pigz -dc | tar -xf- -C system

# setup extlinux config
mkdir -p system/boot/extlinux/
cat <<-END > system/boot/extlinux/extlinux.conf
label ArchLinux
kernel /boot/Image.gz
initrd /boot/initramfs-linux.img
fdtdir /boot/dtbs
append root=LABEL=ROOT
END

# setup rootfs
echo LABEL=ROOT / auto errors=remount-ro 1 1 >> system/etc/fstab

# setup host name
echo $BOARD > system/etc/hostname

# setup dhcp for ethernet
echo dhcpcd eth0 -d > system/etc/rc.local
chmod 0777 system/etc/rc.local

# setup secure tty
echo ttyAML0 >> system/etc/securetty
echo ttyFIQ0 >> system/etc/securetty

umount system

# install uboot to eMMC
mmc_update_uboot online

# optional install uboot to SPI flash
spi_update_uboot online -k && echo need poweroff and poweron device again

# DONE plz reboot device
```

## install Alt-Lihux into eMMC via ssh

Device with Krescue must be available by krescue.local hostname inside local net

    wget https://raw.githubusercontent.com/khadas/krescue/master/scripts/ArchLinux-install.sh
    ssh root@krescue.local < ArchLinux-install.sh

## user/root access

    user: root
    password: root

## Problems

+ not detected
+ not tested

ENJOY ;-)

## LINKS

+ https://archlinuxarm.org/
+ https://archlinuxarm.org/platforms/armv8/generic
+ https://wiki.archlinux.org/title/Arch_Linux

+ https://dl.khadas.com/Firmware/Krescue/system
+ https://dl.khadas.com/Firmware/Krescue/#Info
+ https://dl.khadas.com/Firmware/Krescue/system/README-rescue-boot.txt
+ https://dl.khadas.com/Firmware/Krescue/system/README-rescue-access.txt
+ https://github.com/khadas/krescue/blob/master/docs/README-ArchLinux-install.example.md