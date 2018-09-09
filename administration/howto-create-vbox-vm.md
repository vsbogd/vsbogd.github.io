Full user manual: [https://www.virtualbox.org/manual/ch08.html](https://www.virtualbox.org/manual/ch08.html)

Things you need to define:
```
export VMNAME=<name>
export HDDNAME=./HardDisks/${VMNAME}.vdi
export SSHPORT=<ssh-to-use-from-host-machine>
export VNCPORT=<vnc-to-use-from-host-machine>
export HOST=<host-machine>
```

Create hard drive for the guest system.

Old syntax:
```
VBoxManage createhd --filename ${HDDNAME} --size <size> --format VDI
# new syntax: VBoxManage createmedium disk --filename ${HDDNAME} --size <size> --format VDI
```

Create and register new virtual machine.
```
VBoxManage createvm --name ${VMNAME} --ostype Debian_64 --register
```

Modify VM properties:
```
VBoxManage modifyvm ${VMNAME} --memory 1024 --vrde on --vrdeport ${VNCPORT} --nic1 nat
```

Add port forwarding rules if required. ```--natpf<1-N>``` is an id of a NAT network
interface to apply rule, if system has only network interface then all rules
should be applied using ```--natpf1```:
```
VBoxManage modifyvm ${VMNAME} --natpf1 ssh,tcp,127.0.0.1,${SSHPORT},,22
# VBoxManage modifyvm ${VMNAME} --natpf<1-N> [<rulename>],tcp|udp,[<hostip>],<hostport>,[<guestip>],<guestport>
```

Add SATA controller:
```
VBoxManage storagectl ${VMNAME} --name sata1 --add sata --bootable on
```

Add hard drive:
```
VBoxManage storageattach ${VMNAME} --storagectl sata1 --port 0 --device 0 --type hdd --medium ${HDDNAME}
```

Add DVD drive to install an operating system:
```
VBoxManage storageattach ${VMNAME} --storagectl sata1 --port 1 --device 0 --type dvddrive --medium ./debian-9.5.0-amd64-netinst.iso
```

Start VM:
```
VBoxHeadless --startvm ${VMNAME}
```

Connect to VM using VNC client:
```
vncviewer ${HOST}:${VNCPORT}
```

Install operating system. Remove install disk:
```
VBoxManage storageattach ${VMNAME} --storagectl sata1 --port 1 --device 0 --type dvddrive --medium none
```
