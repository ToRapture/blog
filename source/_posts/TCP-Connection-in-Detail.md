---
title: TCP Connection in Detail
date: 2020-05-12 17:48:00
mathjax: true
tags:
- Network
- TCP/IP
---

# Reset
In general, a reset is sent by TCP whenever a segment arrives that does not appear to be correct for the referenced connection.
The reset segment elicits no response from the other end—it is not acknowledged at all.
The receiver of the reset aborts the connection and advises the application that the connection was reset. This often results in the error indication “Connection reset by peer” or a similar message.

# TIME-WAIT
http://www.serverframework.com/asynchronousevents/2011/01/time-wait-and-its-design-implications-for-protocols-and-scalable-servers.html

https://stackoverflow.com/a/60075710/13133551
> The two reasons for the existence of the TIME-WAIT state and the 2SML timer:
> * If the last ACK segment is lost, the server TCP, which sets a timer for the last FIN (Finish) bit set, assumes that its FIN is lost and resends it. If the client goes to the CLOSED state and closes the connection before the 2MSL timer expires, it never receives this resent FIN segment, and consequently, the server never receives the final ACK. The server cannot close the connection. The 2MSL timer makes the client wait for a duration that is enough time for an ACK to be lost (one SML) and a FIN to arrive (another SML). If during the TIME-WAIT state, a new FIN arrives, the client sends a new ACK and restarts the 2SML timer.
> * A duplicate segment from one connection might appear in the next one. Assume a client and a server have closed a connection. After a short period, they open a connection with the same socket addresses (same source and destination IP addresses and the same source and destination port numbers). A duplicated segment from the previous connection may arrive in this new connection and be interpreted as belonging to the new connection if there is not enough time between the two connections. To prevent this problem, TCP requires that an incarnation cannot occur unless a 2MSL amount of time has elapsed.