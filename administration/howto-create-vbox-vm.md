# HOWTO Create VirtualBox VM using command line

## References

Full user manual: [https://www.virtualbox.org/manual/ch08.html](https://www.virtualbox.org/manual/ch08.html)

## Preparation

Before starting doing something you should think about the following things:
- name of VM to use it in command line tool parameters
- memory size; I use 1024Mb
- hard drive size, and location
- ports you will allocate on the host machine to connect your VM:
    - VNC port
    - SSH port - optional
- operating system type; I use ```Debian_64```; you can list all OS types
  supported using ```VBoxManage list ostypes```

For the sake of simplicity I will use the following variables below:
```
export VMNAME=<name>
export HDDNAME=${HOME}/.VirtualBox/HardDisks/${VMNAME}.vdi
export SSHPORT=<ssh-to-use-from-host-machine>
export VNCPORT=<vnc-to-use-from-host-machine>
export HOST=<host-machine>
```

## Install VirtualBox

Install VirtualBox itself and extension pack to make VRDP work:
```
sudo apt-get install virtualbox virtualbox-ext-pack
```

## Create VM

Create hard drive for the guest system.
```
VBoxManage createmedium disk --filename ${HDDNAME} --size <size> --format VDI
# old syntax: VBoxManage createhd --filename ${HDDNAME} --size <size> --format VDI
```

Create and register new virtual machine.
```
VBoxManage createvm --name ${VMNAME} --ostype Debian_64 --register
```

Modify VM properties to add VNC remote control, add network interface and
change memory size if required:
```
VBoxManage modifyvm ${VMNAME} --memory 1024 --vrde on --vrdeport ${VNCPORT} --nic1 nat
```

Add port forwarding rules if required. ```--natpf<1-N>``` is an id of a NAT network
interface to apply rule, if system has only network interface then all rules
should be applied using ```--natpf1```. For instance add forwarding for the SSH
server port:
```
VBoxManage modifyvm ${VMNAME} --natpf1 ssh,tcp,127.0.0.1,${SSHPORT},,22
# full rule syntax: VBoxManage modifyvm ${VMNAME} --natpf<1-N> [<rulename>],tcp|udp,[<hostip>],<hostport>,[<guestip>],<guestport>
```

Add bootable SATA controller:
```
VBoxManage storagectl ${VMNAME} --name sata1 --add sata --bootable on
```

Add hard drive:
```
VBoxManage storageattach ${VMNAME} --storagectl sata1 --port 0 --device 0 --type hdd --medium ${HDDNAME}
```

## Start/stop VM

Show VM information:
```
VBoxManage showvminfo ${VMNAME} | less
```

Start VM:
```
VBoxHeadless --startvm ${VMNAME}
```

Stop VM:
```
VBoxManage controlvm ${VMNAME} acpipowerbutton
```

## Install operating system

Add DVD drive to install an operating system.
I use Debian netinstall disk which is available at
[Getting Debian](https://www.debian.org/distrib/) page.
```
VBoxManage storageattach ${VMNAME} --storagectl sata1 --port 1 --device 0 --type dvddrive --medium ./debian-9.5.0-amd64-netinst.iso
```

Start your VM and connect to it using VNC client:
```
VBoxHeadless --startvm ${VMNAME}
vncviewer ${HOST}:${VNCPORT}
```

Install operating system. Remove install disk after installation:
```
VBoxManage storageattach ${VMNAME} --storagectl sata1 --port 1 --device 0 --type dvddrive --medium none
```

Install guest additions. You can download guest additions for different virtual
box versions at [VirtualBox downloads](http://download.virtualbox.org/virtualbox/)

