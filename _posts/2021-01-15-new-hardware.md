---
title: New hardware
author: bart
date: 2021-01-15 10:00:00 +0100
categories: [Hardware, Tutorial, Tech]
tags: [Hardware, VMware, Tech, Wheatley]

redirect_from:
  - /tech/hardware/new-hardware/

published: true
img_path: /assets/img/2021-01-15-new-hardware

image:
  path: /assets/img/2021-01-15-new-hardware/01_servers_in_the_rack.jpg
  alt: New servers in the rack!
---

On the 24th of September 2017, I’ve placed a single Dell PowerEdge R710 in the XS4ALL datacentre in Amsterdam. Now, three years later, it finally came to an upgrade. I already bought two newer servers (two Dell PowerEdge R720’s) and had them laying around since the end of 2017. Sadly, I never found the money or use-case to swap the R710 for the pair.

In May 2020 I decided that enough is enough! These beauties belong in a nice server rack and should be powered on all the time, not catching dust in my living room- more powered off then on. I decided to contact a good friend of mine and persuaded him to ditch the server he had at home and join me so that I can host both servers in the datacentre. This worked out well and we arranged it such that 1) my lab wouldn’t cost me that much and 2) he would also have access to a nice lab environment 🙂

After this decision to get the two R720’s into the datacentre, time went by so quickly. Now the R720 servers are in use and they are working awesome! The servers are co-located in DC2 of XS4ALL and both servers have access to a 1 Gbps link to the XS4ALL network. In the datacentre, I have a Service Level Agreement (SLA) that has a fair use policy for power and network traffic. I noticed that I can stash quite some traffic before they start asking questions.

![The new servers in the rack](/assets/img/2021-01-15-new-hardware/01_servers_in_the_rack.jpg)

It was quite tricky though, I needed to migrate the VM’s from Citrix XenServer, the hypervisor I had running on the single R710, to VMware ESXi. Luckily, I already used my R710 with Citrix XenServer in a customer case to test migrate from XenServer to ESXi. In that project, I was able to automate the migration and I was able to re-use parts of the code for the migration (I’ll see whether I can share some code snippets in a future blog).

## The Specs
![Available resources in the cluster](/assets/img/2021-01-15-new-hardware/02_resources_available_in_vCenter.png)

Now the interesting part: what hardware am I running in those R720’s? Well, both ESXi hosts have roughly the same hardware, except one server has additional disks as this is the arrangement that I made with the earlier-mentioned friend. So what hardware are we talking about?

- CPU: Intel(R) Xeon(R) CPU E5-2660 v2 @ 2.20GHz (2x) 44 Ghz ESXi host
- Memory: DDR3L ECC 10600R (1333 Mhz) (16x16gb) 256 GB per ESXi host
- Storage:
    - Controller: PERC H710P Mini
    - SSD: Samsung 850 Pro 1TB (2x per server)
    - SAS: Hitachi 900 GB SAS HUC10909 CLAR900 (900GB, 10k) (9x per server)
    - HDD: Seagate Barracuda Compute (5TB, 5400rpm) 4x only ESXi04)
    - HDD: Seagate Desktop (3TB, 7200rpm) 1x per server
- Networking:
    - Intel(R) X520-DA2 (2x 10 Gbps)
    - Intel(R) Gigabit 4P I350-t (8x 1 Gbps)

At this moment, sadly, my hoster doesn’t provide a 10 Gbps uplink port. So that’s why the servers now have their own 1 Gbps uplink. I do keep one of the 10 Gbps ports free, for possible future use.

I’ll end this blog with one of my favorite quotes that I just need to share with you. Thanks for hanging on till the end of this blogpost. In my future posts, I hope to show you what cool stuff I have running in my lab environment. Leave a comment if you have any questions or if you liked this blog and I’ll get back to you.

> The most elementary and valuable statement in science, the beginning of wisdom, is “I do not know”.
> (Berman, 1987)