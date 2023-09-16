# HOWTO reset Windows 10 password using Linux

This instruction helps in case you forgot password from Windows account or
Microsoft Live account you used to login and you cannot reset your password.

You can create a boot disk with registry editor and using this editor get the
command line with administrative privileges in Windows to fix this issue. You
need to be able to boot computer from USB stick.

Bootdisk can be created using Windows or Linux computer. Below I explain how to
do it using Linux, but similar steps can be used in Windows environment.

## Make a rescue USB stick

Make a FAT32 partition on USB drive:
```
export DEV=/dev/<usb-drive>
parted ${DEV} rm 1-99
parted ${DEV} mkpart primary fat32 0% 100%
parted ${DEV} set 1 boot on
mkfs.vfat -F32 ${DEV}1
```

Download or install [Syslinux](https://wiki.syslinux.org) package and use it to
make disk bootable:
```
syslinux -i ${DEV}1
dd conv=notrunc bs=440 count=1 if=<mbr.bin> of=${DEV}
```
`<mbr.bin>` is a full path to this file which is a part of the syslinux
package.

Download offline password recovery and registry editor [boot disk
files](https://pogostick.net/~pnh/ntpasswd/usb140201.zip). Unpack archive and
copy content to the USB drive:
```
wget -O /tmp/usb140201.zip https://pogostick.net/~pnh/ntpasswd/usb140201.zip
mkdir /tmp/ntpass
cd /tmp/ntpass
unzip /tmp/usb140201.zip
sudo mount ${DEV}1 /mnt -o uid=`whoami`
cp * /mnt
sudo umount /mnt
```

Now you have a USB stick that can be used to edit Windows registry.

## Run Windows command line with administrator privileges

Boot Windows computer using USB stick. See
[instruction](https://pogostick.net/~pnh/ntpasswd/walkthrough.html) on how to
use it. You need to run registry editor on a `SYSTEM` hive. Then go to `Setup`
registry directory. Set `SetupType` value to `2` and `CmdLine` value to `cmd`.
Close editor and reboot Windows machine.

After a while you should see a command line console. List all users of the
system by `net users`. You goal is to enable `Administrator` user and reset its
password:
```
net users
net users Administrator <new-password>
net users Administrator /active:yes
```
You may also lock the account which password you don't remember:
```
net users <username> /active:no
```
You can try to reset its password but if your user is authenticated using
Microsoft Live account it doesn't work.

Run `regedit` open `HKEY_LOCAL_MACHINE\SYSTEM\Setup` registry directory and set
`SetupType` value to `0` and `CmdLine` value to empty string. Close the command
line window and system will be restarted. Now you should be able to login as an
Administrator using `<new-password>`.

## Fixing the login issue

After logging in as Administrator you can execute `control userpassword2`
command in order to edit user passwords. If account under the question is local
you can just reset its password. If it is a Microsoft Live account then the
only way is to create new user, copy all files from the old account and remove
old account.

## Links

Links:
- [Parted User's Manual](https://www.gnu.org/software/parted/manual/parted.html)
- [Syslinux HOWTO](https://wiki.syslinux.org/wiki/index.php?title=HowTos)
- [Offline NT Password and Registry Editor](https://pogostick.net/~pnh/ntpasswd/)
