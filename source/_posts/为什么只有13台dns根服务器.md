title: 为什么只有13台dns根服务器
date: 2014-12-23 17:17:53
author: 赵空暖
tags: 
- 读书有疑
categories: 
- 读书有疑
---

Fitting the DNS Server List Into a Single IP Packet

Because DNS operation relies on potentially millions of other Internet servers finding the root servers at any time, the addresses for root servers must be distributable over IP as efficiently as possible. Ideally, all of these IP addresses should fit into a single packet (datagram) to avoid the overhead of sending multiple messages between servers. In the IP version 4 (IPv4) prevalent today, the DNS data that can fit inside a single packet is as small as 512 bytes (after subtracting all of the other protocol supporting information contained in packets). Each IPv4 address requires 32 bytes. Accordingly, the designers of DNS have chosen 13 as the number of root servers for IPv4, taking 416 bytes of a packet and leaving up to 96 bytes for other supporting data (and flexibility to add a few more DNS root servers in the future if needed).