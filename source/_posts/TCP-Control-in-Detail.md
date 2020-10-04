---
title: TCP Control in Detail
date: 2020-05-12 18:30:00
mathjax: true
tags:
- Network
- TCP/IP
---

# Retransmission
TCP sets a timer when it sends data, and if the data is not acknowledged when the timer expires, a timeout or timer-based retransmission of data occurs. The timeout occurs after an interval called the retransmission timeout (RTO). It has another way of initiating a retransmission called fast retransmission or fast retransmit, which usually happens without any delay. Fast retransmit is based on inferring losses by noticing when TCP’s cumulative acknowledgment fails to advance in the ACKs received over time, or when ACKs carrying selective acknowledgment information (SACKs) indicate that out-of-order segments are present at the receiver. Generally speaking, when the sender believes that the receiver might be missing some data, a choice needs to be made between sending new (unsent) data and retransmitting.

## Fast retransmit
![](/images/posts/TCP-Control-in-Detail/0.png)

# Flow Control and Window Management
![](/images/posts/TCP-Control-in-Detail/1.png)
![](/images/posts/TCP-Control-in-Detail/2.png)