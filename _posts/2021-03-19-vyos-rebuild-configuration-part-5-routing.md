---
title: "VyOS rebuild configuration – part 5 – Routing"
author: bart
date: 2021-03-19 10:00:00 +0100
categories: [VyOS, Tutorial, Tech]
tags: [VyOS, Tutorial, Tech]

redirect_from:
  - /tech/vyos/vyos-rebuild-configuration-part-4-remote-connections/

published: true

img_path: /assets/img/2021-03-15-vyos-rebuild/
image:
  path: traffic-routing.jpg
---
Welcome to the 5th and final part of my blog series about the network configuration that I have built for my VyOS routers! In part 1 I gave a design overview followed by part 2 with the firewall, DHCP, DNS, and NTP configuration. In the third part, I configured the interfaces and added a feature called VRRP. Then, in part 4 I made sure that I can connect from anywhere in the world to this network and created a Site-to-Site tunnel with my home network, and in this last part, I’ll be configuring network routing.

Interested in the complete story and configuration? You can find all parts of this blog series listed here once they are online:
- [VyOS rebuild configuration - part 1 - design overview]({% post_url 2021-03-15-vyos-rebuild-configuration-part-1-design-overview %})
- [VyOS rebuild configuration - part 2 - Firewall and Services]({% post_url 2021-03-16-vyos-rebuild-configuration-part-2-firewall-and-services %})
- [VyOS rebuild configuration - part 3 - Interface configuration]({% post_url 2021-03-17-vyos-rebuild-configuration-part-3-interface-configuration %})
- [VyOS rebuild configuration - part 4 - Remote connections]({% post_url 2021-03-18-vyos-rebuild-configuration-part-4-remote-connections %})
- [VyOS rebuild configuration - part 5 - Routing]({% post_url 2021-03-19-vyos-rebuild-configuration-part-5-routing %})

## *Dynamic Routing*
With all the connections from the previous blogs set up, someone needs to know how to reach all the different IP-addresses that are active within the network. The VyOS routers are at the core of my network and they will learn the routes to all the different networks. For the future, I’m planning to play around more with products of VMware, for example, VMware NSX-T and link those networks to the VyOS routers. What I try to prevent is that every time I add something new to the network, I need to login to the VyOS routers and tell them manually how they can reach an IP-range. Also, dynamic routing is widely used in companies of all sizes, so I see the benefit of having some additional hands-on experience. For these reasons, I want to dive into the world of dynamic routing.

There are a few ways of achieving dynamic routing; one of the most likely options is to use dynamic routing protocols like RIPv2, OSPF, or BGP. Before VMware NSX-T version 3.1.1 it was only possible to use BGP as a routing protocol, now with the release is VMware NSX-T 3.1.1. it’s possible to use both OSPF or BGP. I’ve chosen to connect anything that has a connection over the internet via BGP and when I’m ready to deploy VMware NSX I’ll make the decision if I want to use OSPF or BGP.

BGP stands for the Border Gateway Protocol which allows routers to exchange routing information about the infrastructure. BGP routers will tell each other which IP-ranges they have connected to them and this way other routers learn where which IP-range can be reached. I started configuring BGP but my home router decided to start announcing a few too many IP-ranges via BGP. My Ubiquiti USG decided that it would also be fun to announce its own public IP-range via BGP and therefore the VyOS router became unreachable from my home -OEPS.

![facepalm](facepalm.png)

This showed me that BGP was working as it should but now I lost access to the VyOS routers. I turned off BGP on the Ubiquiti USG and gained full access again but there is a lesson to be learned here, I needed to protect the BGP of my VyOS routers from having any unwanted routes injected. In the future, I might want to connect more locations to the VyOS routers therefore I’m trying to prevent having a big dependency on the BGP configuration of any remote location like my home. I then set up import and export filters on both VyOS routers which make sure that only allowed IP-ranges are placed in the routing table. All other IP-ranges will not be added to the routing table of the VyOS routes and thus protecting them. Since, at this moment, I only connect my home network to the VyOS routers, I made a filter that only allows 10.0.xx.xx/16 to be received and deny all other IP-ranges.

```shell
set policy prefix-list bgp-core-out rule 10 action 'permit'
set policy prefix-list bgp-core-out rule 10 le '16'
set policy prefix-list bgp-core-out rule 10 prefix '10.10.0.0/16'
set policy prefix-list bgp-core-out rule 11 action 'permit'
set policy prefix-list bgp-core-out rule 11 description 'Quad zero'
set policy prefix-list bgp-core-out rule 11 le '0'
set policy prefix-list bgp-core-out rule 11 prefix '0.0.0.0/0'
set policy prefix-list bgp-core-from-home-in rule 10 action 'permit'
set policy prefix-list bgp-core-from-home-in rule 10 description 'Home IP-range'
set policy prefix-list bgp-core-from-home-in rule 10 prefix '10.0.0.0/16'
set policy route-map bgp-core-out rule 10 action 'permit'
set policy route-map bgp-core-out rule 10 match ip address prefix-list 'bgp-core-out'
set policy route-map bgp-core-out rule 10 set
set policy route-map bgp-core-from-home-in rule 10 action 'permit'
set policy route-map bgp-core-from-home-in rule 10 match ip address prefix-list 'bgp-core-from-home-in'
set protocols bgp 65500 address-family ipv4-unicast network 10.10.0.0/16
set protocols bgp 65500 address-family ipv4-unicast redistribute connected
set protocols bgp 65500 neighbor 10.255.200.2 address-family ipv4-unicast route-map export 'bgp-core-out'
set protocols bgp 65500 neighbor 10.255.200.2 address-family ipv4-unicast route-map import 'bgp-core-from-home-in'
set protocols bgp 65500 neighbor 10.255.200.2 address-family ipv4-unicast soft-reconfiguration inbound
set protocols bgp 65500 neighbor 10.255.200.2 remote-as '65510'
set protocols bgp 65500 neighbor 10.255.200.2 update-source '10.255.200.1'
```

