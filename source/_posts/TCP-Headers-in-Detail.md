---
title: TCP Headers in Detail
date: 2020-05-12 17:19:00
tags: Network, TCP/IP
---

# Timestamp option
http://www.networksorcery.com/enp/protocol/tcp/option008.htm

## Timestamp Value (TSval). 32 bits.
This field contains the current value of the timestamp clock of the TCP sending the option.

## Timestamp Echo Reply (TSecr). 32 bits.
This field is only valid if the ACK bit is set in the TCP header. If it is valid, it echos a timestamp value that was sent by the remote TCP in the TSval field of a Timestamps option. When TSecr is not valid, its value must be zero. The TSecr value will generally be from the most recent Timestamp option that was received; however, there are exceptions that are explained below. A TCP may send the Timestamp option in an initial SYN segment (i.e., segment containing a SYN bit and no ACK bit), and may send a TSopt in other segments only if it received a TSopt in the initial SYN segment for the connection.