# HOWTO simulate network bandwidth and latency

Links:
- [Simulate your datacenters latency with Linux](https://blog.worldline.tech/2020/11/26/simulate-datacenters-latency-with-linux.html)
- [Traffic Control HOWTO](https://tldp.org/HOWTO/html_single/Traffic-Control-HOWTO/)
- [tc-netem(8) man pages](https://www.man7.org/linux/man-pages/man8/tc-netem.8.html)

Sometimes you need to find out how does your application work when bandwidth
and latency are not perfect. Below you can find recipes to simulate restricted
bandwidth, high latency and even package loss.

You need to choose network interface for the experiments and save its name into
`IF` variable:
```
export IF=enp0s31f6
export IP1=192.168.0.1
export IP2=192.168.0.2
```

Setup htb queue discipline on the interface (all commands below require sudo):
```
tc qdisc add dev $IF root handle 1:0 htb
tc qdisc show dev $IF
```

Add traffic class(es) with bandwidth restricted to 20Mbit:
```
tc class add dev $IF parent 1:0 classid 1:1 htb rate 20mbit
tc class show dev $IF
```

Add filters to assign traffic classes for the IPs of the packets:
```
tc filter add dev $IF protocol ip parent 1:0 prio 1 u32 match ip dst $IP1/32 match ip src $IP2/32 flowid 1:1
tc filter add dev $IF protocol ip parent 1:0 prio 1 u32 match ip dst $IP2/32 match ip src $IP1/32 flowid 1:1
tc filter show dev $IF
```

Use netem module to simulate latency (150ms) and loss (1%):
```
tc qdisc add dev $IF parent 1:1 handle 2:1 netem delay 150ms loss 1%
```
