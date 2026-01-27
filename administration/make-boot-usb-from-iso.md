# HOWTO make a bootable USB stick from isolinux ISO

The typical method for writing an ISO file to a USB stick in Linux is using the
dd command. However, there are instances where this approach may fail, and the
PC may ignore the USB flash drive, continuing to boot from the hard drive
instead. This issue can occur when the ISO image relies solely on the isolinux
bootloader. In such cases, it's necessary to repack the ISO image to include an
additional bootloader.

## Prerequisites

You need to install the `parted`, `libarchive`, `syslinux`, and `mtools`
packages. These packages should be available in the standard repositories. For
example, in Debian-based distributions:
```
apt-get install -y parted libarchive-tools syslinux mtools
```

Export path to the source image and target devices as variables:
```
export ISO_PATH=<path-to-iso-image>
export USB_DEV=<path-to-usb-stick-device>
```

Prepare mount points:
```
export VFAT_MOUNT=$(mktemp -d)
```

## Partitioning

Make two partitions on USB drive, boot partition and one which contains setup
files:
```
parted ${USB_DEV} mklabel msdos
parted ${USB_DEV} mkpart primary fat32 0% 100%
parted ${USB_DEV} set 1 boot on
parted ${USB_DEV} unit B print

mkfs.vfat -n BOOT ${USB_DEV}1
```

## Copy files to boot partition

Mount source ISO image:
```
mount ${USB_DEV}1 ${VFAT_MOUNT}
bsdtar -x -f ${ISO_PATH} -C ${VFAT_MOUNT}
``` 

## Add bootloader to the partition

Copy menu and all other files from old bootloader, then replace bootloader
files by syslinux ones and add syslinux menu which includes isolinux one:
```
export ISOLINUX_DIR=$(find ${VFAT_MOUNT} -type d -name 'isolinux')
export SYSLINUX_DIR=${VFAT_MOUNT}/boot/syslinux
mkdir -p ${SYSLINUX_DIR}
cp ${ISOLINUX_DIR}/* ${SYSLINUX_DIR}
cp /usr/lib/syslinux/bios/*.c32 ${SYSLINUX_DIR}
cat <<EOF >${SYSLINUX_DIR}/syslinux.cfg
path
prompt 0
timeout 0
include isolinux.cfg
default vesamenu.c32
EOF
umount ${VFAT_MOUNT}
```

Install bootloader:
```
syslinux --directory boot/syslinux --install ${USB_DEV}1
dd bs=440 count=1 conv=notrunc if=/usr/lib/syslinux/bios/mbr.bin of=${USB_DEV}
```

## Links

- [USB installation medium Arch Wiki
  page](https://wiki.archlinux.org/title/USB_flash_installation_medium)
- [Syslinux Arch Wiki page](https://wiki.archlinux.org/title/Syslinux)
