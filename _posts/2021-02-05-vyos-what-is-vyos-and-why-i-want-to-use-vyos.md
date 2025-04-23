---
title: "VyOS, what is it? And why do I want to use VyOS!"
author: bart
date: 2021-02-05 10:00:00 +0100
categories: [VyOS, Tutorial, Tech]
tags: [VyOS, Tutorial, Tech, Wheatley]

redirect_from:
  - /tech/vyos/vyos-what-is-vyos-and-why-i-want-to-use-vyos/

published: true
toc: true

img_path: /assets/img/2021-02-05-why-vyos
image:
  path: /vyos-logo.webp
  alt: VyOS logo
---

This blog will be about VyOS but, just to make sure: I’m in no way sponsored by VyOS. I just like to use their software a lot as it’s flexible and easy to configure.

## So, what is this VyOS you speak of?

> VyOS is a Linux-based network operating system that provides software-based network routing, firewall, and VPN functionality.
_[VyOS Docs](https://docs.vyos.io/en/latest/introducing/history.html)_

In short, VyOS is an *Operating System* (OS) that functions comparable to an enterprise router. Nowadays it’s even possible to buy a hardware device and load VyOS on there. The hardware device then becomes a router, just as if you would have your conventional home or enterprise router. However, I use VyOS exclusively as a virtual machine, in my lab environment. The VyOS OS is originally ‘forked off’ the GPL portions of the Vyatta Core in 2013. Brocade, a big manufacturer of networking hardware, bought Vyatta in 2013 and they stopped the public development of the Vyatta Core. The VyOS community continued to develop the Vyatta Core into VyOS from that point on. In November 2017 VyOS version 1.1.8 was released and then the project went silent for a while. Development of VyOS remained silent until the end of 2018 when development started again to release version 1.2.0 by the end of January 2019.

From January 2019 on there’s been a steady stream of new features, enhancements, and bug fixes. One of the new features is that it’s nowadays even possible to get a *Long Term Support* (LTS) version of VyOS. The LTS version provides the possibility to get support for when something breaks or is not working as expected. VyOS support can even help you out if a specific configuration is needed or provide knowledge on how to build a good network configuration. For a few months now you can even become a [VyOS-Certified Network Engineer](https://vyos.io/certification/). All of these improvements made VyOS massively more popular in the networking community. I also noticed this during a course on VMware NSX-T 3.0, where they used the VyOS router in the [Hands-On labs](https://www.vmware.com/try-vmware/try-hands-on-labs.html) to peer the NSX-T BGP sessions with.

By the way, VyOS was not the only initiative that uses the Vyatta Core as the basis for their router. A quite big company in the networking world also uses a fork of the Vyatta Core -that is Ubiquiti. Ubiquiti created their own ‘EdgeOS’ which is based on the same Vyatta Core version as VyOS. Both initiatives have continued developing their own software in-house and now their differences in configuration start to show. However, in many ways, both routers are still quite similar.

## Why do I use VyOS?

VyOS, a software-based firewall and router -what would be the use for that? Well, I have some workloads that don’t need to be connected directly to the internet. For example, I use *VMware vRealize Network Insight* (vRNI) to monitor the network traffic in the lab. VMware vRNI gives insight into network traffic and intelligently stores metrics about all monitored network traffic. For a potential hacker, knowledge about how the network is built and which communication is allowed is extremely valuable. VMware vRNI, therefore, needs to be protected from any unauthorized access. The first step is to block all access from the internet to vRNI and only allowed access from protected networks, this could be a jumphost or via a *Virtual Private Network* (VPN) connection to the VyOS router. The VPN connection allows access to the protected workloads in the lab and also provides secure access to the internet for when I’m not at home.

When I’m at home, I have a [Ubiquiti USG](https://www.ui.com/unifi-routing/usg/) router that has a permanent Site-to-Site IPSEC VPN connection to link the VyOS router and the USG with each other. This allows communication from the lab to my home and the other way around. Again, this is handy for management and is always nice for when I’m playing around with stuff like home automation. My home network is therefore an extension of the lab network that I have in the datacentre.

![HLD of infrastructure](/NewDC_Infra.png)
_HLD overview of infrastructure_

At this moment I’m still working on the best configuration for VyOS; there’s literally a world of possible configurations. I do have a working configuration based on a single VyOS router but there’s always room for improvements and I’d like to expand redundancy with a second VyOS router and have them work together. This, for example, could be with the use of the Virtual Router Redundancy Protocol (VRRP) that creates the possibility to configure the routes such that they take over each other’s tasks. For example, if one router fails or is turned off the second one will take over the tasks of the first. This could be handy for when I take one ESXi host offline for maintenance or to upgrade the VyOS router to a new version. The configuration needs tweaking anyway -I have a remote-VPN possibility at the moment but it’s not yet working as I want since it doesn’t provide internet access right now.

When I take a quick look at the VyOS [roadmap](https://vyos.io/roadmap/) there are some awesome new features that I’m interested in testing. The features make it easier to deploy VyOS and extend my networks even broader.

- DHCPv6 Prefix Delegation
- Configuration sync
- A Network Controller for VyOS. 

With the DHCPv6 Prefix Delegation, I hope it becomes possible to assign IPv6 addresses to my home equipment from the datacenter. I’ve bridged my ISP router and I’m using the Ubiquiti USG router for all internet traffic, my ISP doesn’t support IPv6 on own bought routers yet. Because I want to have two VyOS routers, it’s a struggle to keep both configurations in sync. A feature like Configuration sync would make it possible to keep both configurations of the router in sync. This is a huge plus for maintainability and a network controller is just fun to play with. Wait, I just have the whole lab because it’s fun to play with 😀 

![Playing Santa Claus Office GIF by The Elves!](https://i.giphy.com/media/l3mZpWc5rNcnN261a/giphy.webp)