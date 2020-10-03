---
title: Commands for IP
date: 2020-04-24 16:47:00
mathjax: true
tags:
- Network
- TCP/IP
---

## ifconfig
https://linux.die.net/man/8/ifconfig
> Ifconfig is used to configure the kernel-resident network interfaces. It is used at boot time to set up interfaces as necessary. After that, it is usually only needed when debugging or when system tuning is needed.
If no arguments are given, ifconfig displays the status of the currently active interfaces. If a single interface argument is given, it displays the status of the given interface only; if a single -a argument is given, it displays the status of all interfaces, even those that are down. Otherwise, it configures an interface.

```
$ ifconfig
eth0      Link encap:Ethernet  HWaddr 00:16:3e:10:27:9e
          inet addr:172.16.62.130  Bcast:172.16.63.255  Mask:255.255.192.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:10188512 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10123903 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:1554718007 (1.5 GB)  TX bytes:15002653955 (15.0 GB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:5561316 errors:0 dropped:0 overruns:0 frame:0
          TX packets:5561316 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:720550645 (720.5 MB)  TX bytes:720550645 (720.5 MB)

$ ifconfig <interface> up
$ ifconfig <interface> down
```

- - -

## route
https://linux.die.net/man/8/route
> Route manipulates the kernel's IP routing tables. Its primary use is to set up static routes to specific hosts or networks via an interface after it has been configured with the ifconfig(8) program.
When the add or del options are used, route modifies the routing tables. Without these options, route displays the current contents of the routing tables.

- - -
https://superuser.com/a/580674
> The Gateway column identifies the defined gateway for the specified network. An asterisk (*) appears in this column if no forwarding gateway is needed for the network.

```
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.16.63.253   0.0.0.0         UG    0      0        0 eth0
172.16.0.0      *               255.255.192.0   U     0      0        0 eth0

$ host baidu.com
baidu.com has address 220.181.38.148
baidu.com has address 39.156.69.79
baidu.com mail is handled by 15 mx.n.shifen.com.
baidu.com mail is handled by 20 mx50.baidu.com.
baidu.com mail is handled by 10 mx.maillb.baidu.com.
baidu.com mail is handled by 20 jpmx.baidu.com.
baidu.com mail is handled by 20 mx1.baidu.com.

$ ip route get 220.181.38.148
220.181.38.148 via 172.16.63.253 dev eth0  src 172.16.62.130
    cache

$ ping -c1 220.181.38.148
PING 220.181.38.148 (220.181.38.148) 56(84) bytes of data.
64 bytes from 220.181.38.148: icmp_seq=1 ttl=49 time=36.6 ms

--- 220.181.38.148 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 36.620/36.620/36.620/0.000 ms
```

```
$ sudo route add 220.181.38.148 eth0

$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.16.63.253   0.0.0.0         UG    0      0        0 eth0
172.16.0.0      *               255.255.192.0   U     0      0        0 eth0
220.181.38.148  *               255.255.255.255 UH    0      0        0 eth0

$ ip route get 220.181.38.148
220.181.38.148 dev eth0  src 172.16.62.130
    cache

$ ping -c1 220.181.38.148
PING 220.181.38.148 (220.181.38.148) 56(84) bytes of data.
From 172.16.62.130 icmp_seq=1 Destination Host Unreachable

--- 220.181.38.148 ping statistics ---
1 packets transmitted, 0 received, +1 errors, 100% packet loss, time 0ms

$ sudo route del 220.181.38.148

$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.16.63.253   0.0.0.0         UG    0      0        0 eth0
172.16.0.0      *               255.255.192.0   U     0      0        0 eth0
```

```
$ sudo route add 220.181.38.148 gw 172.16.63.253

$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.16.63.253   0.0.0.0         UG    0      0        0 eth0
172.16.0.0      *               255.255.192.0   U     0      0        0 eth0
220.181.38.148  172.16.63.253   255.255.255.255 UGH   0      0        0 eth0

$ ip route get 220.181.38.148
220.181.38.148 via 172.16.63.253 dev eth0  src 172.16.62.130
    cache

$ ping -c1 220.181.38.148
PING 220.181.38.148 (220.181.38.148) 56(84) bytes of data.
64 bytes from 220.181.38.148: icmp_seq=1 ttl=49 time=36.5 ms

--- 220.181.38.148 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 36.538/36.538/36.538/0.000 ms

$ sudo route del 220.181.38.148

$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.16.63.253   0.0.0.0         UG    0      0        0 eth0
172.16.0.0      *               255.255.192.0   U     0      0        0 eth0
```

- - -

## iptables
https://linux.die.net/man/8/iptables
> iptables - administration tool for IPv4 packet filtering and NAT

### default
```
root@nenuoj$ iptables -L -v --line-numbers
Chain INPUT (policy ACCEPT 290 packets, 18937 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 202 packets, 41678 bytes)
num   pkts bytes target     prot opt in     out     source               destination

torapture@rapture$ ping -c4 D.D.D.D
PING D.D.D.D (D.D.D.D) 56(84) bytes of data.
64 bytes from D.D.D.D: icmp_seq=1 ttl=45 time=192 ms
64 bytes from D.D.D.D: icmp_seq=2 ttl=45 time=180 ms
64 bytes from D.D.D.D: icmp_seq=3 ttl=45 time=194 ms
64 bytes from D.D.D.D: icmp_seq=4 ttl=45 time=180 ms

--- D.D.D.D ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2998ms
rtt min/avg/max/mdev = 180.029/186.853/194.475/6.724 ms
```

