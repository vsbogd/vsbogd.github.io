# HOWTO setup traffic routing via OpenVPN server using NAT

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
- `cat /proc/sys/net/ipv4/ip_forward` outputs `1`. 
- `sudo iptables-save | grep FORWARD` outputs:
    ```
    :FORWARD DROP [0:0]
    -A FORWARD -i <VPNIF> -j ACCEPT
    -A FORWARD -o <VPNIF> -m state --state ESTABLISHED -j ACCEPT
    ```
- `sudo iptables-save | grep POSTROUTING` outputs:
    ```
    :POSTROUTING ACCEPT [0:0]
    -A POSTROUTING -o <WANIF> -j MASQUERADE
    ```

Links:
- [OpenVPN official traffic routing
  HOWTO](https://openvpn.net/community-resources/how-to/#routing-all-client-traffic-including-web-traffic-through-the-vpn)
- [Netfilter Source NAT
  HOWTO](https://www.netfilter.org/documentation/HOWTO/NAT-HOWTO-6.html#ss6.1)
