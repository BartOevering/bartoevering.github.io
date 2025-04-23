---
title: "Why did I choose to migrate to VMware?"
author: bart
date: 2021-02-19 10:00:00 +0100
categories: [VMware, Tech, Wheatley]
tags: [VMware, Tech, Wheatley]

redirect_from:
  - /tech/vyos/vyos-what-is-vyos-and-why-i-want-to-use-vyos/

published: true
toc: true

img_path: /assets/img/2021-02-19-why-vmware/
image:
  path: meme_change.jpg
  alt: Change is coming meme
---

In the blog about "[New Hardware]({% post_url 2021-01-15-new-hardware %})" I've talked about my old server running Citrix XenServer. On the new servers I changed to VMware ESXi, but why did I switch vendor?

## First off: what do Citrix and VMware even do?

![Citrix XenServer](Citrix_icon.png){: w="200" .left }
![VMware vCenter](vCenter.png){: w="200" .right }

<br /><br /><br /><br /><br /><br /><br /><br />

Both [Citrix XenServer](https://www.citrix.com/en-gb/downloads/citrix-hypervisor/) and [VMware ESXi](https://www.vmware.com/products/esxi-and-esx.html) are what we call a “hypervisor”. A hypervisor is software that allows you to create and or run Virtual Machine (VM), and that’s what we call “virtualization”. If you follow the definition of VMware virtualization means:

> Virtualization is the process of creating a software-based, or virtual, representation of something, such as virtual applications, servers, storage, and networks. It is the single most effective way to reduce IT expenses while boosting efficiency and agility for all size businesses. _[VMware 2021](https://www.vmware.com/nl/solutions/virtualization.html)_

Both Citrix XenServer and VMware ESXi make virtualization possible. The definition of VMware shows that we get virtual servers or applications, that however is still a bit abstract so let’s dive into that first. Something important to know is that there are two main ways of realizing virtualization.

# Ways of virtualization? Tell me more!

To illustrate how virtualization works, I’ve created a visualization. Below there are three examples of a server. Let’s go through them one by one. We start on the left side, here we see a traditional server where the *Operating System* (OS) runs directly on the hardware. The OS can be any OS, for example, Windows, macOS, Linux, etc. In that one OS, you run your applications- this could be anything from a web server to even your browser that you use to read this article with. If you have a very powerful server running a tiny application, the traditional approach might not the most efficient use of resources like CPU and memory. The unused resources are sitting idly waiting for anything to do but in the end, they do cost money and power.

The image in the middle is showing the way that enterprises use virtualization on their servers. This is classified as a type 1 hypervisor. In the image, the hypervisor is labeled as VMware but this could also be Citrix, KVM, Hyper-V, etc. In my lab, I also use the type 1 hypervisor. The type 1 hypervisor runs the hypervisor OS straight on top of the hardware. In that hypervisor, it’s possible to run multiple OSs. Within each OS you can run any application just as with the traditional server. With a type 1 hypervisor, the use of resources is more efficient and now we can run many OSs in parallel. The communication with the hardware is made efficient as that’s the sole task of the type 1 hypervisor OS.

![Different ways of virtualization](virtualization.png)

The far-right image shows a type 2 virtualization. We have a traditional server/workstation -this could be your laptop or computer for example. On that workstation, a normal OS is installed -like Windows or Linux. On that OS a software package is installed that allows running many OSs in parallel, this software package could for example be [VMware Workstation](https://www.vmware.com/products/workstation-pro.html) or [VirtualBox](https://www.virtualbox.org/). As the underlying OS also has the possibility to run applications, it’s really an additional layer that makes communication to the hardware less efficient. This type of hypervisor can be useful for testing purposes, running containers, or security reasons. A VM is quite easy to install or delete, it runs its own isolated OS where outside access to files is limited and it’s hard to break out of that VM -however not [completely impossible](https://en.wikipedia.org/wiki/Virtual_machine_escape).

With the type 2 hypervisor, the underlying OS also consumes resources to run its own processes. So this is approach is good for temporary solutions or using containerized workloads -more about that later- but not working for the majority of workloads in datacentres. All that extra use of resources is not needed and this must be minimized if possible. Reducing overhead of resources saves cost and allows to run more VM’s which makes everyone happy.

## So why the change from Citrix to VMware?

![Change is coming meme](meme_change.jpg)

Well, I mainly used Citrix because many options could be used for free. They even gave free access to their *Long Term Service Release* (LTSR) images which was awesome. As time went on, Citrix decided to change its licensing model. With the change in licensing model, the LTSR imaged became a paid product and the not LTSR version released a new version about every other month Next to that, my old server (Dell PowerEdge R710) had still iDRAC6 that only works via an insecure TLS 1.0 connection. TLS 1.0 is not supported any more by any browser and that made updating XenServer impossible. In the end, the server had an uptime -was not updated for- of almost 850 days, OEPS!

During a previous study, I did manage to pass my Citrix certification for Citrix XenServer. This created a base about virtualization However, in the field, I hardly come across big Citrix XenServer implementations. In 2018, after finishing my education I started working at [RedLogic](https://redlogic.nl/) which is a VMware knowledge house. This in combination with the updated license model made the love for Citrix XenServer go away and now I’ve updated my lab completely to VMware.

VMware offers a free version of ESXi that, out-of-the-box, works great. But if you want to be able to use the really cool stuff, you need to have a valid license. Therefore, in 2020 I bought myself a [VMUG Advantage Membership](https://www.vmug.com/membership/vmug-advantage-membership). The membership costs $170 a year and gives me access to VMware licenses like vCenter, ESXi. All the licenses are marked as *Not For Resale* (NFR) but as I’m using them exclusively for my lab that will not be a problem. An added bonus is that I also get access to more VMware products like NSX, VCF, vRNI, vRLI, and more cool stuff to play with.

## Is the love for Citrix now really over?

I don’t know. I mean, I liked their product in the past but that is mainly based on experience from 2010. Since then I haven’t worked with XenApp or XenDesktop although I did my graduation project for the Deltion College with a combination of XenServer and XenApp. Citrix XenApp I’ve never used since. However, Citrix XenServer I did use a lot, we had our ups and downs, but I managed to learn a lot. Although that could also have to do with the hoster where I leased the (often broken) hardware at that time. Now running XenServer just feels a bit ‘old skool’ compared to the slick HTML5 interface of VMware ESXi/vCenter. In XenServer advanced features for the virtual Distributed Switch (vDS) need to be installed with an additional VM. In VMware ESXi, it is installed in the Linux kernel of an ESXi host. This makes it a lightweight solution not taking up to many resources of the hyporvisor.

![Comparison of the XenServer UI vs vCenter UI](XenServer_vs_vCenter.png)

Because of Citrix XenServer, I’ve been playing around on the *Command Line Interface* (CLI) a lot and even managed to change a hard disk RAID-level without downtime. I probably still have the text files that I used to document what I did somewhere, it was all good fun. All this playing around really gave me an extra edge on the job. A while ago a customer came to RedLogic that wanted their Citrix VM’s to be migrated to VMware. Migrating a VM from one vendor to the other is not easy. Even with all the open standards for virtualization one cannot simply pick up a VM from Citrix and drop it on a VMware ESXi host. However, because of all the fiddling around that I did in the past, I was able to automate a great part of the migration immensely limiting the needed downtime for the customer. This resulted in a happy customer, happy boss, and therefore, happy me.

So for now, thanks Citrix for the awesome trip that we’ve had, but it’s time to start a new chapter. It might not be a farewell, who knows where our paths may cross again.

## Closing thought
Thanks for reading. Hopefully, you found it interesting and maybe even learned something new! Interested in being informed about new posts? You can subscribe to new posts at the bottom of the page! Any questions or just want to leave a remark? You can do that below, I’m very curious what you think of the content. Enjoy your day!