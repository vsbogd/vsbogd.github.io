# HOWTO make a Windows 10 and 11 bootable USB stick on Linux

This is small rework of the article [Creating a bootable USB for Windows 10 and
11 on
Linux](https://linuxconfig.org/creating-a-bootable-usb-for-windows-10-and-11-on-linux)
which is more suitable for the manual run. You can also use the script from the
link above.

To make a bootable USB stick with Windows on Linux one should basically make
two partitions on the USB drive: one is bootable FAT32 partition and second one
is NTFS partition which contains installation files. After partitioning one
should copy files from the original image to the partitions.

## Prerequisites

Export path to the source image and target devices as variables:
```
export ISO_PATH=<path-to-windows-iso-image>
export USB_DEV=<path-to-usb-stick-device>
```

Prepare mount points:
```
export ISO_MOUNT=$(mktemp -d)
export VFAT_MOUNT=$(mktemp -d)
export NTFS_MOUNT=$(mktemp -d)
```

## Partitioning

Make two partitions on USB drive, boot partition and one which contains setup
files:
```
parted ${USB_DEV} mklabel gpt
parted ${USB_DEV} mkpart BOOT fat32 0% 1GiB
parted ${USB_DEV} mkpart INSTALL ntfs 1GiB 100%
parted ${USB_DEV} unit B print

mkfs.vfat -n BOOT ${USB_DEV}1
mkfs.ntfs --quick -L INSTALL ${USB_DEV}2
```

## Copy files to boot partition

Mount source image:
```
mount ${ISO_PATH} ${ISO_MOUNT}
``` 

Copy boot files:
```
mount ${USB_DEV}1 ${VFAT_MOUNT}
rsync -r --progress --exclude sources --delete-before ${ISO_MOUNT}/ ${VFAT_MOUNT}/
mkdir -p ${VFAT_MOUNT}/sources
rsync --progress ${ISO_MOUNT}/sources/boot.wim ${VFAT_MOUNT}/sources/
umount ${VFAT_MOUNT}
```

Copy install files:
```
mount ${USB_DEV}2 ${NTFS_MOUNT}
rsync -r --progress --delete-before ${ISO_MOUNT}/ ${NTFS_MOUNT}/
umount ${NTFS_MOUNT}
```

## Finish

Unmount source image, remove mount points, safely switch-off USB drive:
```
umount ${ISO_MOUNT}
rmdir ${ISO_MOUNT} ${VFAT_MOUNT} ${NTFS_MOUNT}
udisksctl power-off -b ${USB_DEV}
```
