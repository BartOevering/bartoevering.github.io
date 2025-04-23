---
title: "Home lab, again an new version"
author: bart
date: 2023-10-25 10:00:00 +0100
categories: [Hardware, Tutorial, Tech]
tags: [Hardware, Tutorial, Tech]

redirect_from:
  - /tech/hardware/home-lab-again-an-new-version/

published: true

image:
  path: /assets/img/2023-10-25-new-home-lab/R730-Front.jpeg
---

Okay, okay I know I've just done a write-up about my home labs, and now I already start with a new version? Well, the idea for the write-up came to me since I was already thinking and working towards this new version. So this will be it! A huge upgrade for my (not so) home-lab where I'll more than double the available resources. This blog will focus on the hardware side since that's what I have. The servers are not yet fully ready to move to the datacenter, but hopefully, that'll be soon. So let's dive into the new hardware.

## Blast from the past - Version two
First a quick recap of version two. Version two consists of two Dell PowerEdge R720 servers with the following specs:

- CPU: Intel(R) Xeon(R) CPU E5-2660 v2 @ 2.20GHz (2x) 44 Ghz per ESXi host
- Memory: DDR3L ECC 10600R (1333 Mhz) (16x16gb) 256 GB per ESXi host
- Storage:
    - Controller: PERC H710P Mini
    - SSD: Samsung 850 Pro 1TB (2x per server)
    - SAS: Hitachi 900 GB SAS HUC10909 CLAR900 (900GB, 10k) (9x per server)
    - HDD: Seagate Desktop (3TB, 7200rpm) 1x per server
- Networking:
    - Intel(R) X520-DA2 (2x 10 Gbps)
    - Intel(R) Gigabit 4P I350-t (8x 1 Gbps)

![Twee Dell PowerEdge R720s - OeveringIT](/assets/img/2023-05-01-home-lab-history/Dell-PowerEdge-R720.jpeg)
_Dell PowerEdge R720 without bezels_

### Things I'd like to change for a new version

I've used these R720s for a few years now and there are a few things I'd like to address for a new version:
- [ ] When moving the servers to the DC, I used to use both 10 Gbps interfaces to get 20 Gbps for the network between the servers. To accomplish this I've added both ports as uplinks to the Distributed Virtual Switch. This however didn't work as expected since there is no real switch between the servers, and they are directly connected. This caused network traffic to sometimes not find the correct way back because it came through the other physical interface. Normally a switch would fix this, but that means paying for another 1U rack spot and that's not going to happen. I ended up just using one link between the servers and having 10 Gbps.

- [ ] The 3TB HDD was used for ISO's and for 'slow' backup/archive storage. The files on here don't require high-speed access so the slow 7200 rpm HDD worked fine. Sadly the HDD on one server stopped working completely (it can no longer be found on the server) and on the other server, it has been working intermittently for about a year now. That means that I haven't had any data on the HDDs for a while now. That got me thinking do I really need this type of storage?

- [ ] The Samsung 850 Pro 1TB SSDs that are in RAID-1 are way slower than I expected. In my R710 server (version one) I used four of these as quick storage (RAID-5) and that worked well. However, since they have been split up and in RAID-1 in the R720's they've been everything but quick. I tried using them for workload for a while but the VM on there always gave me the feeling of being slower than the main RAID-5 array, 8 times 900 GB Hitachi SAS 10k disks. Since I noticed this, I haven't been using the SSDs in the servers anymore.

- [ ] Both servers are fitted with a whopping 8(!), 1 Gbps network ports. I ended up only using one per server as this is the uplink to the provider. The provider also provides a 10 Gbps over T-BASE, so these ports are too slow and too many!

