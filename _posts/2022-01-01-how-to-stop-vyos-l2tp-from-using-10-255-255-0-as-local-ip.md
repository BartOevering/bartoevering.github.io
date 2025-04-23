---
title: "[Archive] How to: stop VyOS L2TP from using 10.255.255.0 as local IP"
author: bart
date: 2022-01-01 10:00:00 +0100
categories: [VyOS, Tech, Archive]
tags: [VMware, Tech, xl2tpd]

redirect_from:
  - /tech/archive/how-to-stop-vyos-l2tp-from-using-10-255-255-0-as-local-ip/

published: true
toc: true

image:
  path: /assets/img/2022-01-01-vyos-xl2tpd/vpn-tunnel.jpg
---

In a previous post that I wrote about VyOS, I’ve shown that the L2TP VPN is using 10.255.255.0 as an IP address. In this blog post, I’ll dive into why this is a problem for me and how to change that.

> **Heads up**! VyOS no longer uses xl2tpd, rendering this blog post obsolete 
{: .prompt-info }

## Why is this a problem for me?

Well, I try to divide my subnets logically, at home I use 10.0.0.0/16, and anything in the datacentre uses 10.10.0.0/16. Now I thought to make it handy I’d start using 10.255.0.0/16 for any remote connection like Site-to-Site VPN, L2TP, or anything else that comes in the future. I started off with using 10.255.255.0/30 as the subnet for the Site-to-Site VPN to home and this all work till I started an L2TP VPN and then all remote connected ceased to work.

That remote connection stop working has to do with routing. The VyOS router looks in it’s routing table where to forward the packet. There are two routes in there and the winner is selected based on the ‘[metric](https://en.wikipedia.org/wiki/Metrics_(networking))‘ which defines the trustworthiness of that route.

VyOS uses xl2tpd as the software for L2TP VPN clients. So I found that xl2tpd is configured such that it uses the IP-address 10.255.255.0. This means that it’s not possible to use an IP range where that IP address is in. For the Site-to-Site VPN, I was using the IP range 10.255.255.0/24. Using that IP range breaks the routing of network traffic. And then the VyOS router no longer knows where the network traffic needs to be routed to, so it arrives at the right destination. Because I noticed this behavior I started to use the IP range 10.255.200.0/30 for the Site-to-Site VPN. But it’s not nice to have a gateway out of the IP range 10.255.10.0/24 that I use for L2TP VPN clients. This simply is an unwanted configuration and therefore must change!

```bash
C:\Users\Bart>tracert -d 8.8.8.8
Tracing route to 8.8.8.8 over a maximum of 30 hops
  1    51 ms    37 ms    49 ms  10.255.255.0
  <<< output omitted >>>
  9    50 ms    44 ms    37 ms  8.8.8.8

Trace complete.
```

## Now, how to fix this?
So I started searching around and the beauty of VyOS is, that it’s still a Linux distribution, Debian. But the search online didn’t give many results. However, I found out that VyOS uses the Linux application ‘xl2tpd’. This is a term that I also found in the VyOS logs and I started digging deeper. Sadly without much result, so I started to dig into VyOS. Many applications on Linux install in /etc/ and xl2tpd is, luckily, no exception.

```bash
EU-GW04:/home/vyos# ls -al /etc/xl2tpd/
total 13
drwxr-xr-x 1 root root 4096 Jan  4 13:51 .
drwxr-xr-x 1 root root 4096 Jan  4 13:51 ..
-rw------- 1 root root  109 May 22  2019 l2tp-secrets
-rw-r--r-- 1 root root  306 Jan  4 19:31 xl2tpd.conf
```

The folder `/etc/xl2tpd` just contains two files and `xl2tpd.conf` sounds to make sense. I like `nano` a bit more than `vi` and `nano` works great on VyOS. So I used `nano` to open the file.

```bash
;### VyOS L2TP VPN Begin ###
[global]
listen-addr = XX.XX.XX.192

[lns default]
ip range = 10.255.10.10-10.255.10.254
local ip = 10.255.255.0
refuse pap = yes
require authentication = yes
name = VyOSL2TPServer
ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
;### VyOS L2TP VPN End ###
```

Okay, okay, this looks good. There is a **local IP** configured here with the value 10.255.255.0. Would it be possible to just change it there? Would it be so simple? Yes, it is. I’ve updated the local IP to 10.255.10.1 which is more logical and restarted xl2tpd. Connected to the VPN and, it works as I want! Awesome!

```bash
C:\Users\Bart>tracert -d 8.8.8.8
Tracing route to 8.8.8.8 over a maximum of 30 hops
  1    72 ms    42 ms    38 ms  10.255.10.1
  <<< output omitted >>>
  9    89 ms    48 ms    62 ms  8.8.8.8

Trace complete.
```

## Closing thought
Now the IP use makes more sense. At some point, I might even change the Site-to-Site VPN to use 10.255.255.0/24 once again. But for now, I’m pretty happy with the current configuration.

Thanks for reading, hopefully, you found it interesting and maybe even learned something new! Want to be informed about a new post? Subscribe! Any questions or just want to leave a remark? Please do, I’m very curious what you think of the content. Enjoy your day!