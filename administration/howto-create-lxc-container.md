[LXC getting starting](https://linuxcontainers.org/lxc/getting-started/)

```
sudo apt-get install lxc uidmap cgmanager libvirt-clients

sudo sh -c 'echo 1 > /sys/fs/cgroup/cpuset/cgroup.clone_children'
sudo sh -c 'echo 1 > /proc/sys/kernel/unprivileged_userns_clone'

sudo adduser -u 2001 lxc

$ cat /etc/lxc/lxc-usernet
lxc veth lxcbr0 10

sudo cgm create all lxc
sudo cgm chown all lxc $(id -u lxc) $(id -g lxc)
```

Logout/login as lxc

```
mkdir -p ~/.config/lxc
cp /etc/lxc/default.conf ~/.config/lxc

$ cat /etc/sub{uid,gid} | grep lxc
lxc:1279648:65536
lxc:1279648:65536

$ cat ~/.config/lxc/default.conf
lxc.network.type = empty

# for old LXC version
lxc.id_map = u 0 1279648 65536
lxc.id_map = g 0 1279648 65536

# for new LXC version
lxc.idmap = u 0 1279648 65536
lxc.idmap = g 0 1279648 65536

lxc-create -t download -n ${CONTAINER_NAME}

cgm movepid all $USER $PPID #? $$

```
