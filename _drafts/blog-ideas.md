---
author: bart
date: 3500-06-01 10:00:00 +0100
published: false
---


## VCD Three persona (reseller level)
- [ ] Efect on NSX-T
- [ ] API calls
- [ ] UI changes

## NSX-T bridging without NSX-T managers
- [ ] Is briding failing when the NSX-T managers are not available?


## VMware port best practice
- [ ] Ephemeral vs Static Binding

https://kb.vmware.com/s/article/1022312


##  Docker-swarm overlay

Not running because of VMware NSX Port
https://stackoverflow.com/questions/43933143/docker-swarm-overlay-network-is-not-working-for-containers-in-different-hosts


## VMware vCloud Director - Deploying without reading
- [ ] Two nics?
    - [ ] 1 for all traffic
    - [ ] 1 for DB replication
    https://docs.vmware.com/en/VMware-Cloud-Director/10.4/VMware-Cloud-Director-Install-Configure-Upgrade-Guide/GUID-3BBE2901-6252-4DB7-970F-94901E7453D2.html
- [ ] Error 1: No search domain
- [ ] Error 2: No reverse DNS
    - [ ] Host records (A) need to be created for VCD http and VCD VMRC incl reverse lookup
    - [ ] All other DNS records for vCenter, ESXi hosts and NSX
- [ ] Error 3: Anything else?

## VMware vCloud Director - Adding NSX-T mananager just doesn't work

- [ ] Deployment of NSX manager and impact SSL certificate
- [ ] NSX-manager SSL certificate and impoved SSL check VCD
- [ ] Fix

## How to: Fix Dell iDRAC 8 'No Signal'

To read: 
- [ ] [Dell Community topic - Page 3](https://www.dell.com/community/PowerEdge-Hardware-General/Virtual-console-not-working-after-iDRAC-update-to-2-70-70-70/td-p/7518493/page/3)
- [ ] [Dell Community topic](https://www.dell.com/community/PowerEdge-Hardware-General/Virtual-console-not-working-after-iDRAC-update-to-2-70-70-70/td-p/7518493)

## Change the IP of a VMware vSAN node

Stupid idea, if done the wrong way, Sort of lessons learned! Don't do it, but we're going to do it anyway!

Change the IP-address via the GUI and all will stop working! Would be better to re-ip by adding a vmk port and switch over traffic
https://kb.vmware.com/s/article/76162

But we're going to break it and learn!
Change the IP-address via vCenter (you might want to shutdown some import VMs first?)


```shell
esxcli vsan cluster get
esxcli vsan network list
esxcli vsan health summary list
esxcli vsan debug object health summary get
```

https://developer.vmware.com/docs/6676/vsphere-command-line-interface-reference/doc/esxcli_vsan.html

https://kb.vmware.com/s/article/2150303

Reboot some VMs via esxcli
https://kb.vmware.com/s/article/1038043

This KB was needed because vCenter didn't got shutdown
https://kb.vmware.com/s/article/2081464





## More Ideas
- [ ] VMware, VyOS migrating from many LAN NIC's to trunk
- [ ] VMware vSphere Mobile Client
- [ ] VMware NSX, what is it and why do I post about it?
- [ ] Explaining IP-addresses and ranges
- [ ] What is 'Defence in Depth'?
- [ ] VyOS ECMP S2S with home
- [ ] Rebuild Site-to-Site to GRE with IPSEC