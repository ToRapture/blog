---
title: routetrace
date: 2020-04-26 20:03:00
mathjax: true
tags:
- Network
---

## Command
https://linux.die.net/man/8/traceroute
> This program attempts to trace the route an IP packet would follow to some internet host by launching probe packets with a small ttl (time to live) then listening for an ICMP "time exceeded" reply from a gateway. We start our probes with a ttl of one and increase by one until we get an ICMP "port unreachable" (or TCP reset), which means we got to the "host", or hit a max (which defaults to 30 hops). Three probes (by default) are sent at each ttl setting and a line is printed showing the ttl, address of the gateway and round trip time of each probe. The address can be followed by additional information when requested. If the probe answers come from different gateways, the address of each responding system will be printed. If there is no response within a 5.0 seconds (default), an "*" (asterisk) is printed for that probe.

```
$ traceroute iqiyi.com
traceroute to iqiyi.com (111.206.13.64), 30 hops max, 60 byte packets
 1  * * *
 2  11.220.36.9 (11.220.36.9)  6.906 ms 11.220.37.73 (11.220.37.73)  5.219 ms 11.220.36.9 (11.220.36.9)  7.193 ms
 3  * * *
 4  10.54.136.253 (10.54.136.253)  0.419 ms 11.217.38.218 (11.217.38.218)  0.366 ms 10.54.137.5 (10.54.137.5)  7.548 ms
 5  119.38.215.78 (119.38.215.78)  1.215 ms 42.120.253.1 (42.120.253.1)  1.148 ms 117.49.35.198 (117.49.35.198)  1.040 ms
 6  116.251.113.145 (116.251.113.145)  1.798 ms 42.120.239.249 (42.120.239.249)  2.112 ms 116.251.113.149 (116.251.113.149)  2.287 ms
 7  157.255.237.149 (157.255.237.149)  5.927 ms  5.809 ms  5.758 ms
 8  120.80.99.17 (120.80.99.17)  5.375 ms 120.80.98.177 (120.80.98.177)  4.409 ms 120.80.98.181 (120.80.98.181)  6.039 ms
 9  221.4.0.181 (221.4.0.181)  9.088 ms  8.350 ms  8.086 ms
10  219.158.7.21 (219.158.7.21)  35.774 ms  35.622 ms 219.158.7.17 (219.158.7.17)  39.304 ms
11  124.65.194.34 (124.65.194.34)  41.728 ms 202.96.12.26 (202.96.12.26)  40.799 ms 125.33.186.50 (125.33.186.50)  38.370 ms
12  202.106.34.98 (202.106.34.98)  72.417 ms  67.925 ms  66.970 ms
13  bt-211-070.bta.net.cn (202.106.211.70)  39.312 ms * *
14  111.206.13.64 (111.206.13.64)  39.598 ms  36.527 ms  39.293 ms
```

## Simulate by ping
```
$ ping -c2 -W 1.5 -t 1 iqiyi.com
PING iqiyi.com (111.206.13.64) 56(84) bytes of data.

--- iqiyi.com ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1007ms

$ ping -c2 -W 1.5 -t 2 iqiyi.com
PING iqiyi.com (111.206.13.63) 56(84) bytes of data.
From 11.220.36.73 icmp_seq=1 Time to live exceeded
From 11.220.36.73 icmp_seq=2 Time to live exceeded

--- iqiyi.com ping statistics ---
2 packets transmitted, 0 received, +2 errors, 100% packet loss, time 1001ms

$ ping -c2 -W 1.5 -t 3 iqiyi.com
PING iqiyi.com (111.206.13.61) 56(84) bytes of data.
From 11.220.37.130 icmp_seq=1 Time to live exceeded
From 11.220.37.130 icmp_seq=2 Time to live exceeded

--- iqiyi.com ping statistics ---
2 packets transmitted, 0 received, +2 errors, 100% packet loss, time 1001ms

$ ping -c2 -W 1.5 -t 4 iqiyi.com
PING iqiyi.com (111.206.13.64) 56(84) bytes of data.
From 10.54.140.5 icmp_seq=1 Time to live exceeded
From 10.54.140.5 icmp_seq=2 Time to live exceeded

--- iqiyi.com ping statistics ---
2 packets transmitted, 0 received, +2 errors, 100% packet loss, time 1001ms

$ ping -c2 -W 1.5 -t 5 iqiyi.com
PING iqiyi.com (111.206.13.63) 56(84) bytes of data.
From 117.49.35.62 icmp_seq=1 Time to live exceeded
From 117.49.35.62 icmp_seq=2 Time to live exceeded

--- iqiyi.com ping statistics ---
2 packets transmitted, 0 received, +2 errors, 100% packet loss, time 1001ms

$ ping -c2 -W 1.5 -t 6 iqiyi.com
PING iqiyi.com (111.206.13.65) 56(84) bytes of data.
From 42.120.242.229 icmp_seq=1 Time to live exceeded
From 42.120.242.229 icmp_seq=2 Time to live exceeded

--- iqiyi.com ping statistics ---
2 packets transmitted, 0 received, +2 errors, 100% packet loss, time 1000ms

--------

$ ping -c2 -W 1.5 -t 12 iqiyi.com
PING iqiyi.com (111.206.13.66) 56(84) bytes of data.
From 123.126.0.46 icmp_seq=1 Time to live exceeded
From 123.126.0.46 icmp_seq=2 Time to live exceeded

--- iqiyi.com ping statistics ---
2 packets transmitted, 0 received, +2 errors, 100% packet loss, time 1001ms

$ ping -c2 -W 1.5 -t 13 iqiyi.com
PING iqiyi.com (111.206.13.62) 56(84) bytes of data.
From bt-211-042.bta.net.cn (202.106.211.42) icmp_seq=1 Time to live exceeded
From bt-211-042.bta.net.cn (202.106.211.42) icmp_seq=2 Time to live exceeded

--- iqiyi.com ping statistics ---
2 packets transmitted, 0 received, +2 errors, 100% packet loss, time 1001ms

$ ping -c2 -W 1.5 -t 14 iqiyi.com
PING iqiyi.com (111.206.13.61) 56(84) bytes of data.
64 bytes from 111.206.13.61: icmp_seq=1 ttl=52 time=36.7 ms
64 bytes from 111.206.13.61: icmp_seq=2 ttl=52 time=36.7 ms

--- iqiyi.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 36.726/36.737/36.748/0.011 ms
```