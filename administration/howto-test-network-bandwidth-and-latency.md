# HOWTO test network bandwidth and latency

Links:
- [iPerf3](https://iperf.fr/)
- [MTR](http://www.bitwizard.nl/mtr/)

## Prerequisites

In order to measure network bandwidth you need two hosts. We will say one of
them is a server and second one is a client. We will collect information about
connections between them.

Install `iperf3` tool on both a server and a client:
```
apt-get install -y iperf3
```

Install `mtr` tool on a client:
```
apt-get install -y mtr
```

## Prepare a server

Choose a port (default one is 5001) and keep it in `PORT` environment variable.
Run `iperf3` in a server mode on a server side for both TCP and UDP protocols:
```
PORT=5001
iperf3 -s -p $PORT
iperf3 -s -u -p $PORT
```
Ensure TCP and UDP ports are opened and ICMP traffic is allowed by the server.

## Collecting latency

On a client side use `mtr` to measure latency. I used `HOST` variable to keep
name of the server here and below:
```
HOST=192.168.0.1
mtr -r -c 60 -4 -o 'LDRS NBAWVG JMXI' $HOST
```

## Collecting bandwidth

On a client side use `iperf3` to collect bandwidth information. Use `-R` to
measure a bandwidth in reverse direction:
```
PORT=5001

echo "Collecting bandwidth information TCP"
iperf3 -c $HOST -p $PORT -V -4 -t 60 --get-server-output
iperf3 -c $HOST -p $PORT -V -4 -t 60 --get-server-output -R

echo "Collecting bandwidth information UDP"
iperf3 -c $HOST -p $PORT -V -4 -t 60 --get-server-output -u -b 30m
iperf3 -c $HOST -p $PORT -V -4 -t 60 --get-server-output -u -b 30m -R
```
