# HOWTO setup NAT and traffic forwarding via OpenVPN server

Create file `/etc/openvpn/server/<config-name>.up`. Put the following script
into the file:
```
#!/bin/bash

VPNIF=${dev}
WANIF=`ip route | grep default | egrep -o 'dev [^ ]+' | cut -f 2 -d\  `

iptables -P FORWARD DROP
iptables -A FORWARD -i ${VPNIF} -j ACCEPT
iptables -A FORWARD -o ${VPNIF} -m state --state ESTABLISHED -j ACCEPT
iptables -t nat -A POSTROUTING -o ${WANIF} -j MASQUERADE

echo 1 > /proc/sys/net/ipv4/ip_forward
```

Make this file executable:
`sudo chmod a+x /etc/openvpn/server/<config-name>.up`

Add the following lines into a server OpenVPN config:
```
script-security 2
up /etc/openvpn/server/<config-name>.up
```

Restart OpenVPN server, check that:
```
cat /proc/sys/net/ipv4/ip_forward
# prints 1
sudo iptables-save | grep FORWARD
# prints
# :FORWARD DROP [0:0]
# -A FORWARD -i <VPNIF> -j ACCEPT
# -A FORWARD -o <VPNIF> -m state --state ESTABLISHED -j ACCEPT
sudo iptables-save | grep POSTROUTING
# prints
# :POSTROUTING ACCEPT [0:0]
# iptables -t nat -A POSTROUTING -o <WANIF> -j MASQUERADE
```