- [ ] The E5-2660 v2 CPUs are now sadly End-Of-Life (EOL) for VMware ESXi. There are no direct issues with this (they still work fine) but since I run some nested labs with NSX I require ["1G hugepage"](https://communities.vmware.com/t5/VMware-NSX-Discussions/NSX-T-3-2-on-vSphere-7u3c-Hugepage-issues-on-Edge-VMs/td-p/2894117) support to be enabled. On these CPUs that means that all memory of the VMs will be fully reserved and allocated as soon as the VM powers on. As an example, in one lab I've got 5 nested ESXi nodes with 16 RAM per node, which accumulates to a total of 80 GB of memory per lab which just eats away available memory resources. Newer CPUs will handle this better from the ESXi level and might not need to have this setting enabled.

- [ ] When I installed VMware ESXi 7.0 on these servers, it was still okay/allowed/recommended having the installation on the internal SD cards. Since version 7.0U1 VMware changed the recommendation for no longer installing ESXi on the SD cards (["VMware KB"](https://kb.vmware.com/s/article/85685)). Even in my daily job I hear about SD-cards failing regularly and also for one of my servers the SD-card has failed. Since I do have alarms about bad SD-cards in my LAB, I'm not 100% sure that both servers would reboot at the moment. Therefore, they have both not been rebooted for quite some time, and one has an uptime of over 700 days.

## On with the new! Version three
Since I work a lot with VMware and also hardware it's sometimes easy to get my hands on old (for me new) hardware. This can be because of a hardware refresh at a customer or maybe an old environment that was phased out. This way I managed to get my hands on two beautiful Dell PowerEdge R730. With the help of some spare parts and some secondhand bought parts, I managed to Frankenstein these servers to the hardware they have now. To keep the servers easy to manage I'll need to make sure they are equal on hardware. Let's take a look at the hardware per server, and then work through the updates I've made.

- CPU: Intel(R) Xeon(R) CPU E5-2698 v4 @ 2.20GHz (2x) 88 Ghz per ESXi host
- Memory: DDR4 ECC 32GB RDIMM, 2400MT/s (24x) 768 GB per ESXi host
- Storage:
  - Controller: PERC H730P Mini 2GB
  - NVMe: WD Red 500GB (2x in a PCIe to NVMe adapter)
  - SSD: Intel SSD DC S4500 1.92TB (4x)
  - SSD: Toshiba Dell 1.6TB (8x)
  - SSD: Intel SSD DC 480GB (1x on CD-drive location)
- Networking:
  - Intel(R) Dell XL710-QDA2 2x 40Gbps QSFP+ rNDC
  - Intel(R) X550-T2 10GbE (2x 10 Gbps T-BASE)
- Additional:
  - NVIDIA Tesla M10 GPU
  - 64GB SD Card with IDSDM

![Twee Dell PowerEdge R730s - OeveringIT](/assets/img/2023-10-25-new-home-lab/R730-Front.jpeg)
_Dell PowerEdge R730 front without bezels_

![Twee Dell PowerEdge R730s - OeveringIT](/assets/img/2023-10-25-new-home-lab/R730_top.jpeg)
_Dell PowerEdge R730 top view_

![Twee Dell PowerEdge R730s - OeveringIT](/assets/img/2023-10-25-new-home-lab/R730-disk1.jpeg)
_Dell PowerEdge R730 OS disk_

![Twee Dell PowerEdge R730s - OeveringIT](/assets/img/2023-10-25-new-home-lab/R730-disk2.jpeg)
_Dell PowerEdge R730 OS disk back_

![Twee Dell PowerEdge R730s - OeveringIT](/assets/img/2023-10-25-new-home-lab/R730-NVMe.jpeg)
_Dell PowerEdge R730 NVMe location_

![Twee Dell PowerEdge R730s - OeveringIT](/assets/img/2023-10-25-new-home-lab/R730-back.jpeg)
_Dell PowerEdge R730 back with the 40 Gbps NICs_

So, this is really some serious hardware! A single R730 already has more resources than **both** R720s combined! And I've got two of these bad boys! This is really some cool stuff to have as the basis of my home lab. I'm not sure what I'll do with it. For example, I have no immediate use case for the GPUs in the servers, they also require licensing which is usually not cheap to get. But other than that, the cluster will have 176 Ghz(!) with 1.536 GB (!) of RAM and just over 40 TB of raw storage, and 40 Gbps network between the server! Wauw! Let's see what I changed since I got the hardware.

### Upgrades made
- [x] Originally the R730s came with 512GB of memory each. These servers do have the slots for tripe lane memory and that left 8 memory slots open. Via, via I managed to get my hands on similar speed memory DIMMs and added them to the servers raising the memory from 512GB to 768GB per server.

- [x] Since I don't want to use the SD cards in the servers, I need a different place to install ESXi. The ESXi installation footprint is quite small, so I don't want to use an expensive, high-capacity, SSD for that. Also, I'd like to keep the drive bays in the front of the server available. I ended up molesting the CD drive bay filler from Dell to allow for the data and power cable to fit through. Then a 480GB SDD fits in there nice and snug. This 480GB SDD is plenty to hold the ESXi installation, and it's an Intel DC SDD which -hopefully- won't fail on me that quickly (or rather at all).

- [x] Say goodbye to 1 Gbps networking! From here on out it's all going to be high speed! Since I was playing around with the amount of PCI lanes available on the motherboard I ended up swapping all the NICs from the original R730s. I so badly want to upgrade my uplink to the provider to 10 Gbps! Why? Because I can! I have no immediate use-case for this but hey, the provider provides it, so why **not** use it? These Intel X550-T2 have a good reputation, and they have two 10 Gbps RJ45 ports, think that should do right?

- [x] Talking about networking. I've upgraded the connection between the servers from 10 Gbps to 40 Gbps. It was not completely needed, I also could have gone for 25 Gbps as well. However, this was available and not even that much more expensive than 25 Gbps. It took some time to find the right combination. I was looking/hoping for 100 Gbps but since most 100 Gbps PCIe 3.0 cards need a x16 slot that was not possible. Therefore, I swapped out the original *Network Daughter Card* (NDC) for the Dell XL710-QDA2 NDC providing 2 ports of 40 Gbps.

- [x] When I started with the R720s I told a friend of mine that he could host on the platform, but the service is BOYD (Bring Your Own Disk). So, I've got 4 disks in the servers that will need to be moved to the R730. These disks are running a RAID-5, therefore, I need to be able to run a combination of RAID and HBA. That's why I changed the HBA card that was in the server and upgraded it to the PERC H730p Mini. The H730p will give me both options mixed and is even on the [VMware Compatibility Guide](https://www.vmware.com/resources/compatibility/detail.php?deviceCategory=io&productid=34857&vcl=true)

- [x] To play around with some really quick storage I got a PCIe to NVMe adapter. Since it's home-lab I didn't want to spend too much on it and managed to get a very good deal on four 500GB WD Red SSDs. I found a nice PCI slot (luckily since it's two per server, it only needs to be an 8x slot) and I remembered to change the bifurcation to 4x4 on the right slot and both are working perfectly now.

- [x] The last point on the list for improvement was not per se something I'd be able to upgrade to newer, the CPU. I still need to deploy my first lab with NSX and see if the reserved memory is still required, but anyway, now there is way more memory available so bring it on!

## Closing thoughts
Thanks for reading! Hopefully, you found it interesting, and maybe you’ve even learned something new! This blog was all about the hardware, but there will also be a blog about VMware. Leave your email address down below to subscribe! Any questions or just want to leave a remark? Please do so, I’m always curious to read what you think of my content. Enjoy your day!
