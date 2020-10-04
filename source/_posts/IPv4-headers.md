---
title: IPv4 headers
date: 2020-04-24 17:13:00
mathjax: true
tags:
- Network
- TCP/IP
---

From: TCP/IP Illustrated, Volume 1: The Protocols.

![](/images/posts/IPv4-headers/0.png)

### IHL
The Internet Header Length (IHL) field is the number of 32-bit words in the IPv4 header, including any options. Because this is also a 4-bit field, the IPv4 header is limited to a maximum of fifteen 32-bit words or 60 bytes.
### Total Length
The Total Length field is the total length of the IPv4 datagram in bytes.
### Protocol
The Protocol field in the IPv4 header contains a number indicating the type of data found in the payload portion of the datagram. The most common values are 17 (for UDP) and 6 (for TCP).