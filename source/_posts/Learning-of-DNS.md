---
title: Learning of DNS
date: 2020-05-03 13:37:00
mathjax: true
tags:
- Network
- DNS
---

# Headers
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200503133552717-1913887999.png)

## Lables
Each label can be up to 63 characters long, and an entire FQDN(fully qualified domain name) is limited to at most 255 (1-byte) characters.

![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200503133602883-1665474057.png)
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200503133611197-1436171352.png)

## Query Format
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200503133619966-500069655.png)

## Answer Format
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200503133627206-1901747598.png)

# Commands and Packets
```
$ dig baidu.com

; <<>> DiG 9.16.1-Ubuntu <<>> baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22302
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;baidu.com.			IN	A

;; ANSWER SECTION:
baidu.com.		5	IN	A	39.156.69.79
baidu.com.		5	IN	A	220.181.38.148

;; Query time: 3 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: 日 5月 03 13:16:51 CST 2020
;; MSG SIZE  rcvd: 70

```

![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200503133637201-588533106.png)
![](https://img2020.cnblogs.com/blog/1224734/202005/1224734-20200503133652019-1178044910.png)


# Simulation
```
$ dig +trace baidu.com

; <<>> DiG 9.16.1-Ubuntu <<>> +trace baidu.com
;; global options: +cmd
.			5	IN	NS	l.root-servers.net.
.			5	IN	NS	g.root-servers.net.
.			5	IN	NS	b.root-servers.net.
.			5	IN	NS	d.root-servers.net.
.			5	IN	NS	j.root-servers.net.
.			5	IN	NS	c.root-servers.net.
.			5	IN	NS	a.root-servers.net.
.			5	IN	NS	e.root-servers.net.
.			5	IN	NS	m.root-servers.net.
.			5	IN	NS	f.root-servers.net.
.			5	IN	NS	i.root-servers.net.
.			5	IN	NS	k.root-servers.net.
.			5	IN	NS	h.root-servers.net.
;; Received 262 bytes from 127.0.0.53#53(127.0.0.53) in 4 ms

com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			86400	IN	DS	30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
com.			86400	IN	RRSIG	DS 8 1 86400 20200515170000 20200502160000 48903 . ypoqNHa/RJ1a9uTlm4QyNZ9V//oHsA2EnziYqGpmhTGWfZm1VbAriN/R cKPWdyq03XcdJl8chkHjyYFc2O7iLNiqJVTc2zUBKyBCcSY4nqftN4J+ Xhpn8i2PZaK89YoJ0u3ptUuoNQO7r7qqFoQA38TjZmVjb0xtpRsc3Qe8 NFzkHkpGG0lE87BUcxob2xxwG/w2THCu7067TbF9s5pm1NcOIgLpLrPN qAN9gal6OWpPcoj78SLrp+WG66yJLEQ1UgZafO3JaUTTUF78HCC08GL/ YfuQQSNVAJdkdYmhdSMNEPHF3YVYu2C6RWveng0HJeDfY487jKpgsmp7 0qfo8w==
;; Received 1169 bytes from 192.5.5.241#53(f.root-servers.net) in 312 ms

baidu.com.		172800	IN	NS	ns2.baidu.com.
baidu.com.		172800	IN	NS	ns3.baidu.com.
baidu.com.		172800	IN	NS	ns4.baidu.com.
baidu.com.		172800	IN	NS	ns1.baidu.com.
baidu.com.		172800	IN	NS	ns7.baidu.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN NSEC3 1 1 0 - CK0Q1GIN43N1ARRC9OSM6QPQR81H5M9A NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN RRSIG NSEC3 8 2 86400 20200509044919 20200502033919 39844 com. TuW7Gtye26JizEnQgFR2wx0iaSNeV0B+H+NGxpMvErXVC8UgY0qikZYw Q2edAoLCg1NUX+eQe6XJxCg6fxZXgmjY53nFrvS96DVAmlzHXIMGRZz9 XLb6r1dM5a9FmjWj7/Ad8CLFU4tKaARAP1eL7XAuFDIxOquFBlr2ezsz ekU8XCfOPXoFDVBREFxTbUmR09Dca+ZHK5aQmb+22qzr6A==
HPVUNU64MJQUM37BM3VJ6O2UBJCHOS00.com. 86400 IN NSEC3 1 1 0 - HPVVN3Q5E5GOQP2QFE2LEM4SVB9C0SJ6 NS DS RRSIG
HPVUNU64MJQUM37BM3VJ6O2UBJCHOS00.com. 86400 IN RRSIG NSEC3 8 2 86400 20200508042230 20200501031230 39844 com. O1F4LRXLKjRCSFcKBKFZRjVdZ7qGVDs2J3TcPWoiCg3XEfghP2I6lA8e SGhCEbmSuLlKh7bnhLuASVnN69/y+gVUtNa16eK2rNBtIrbBorwm32Dz KzMlZHdK5g5JI73IZtMaSz/oUKdqxpCgsilK0P7guLT5e9+q2pO2ROGg XrA4o3fThVbDWzRXvyXKFX1zCyBaHs+18ko17GGb8qTWDw==
;; Received 757 bytes from 192.42.93.30#53(g.gtld-servers.net) in 104 ms

baidu.com.		600	IN	A	220.181.38.148
baidu.com.		600	IN	A	39.156.69.79
baidu.com.		86400	IN	NS	ns4.baidu.com.
baidu.com.		86400	IN	NS	dns.baidu.com.
baidu.com.		86400	IN	NS	ns7.baidu.com.
baidu.com.		86400	IN	NS	ns3.baidu.com.
baidu.com.		86400	IN	NS	ns2.baidu.com.
;; Received 240 bytes from 220.181.33.31#53(ns2.baidu.com) in 12 ms

```

```
$ dig

; <<>> DiG 9.16.1-Ubuntu <<>>
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18754
;; flags: qr rd ra; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;.				IN	NS

;; ANSWER SECTION:
.			5	IN	NS	l.root-servers.net.
.			5	IN	NS	g.root-servers.net.
.			5	IN	NS	b.root-servers.net.
.			5	IN	NS	d.root-servers.net.
.			5	IN	NS	j.root-servers.net.
.			5	IN	NS	c.root-servers.net.
.			5	IN	NS	a.root-servers.net.
.			5	IN	NS	e.root-servers.net.
.			5	IN	NS	m.root-servers.net.
.			5	IN	NS	f.root-servers.net.
.			5	IN	NS	i.root-servers.net.
.			5	IN	NS	k.root-servers.net.
.			5	IN	NS	h.root-servers.net.

;; Query time: 4 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: 日 5月 03 13:28:35 CST 2020
;; MSG SIZE  rcvd: 239

$ dig @l.root-servers.net. baidu.com

; <<>> DiG 9.16.1-Ubuntu <<>> @l.root-servers.net. baidu.com
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4979
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 13, ADDITIONAL: 27
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;baidu.com.			IN	A

;; AUTHORITY SECTION:
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.

;; ADDITIONAL SECTION:
a.gtld-servers.net.	172800	IN	A	192.5.6.30
b.gtld-servers.net.	172800	IN	A	192.33.14.30
c.gtld-servers.net.	172800	IN	A	192.26.92.30
d.gtld-servers.net.	172800	IN	A	192.31.80.30
e.gtld-servers.net.	172800	IN	A	192.12.94.30
f.gtld-servers.net.	172800	IN	A	192.35.51.30
g.gtld-servers.net.	172800	IN	A	192.42.93.30
h.gtld-servers.net.	172800	IN	A	192.54.112.30
i.gtld-servers.net.	172800	IN	A	192.43.172.30
j.gtld-servers.net.	172800	IN	A	192.48.79.30
k.gtld-servers.net.	172800	IN	A	192.52.178.30
l.gtld-servers.net.	172800	IN	A	192.41.162.30
m.gtld-servers.net.	172800	IN	A	192.55.83.30
a.gtld-servers.net.	172800	IN	AAAA	2001:503:a83e::2:30
b.gtld-servers.net.	172800	IN	AAAA	2001:503:231d::2:30
c.gtld-servers.net.	172800	IN	AAAA	2001:503:83eb::30
d.gtld-servers.net.	172800	IN	AAAA	2001:500:856e::30
e.gtld-servers.net.	172800	IN	AAAA	2001:502:1ca1::30
f.gtld-servers.net.	172800	IN	AAAA	2001:503:d414::30
g.gtld-servers.net.	172800	IN	AAAA	2001:503:eea3::30
h.gtld-servers.net.	172800	IN	AAAA	2001:502:8cc::30
i.gtld-servers.net.	172800	IN	AAAA	2001:503:39c1::30
j.gtld-servers.net.	172800	IN	AAAA	2001:502:7094::30
k.gtld-servers.net.	172800	IN	AAAA	2001:503:d2d::30
l.gtld-servers.net.	172800	IN	AAAA	2001:500:d937::30
m.gtld-servers.net.	172800	IN	AAAA	2001:501:b1f9::30

;; Query time: 8 msec
;; SERVER: 199.7.83.42#53(199.7.83.42)
;; WHEN: 日 5月 03 13:28:42 CST 2020
;; MSG SIZE  rcvd: 834

$ dig @a.gtld-servers.net. baidu.com

; <<>> DiG 9.16.1-Ubuntu <<>> @a.gtld-servers.net. baidu.com
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59027
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 5, ADDITIONAL: 6
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;baidu.com.			IN	A

;; AUTHORITY SECTION:
baidu.com.		172800	IN	NS	ns2.baidu.com.
baidu.com.		172800	IN	NS	ns3.baidu.com.
baidu.com.		172800	IN	NS	ns4.baidu.com.
baidu.com.		172800	IN	NS	ns1.baidu.com.
baidu.com.		172800	IN	NS	ns7.baidu.com.

;; ADDITIONAL SECTION:
ns2.baidu.com.		172800	IN	A	220.181.33.31
ns3.baidu.com.		172800	IN	A	112.80.248.64
ns4.baidu.com.		172800	IN	A	14.215.178.80
ns1.baidu.com.		172800	IN	A	202.108.22.220
ns7.baidu.com.		172800	IN	A	180.76.76.92

;; Query time: 292 msec
;; SERVER: 192.5.6.30#53(192.5.6.30)
;; WHEN: 日 5月 03 13:28:48 CST 2020
;; MSG SIZE  rcvd: 208

$ dig @ns2.baidu.com. baidu.com

; <<>> DiG 9.16.1-Ubuntu <<>> @ns2.baidu.com. baidu.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2415
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 5, ADDITIONAL: 6
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;baidu.com.			IN	A

;; ANSWER SECTION:
baidu.com.		600	IN	A	39.156.69.79
baidu.com.		600	IN	A	220.181.38.148

;; AUTHORITY SECTION:
baidu.com.		86400	IN	NS	ns7.baidu.com.
baidu.com.		86400	IN	NS	ns2.baidu.com.
baidu.com.		86400	IN	NS	dns.baidu.com.
baidu.com.		86400	IN	NS	ns3.baidu.com.
baidu.com.		86400	IN	NS	ns4.baidu.com.

;; ADDITIONAL SECTION:
dns.baidu.com.		86400	IN	A	202.108.22.220
ns2.baidu.com.		86400	IN	A	220.181.33.31
ns3.baidu.com.		86400	IN	A	112.80.248.64
ns4.baidu.com.		86400	IN	A	14.215.178.80
ns7.baidu.com.		86400	IN	A	180.76.76.92

;; Query time: 8 msec
;; SERVER: 220.181.33.31#53(220.181.33.31)
;; WHEN: 日 5月 03 13:28:54 CST 2020
;; MSG SIZE  rcvd: 240

```
We can see the response of `$ dig @ns2.baidu.com. baidu.com` have `aa` flag set.