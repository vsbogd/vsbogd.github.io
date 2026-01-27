# HOWTO make a bootable USB stick from isolinux ISO

Usual way of writing of the ISO file onto USB stick under Linux is using dd.
But sometimes this way doesn't work. PC just ignores USB flash drive and
continues booting from the hard drive. For example this may happen if isolinux
bootloader is the only bootloader used by the ISO image. In this case one need
to repack ISO image and add new bootloader to it.

## Prerequisites

One need to install `parted`, `libarchive`, `syslinux` and `mtools` packages.
They should be available in standard repositories. For example in Debian based
distributions:
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

Mount source image:
```
mount ${USB_DEV}1 ${VFAT_MOUNT}
bsdtar -x -f ${ISO_PATH} -C ${VFAT_MOUNT}
``` 

## Add bootloader to the partition

Copy menu and all other files from old bootloader, then replace bootloader
files by Syslinux ones and add Syslinux menu which includes Isolinux one:
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

Add bootloader to the disk:
```
syslinux --directory boot/syslinux --install ${USB_DEV}1
dd bs=440 count=1 conv=notrunc if=/usr/lib/syslinux/bios/mbr.bin of=${USB_DEV}
```

## Links

- [USB installation medium Arch Wiki
  page](https://wiki.archlinux.org/title/USB_flash_installation_medium)
- [Syslinux Arch Wiki page](https://wiki.archlinux.org/title/Syslinux)
