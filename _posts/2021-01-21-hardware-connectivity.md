---
title: "Hardware connectivity"
author: bart
date: 2021-01-21 10:00:00 +0100
categories: [Hardware, Tutorial, Tech]
tags: [Hardware, Tutorial, Tech, Wheatley]

redirect_from:
  - /tech/wheatley/hardware-connectivity/

published: true
toc: true

img_path: /assets/img/2021-01-21-hardware-connectivity
image:
  path: /NewDC.png
---

The [hardware is now in the datacenter]({% post_url 2021-01-15-new-hardware %}) but there are various ways that the servers could be connected. This means planning the connections between the servers, from the server to the internet, and connecting power. In this blog, I’ll take you along in the design that I made on how to connect the servers.

## The nuts and bolts
I love to make drawings in Microsoft Visio so I’ve created an overview of how the servers need to be connected. I created the overview before I went to the datacentre and configured the ports accordingly. Each server has four Network Interface Cards (NICs) that need to provide all connectivity for the workloads. The four NICs have either one or multiple ports and in total, a server has eight ports at 1 Gbps, two ports at 10 Gbps, and one 1 Gbps connection for remote management. During the installation I forgot the take a photo of the backside of the server, so that photo will come once I went to the datacentre again.

![Datacenter connectivity](/NewDC.png)

On the servers, I’ve got VMware ESXi installed which gives the possibility to separate different types of traffic. With the right configuration it’s possible to separate management traffic, vMotion, vSAN, vSAN Witness, and a [few more](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.networking.doc/GUID-D4191320-209E-4CB5-A709-C8741E713348.html). Most of the network traffic categories are not in use in the lab but I do want to separate management and vMotion traffic. VMware vMotion is a technique that makes it possible to relocate a workload from one server to the other. Relocating requires a massive bandwidth as it needs to transmit the complete workload -active memory and hard disk- to the other server. Management of the ESXi has a really low bandwidth requirement but is absolutely essential and need to be guaranteed.

## The cables
Within VMware, there is a mechanism to help separate and prioritize the network traffic called Network I/O Control (NIOC). However, these servers have loads of network ports, so why not separate them physically? I’ve connected both ports of the X520-DA2 between the servers (<span style="color:##ab9ac0">purple</span>). This gives a theoretical throughput of 20 Gbps between the servers for workload and vMotion network traffic. Additionally, two of the onboard NICs are also connected between the servers (<span style="color:##9dbb61">green</span>) and they are used for all the management network traffic. This brings the total number of connections between the servers to four, at a total bandwidth of 22 Gbps.

Each server has its own up-link to the Internet Service Provider (ISP) via a 1 Gbps connection (<span style="color:##a30000">red</span>). Both servers have an integrated Dell Remote Access Controller (iDRAC), that needs protection. The iDRAC provides the possibility to manage the server remotely as if you’re standing next to the server with a monitor, keyboard, and mouse connected. Via the iDRAC, it’s also possible to remotely reinstall the server or give the server a reboot when the Operating System crashes. The iDRAC has a dedicated NIC that connects to a second, separate, 1 Gbps connection protected by the ISP (also <span style="color:##a30000">red</span>).

With the network connections sorted it’s time to think about the power. Luckily this is quite simple: the servers each have two Power Supply Units (PSUs) that connect to the power of the datacentre. The datacentre provides redundant power ‘feeds’, so both servers have one PSU connected to power ‘feed A’ and the other PSU connected to power ‘feed B’. When to power for feed A is lost during an outage, there is no impact on the servers because the power remains on power feed B.

## Closing thought
Thanks for reading. Hopefully, you found it interesting and maybe even learned something new! Interested in being informed about new posts? You can subscribe to new posts at the bottom of the page! Any questions or just want to leave a remark? You can do that below, I’m very curious what you think of the content. Enjoy your day!