The VyOS routers don’t need a BGP peering between themselves, because the VyOS routers are working together as master/slave and only one will be active at any moment. BGP works with an *Autonomous System Numbers* (ASN) that tells the neighbour what path it needs to take to reach an IP-range. The ASN is a 16-bit number ranging from 1 to 65.535 where 0-64495 are reserved for public use. For private use, it’s allowed to use 64.512 till 65.534 and I’ve decided that the VyOS routers will use ASN 65500 and my home will use ASN 65510.

The VyOS routers are configured in a master/slave set up and that’s why there is no need for setting up a second, redundant, BGP session with the slave VyOS router. When the master VyOS router fails the Site-to-Site tunnel automatically fails over to the slave and a new BGP session is built. This prevents the need for any advanced BGP settings like *[‘local-preference‘](https://networklessons.com/bgp/how-to-configure-bgp-local-preference-attribute)* or to prepend the ASN-path to steer the network traffic toward the desired path. All network traffic will always flow to the master and from there will be routes to the right destination.

For BGP to be able to exchange routing information via a [*Transmission Control Protocol*](https://nl.wikipedia.org/wiki/Transmission_Control_Protocol) (TCP) session that is set up between two BGP-able routers. TCP makes the session reliable because the protocol works with an acknowledgment of received packets and when acknowledgments are missed TCP will resend the packet. To built the TCP session the VyOS routers need an IP-address to set up the TCP session. On both VyOS routers, I’m using the same IP-address -10.255.200.1/30- for the *Virtual Tunnel Interface* (VTI). Now when the master router fails, the slave can take over and a new BGP session can be set up without any change to BGP configuration. It also prevents the need of having multiple BGP configurations and my home router is always having a peering with 10.255.200.1. Which VyOS router that then is depends as this could be either EU-GW03 or EU-GW04 depending on who the master is.

With the configuration loaded in the VyOS and Ubiquiti USG router, I can clearly see that the VyOS router and the Ubiquiti USG see each other as neighbours and that they are exchanging routing information. Now I got excited en started pinging VMs and trying to access vCenter and all is working and I’m very happy with the result! It took some blood sweat and tears but for now, job well done.

```shell
UBNTUSG01:~$ show ip route bgp
Codes: K - kernel route, C - connected, S - static, R - RIP, O - OSPF,
       I - ISIS, B - BGP, > - selected route, * - FIB route

B>* 10.10.0.0/16 [20/0] via 10.255.200.1, vti65, 06:37:04

UBNTUSG01:~$ show ip bgp neighbors 10.255.200.1 received-routes
BGP table version is 0, local router ID is 10.255.200.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, R Removed
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 10.10.0.0/16     10.255.200.1             0             0 65500 i

EU-GW04:~$ show ip route bgp
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route, q - queued route, r - rejected route

B>* 10.0.0.0/16 [20/0] via 10.255.200.2, vti64, 06:37:54

EU-GW04:~$  show ip bgp neighbors 10.255.200.2 received-routes
BGP table version is 0, local router ID is 172.16.23.2, vrf id 0
Default local pref 100, local AS 65500
Status codes:  s suppressed, d damped, h history, * valid, > best, = multipath,
               i internal, r RIB-failure, S Stale, R Removed
Nexthop codes: @NNN nexthop's vrf id, < announce-nh-self
Origin codes:  i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 10.0.0.0/16      10.255.200.2                           0 65510 i
<<< output omitted >>> 
*> XX.XX.XX.XX/23  10.255.200.2             1             0 65510 ?
*> 192.168.1.0/24   10.255.200.2             1             0 65510 ?
```

## Closing thought
And that’s it! There are some more small configurations that I made like *Network Address Translation* (NAT) and user authentication with an ssh-key but they are quite basic so they didn’t make it into this blog series. If you’re interested in the full configuration or have any questions, please reach out to me at info@bartoevering.nl or if you joined the VyOS slack you can send me a DM or tag me in any post.

Just keep in mind that there’s always room for improvement, but now I have at least one VyOS router fully working and I’ll be re-installing the EU-GW03 router. I’m very happy with the current configuration but knowing myself it’ll change over time.

Thanks for following this blog series. Hopefully, you found it interesting and maybe you’ve even learned something new! Want to be informed about a new post? Subscribe! Any questions or just want to leave a remark? Please do so, I’m always very curious to hear what you think of my content. Enjoy your day!