### drop
```
root@nenuoj$ iptables -A INPUT -s S.S.S.S -j DROP

torapture@rapture$ ping -c4 D.D.D.D
PING D.D.D.D (D.D.D.D) 56(84) bytes of data.

--- D.D.D.D ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3024ms

root@nenuoj$ iptables -L -v --line-numbers
Chain INPUT (policy ACCEPT 54 packets, 3339 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        4   336 DROP       all  --  any    any     S.S.S.S       anywhere

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 44 packets, 13276 bytes)
num   pkts bytes target     prot opt in     out     source               destination
```

### reject tcp:80
```
root@nenuoj$ iptables -A INPUT -s S.S.S.S -p tcp --destination-port 80 -j REJECT

torapture@rapture$ curl D.D.D.D
curl: (7) Failed to connect to D.D.D.D port 80: Connection refused

root@nenuoj$ iptables -L -v --line-numbers
Chain INPUT (policy ACCEPT 91 packets, 5449 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        1    60 REJECT     tcp  --  any    any     S.S.S.S       anywhere             tcp dpt:http reject-with icmp-port-unreachable

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 70 packets, 14548 bytes)
num   pkts bytes target     prot opt in     out     source               destination
```

- - -

## ip
https://linux.die.net/man/8/ip
> ip - show / manipulate routing, devices, policy routing and tunnels

### address
```
root@nenuoj$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:16:3e:10:27:9e brd ff:ff:ff:ff:ff:ff
    inet 172.16.62.130/18 brd 172.16.63.255 scope global eth0
       valid_lft forever preferred_lft forever
```

### link
```
root@nenuoj$ ip link show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
```

### route
```
root@nenuoj$ ip route
default via 172.16.63.253 dev eth0
172.16.0.0/18 dev eth0  proto kernel  scope link  src 172.16.62.130
```

- - -

## netstat
https://linux.die.net/man/8/netstat
> netstat - Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships

### List all tcp listening ports
```
root@nenuoj$ netstat -tl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 localhost:32000         *:*                     LISTEN
tcp        0      0 *:27015                 *:*                     LISTEN
tcp        0      0 localhost:mysql         *:*                     LISTEN
tcp        0      0 localhost:11211         *:*                     LISTEN
tcp        0      0 localhost:6379          *:*                     LISTEN
tcp        0      0 *:http                  *:*                     LISTEN
tcp        0      0 *:ssh                   *:*                     LISTEN
tcp        0      0 *:https                 *:*                     LISTEN
tcp        0      0 localhost:1087          *:*                     LISTEN
tcp6       0      0 [::]:http               [::]:*                  LISTEN
```

### List all tcp connections with pid and numericals
```
root@nenuoj$ netstat -atnp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:32000         0.0.0.0:*               LISTEN      972/java
tcp        0      0 0.0.0.0:27015           0.0.0.0:*               LISTEN      1323/Judger
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      1203/mysqld
tcp        0      0 127.0.0.1:11211         0.0.0.0:*               LISTEN      1278/memcached
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      808/redis-server 12
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1270/nginx -g daemo
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1044/sshd
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      1270/nginx -g daemo
tcp        0      0 127.0.0.1:1087          0.0.0.0:*               LISTEN      430/privoxy
tcp        0      0 172.16.62.130:38632     100.100.30.25:80        ESTABLISHED 1403/AliYunDun
tcp        0      0 172.16.62.130:80        182.122.161.236:52680   FIN_WAIT2   -
tcp        0      0 172.16.62.130:80        182.122.161.236:52669   FIN_WAIT2   -
tcp        0      0 172.16.62.130:80        182.122.161.236:52686   FIN_WAIT2   -
tcp        0      0 127.0.0.1:32000         127.0.0.1:31000         ESTABLISHED 933/wrapper
tcp        0      0 172.16.62.130:80        182.122.161.236:52681   FIN_WAIT2   -
tcp        0      0 172.16.62.130:80        182.122.161.236:52679   FIN_WAIT2   -
tcp        0      0 127.0.0.1:31000         127.0.0.1:32000         ESTABLISHED 972/java
tcp        0    560 172.16.62.130:22        120.52.147.54:49840     ESTABLISHED 4232/1
tcp        0      0 172.16.62.130:80        182.122.161.236:52682   FIN_WAIT2   -
tcp        0      0 172.16.62.130:80        182.122.161.236:52668   FIN_WAIT2   -
tcp6       0      0 :::80                   :::*                    LISTEN      1270/nginx -g daemo
```

### Display routing
```
root@nenuoj$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         172.16.63.253   0.0.0.0         UG        0 0          0 eth0
172.16.0.0      *               255.255.192.0   U         0 0          0 eth0
```

### Show interface infomation
```
root@nenuoj$ netstat -i
Kernel Interface table
Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
eth0       1500 0     40465      0      0 0         33318      0      0      0 BMRU
lo        65536 0     10406      0      0 0         10406      0      0      0 LRU
```