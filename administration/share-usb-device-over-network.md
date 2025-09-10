# HOWTO share USB device over network

USBIP allows sharing USB devices over network thus you can access your
web camera using USB interface even on a big distance from it. USBIP supports
USB 3.x speeds but you should ensure your network has bandwidth at least
1Gbit/s or higher.

All commands below require superuser access. USBIP usually doesn't require
additional packages to install but sometimes you need to install additional
packages:
```
apt-get install linux-tools-common linux-tools-`uname -r`
```

# Checking the configuration

## Server configuration (where device is connected)

Load USBIP server drivers:
```
modprobe usbip-core
modprobe usbip-host
```

Start USBIP server daemon:
```
usbipd -D
```

List local USB devices available:
```
usbip list -l
```

Share device over network:
```
usbip bind -b <busid>
```

## Client configuration (where device should be available)

Load USBIP drivers:
```
modprobe usbip-core
modprobe vhci-hcd
```

List USB devices available over network on server:
```
usbip list -r <server>
```

Attach to the remote USB device:
```
usbip attach -r <server> -b <busid>
```

# Making configuration permanent

## Server machine

Create `/etc/modules-load.d/usbip.conf` file:
```
usbip-core
usbip-host
```

Add `/etc/systemd/system/usbip-bind.service` to the client host:
```
[Unit]
Description=USBIP binding
Requires=network-online.target
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/usbipd
ExecStartPost=/usr/bin/usbip bind -b <busid>
ExecStopPre=/usr/bin/usbip unbind -b <busid>

[Install]
WantedBy=multi-user.target
```

Enable service:
```
systemctl daemon-reload
systemctl enable usbip-bind
```

## Client machine

Create `/etc/modules-load.d/usbip.conf` file:
```
usbip-core
vhci-hcd
```

Add `/etc/systemd/system/usbip-attach.service` to the client host:
```
[Unit]
Description=USBIP attaching
Requires=network-online.target
After=network-online.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/usbip attach -r <server> -b <busid>
ExecStop=/usr/bin/usbip detach -p <port>

[Install]
WantedBy=multi-user.target
```

You can find port using the following command after attaching to the device:
```
usbip port
```

Enable service:
```
systemctl daemon-reload
systemctl enable usbip-attach
```

# Links
- [USBIP project webside](https://usbip.sourceforge.net/)
- [StackExchange answer on systemd setup](https://unix.stackexchange.com/a/638097)
