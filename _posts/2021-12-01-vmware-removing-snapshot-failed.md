---
title: "VMware: Removing snapshot failed"
author: bart
date: 2021-12-01 10:00:00 +0100
categories: [VMware, Tech, Scripting]
tags: [VMware, Tech, Scripting]

redirect_from:
  - /tech/vmware/vmware-removing-snapshot-failed/

published: true
toc: true

image:
  path: /assets/img/2021-12-01-removing-snapshot/deleting-vm-snapshots.jpg
  alt: Deleting VM snapshots? It's never going to happen!
---

While working for a client I got into a situation where I was unable to remove a snapshot from a VM. This was quite annoying because the error message that was shown didn’t give me that much information. In this blog, I’ll explain the procedure that got me in this situation and what I did to finally remove the snapshot. 

## Why is a stuck snapshot a problem?
Over the time that I worked with *Virtual Machines* (VMs), it became clear that in essence a VM just consists of a [bunch of files](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vm_admin.doc/GUID-CEFF6D89-8C19-4143-8C26-4B6D6734D2CB.html) located on a storage device. Because of that, it’s easy to create point-in-time capture, a ‘snapshot’, of the VM. The snapshot captures the state, settings, virtual disk, and in most cases the memory state of the VM. From this point on, changes to the virtual disk are kept in a separate ‘delta file‘ which creates the possibility to revert to the state of the VM from when the snapshot was created. When the snapshot is no longer needed, deleting the snapshot will merge the delta file(s) with the actual virtual disk file.

The snapshot functionality provides for example a massive advantage when performing any type of upgrade in the VM *Operating System* (OS). When the upgrade ends up rendering the VM useless, there is an easy way to restore the VM back to the state of before the upgrade. Many backup solutions like [VEEAM](https://www.veeam.com/blog/why-snapshots-alone-are-not-backups.html) or [Commvault](https://documentation.commvault.com/commvault/v11/article?p=32255.htm) use the same functionality of creating a snapshot and then download and store that snapshot as a backup.

However, there is also a downside to having snapshots, when snapshots are kept on the VM there is a possible loss of performance for every snapshot. This is because each snapshot has its own delta file and the ESXi node needs to calculate the differences between those files, which gets trickier with every snapshot. The best practice is to not keep snapshots for longer than absolutely needed and remove them via the snapshot manager.

## How did I end up with a stuck snapshot?
For a client of [RedLogic](https://redlogic.nl/), we did a project where a new infrastructure was built separately from the old infrastructure. With the new infrastructure ready, around 100 VMs then needed to be migrated between the vCenter Server Appliances (VCSA), both appliances running version 6.7.0.45100 (build 17028579). For the migration, both environments had access to the same storage making it possible to easily migrate VMs between the two. For each VM there are a few steps needed to be migrated to the new environment; 
1. Power off the VM in the old environment
2. Remove the VM from the inventory in the old vCenter
3. In the new vCenter find the VM on the datastore and register the VM
4. Create a snapshot of the migrated VM
5. Upgrade the VM hardware level to the most recent
6. Power on the VM
7. If the VM boots as expected, remove the snapshot

As there were about 100 VMs needed to be migrated, this procedure would need to be done around 100 times. After a couple of VMs, I decided to be lazier and I created a script to do the job for me. I found out that there is a PowerCLI option to [connect to multiple vCenters](https://developer.vmware.com/docs/powercli/latest/vmware.vimautomation.core/commands/set-powercliconfiguration/#Default) at the same time and changed my script accordingly. I programmed the script and everything went as expected, except that there was still a snapshot left on the VM. When trying to remove it manually through vCenter it throws a very useful error; `A general system error occurred: Fault cause: vim.fault.GenericVmConfigFault`

![Operation failed](/assets/img/2021-12-01-removing-snapshot/remove-all-snapshots-failed.png)
_A general system error occurred: Fault cause: vim.fault.GenericVmConfigFault_

## How to fix this error?
After some time on Google, I found an entry from ‘*shayrulah*‘ in the VMware community and Cameron Joyce wrote a [blog](https://communities.vmware.com/t5/VMware-vSphere-Discussions/Cannot-delete-old-snapshots-of-vCenter-6-5/td-p/489902) about this topic but neither solution worked out in this case. I started combining the information from both and managed to delete the snapshot in the end. Let me take you along with what I did to finally remove the snapshots.

Heads up! I just describe here what worked for me, always make sure you have a backup!

> **Heads up!** I just describe here what worked for me, always make sure you have a backup!
{: .prompt-warning }

So, the first thing that I needed to do was to make sure that I get a maintenance window approved by the customer because the VM will need to be powered off in the process. These are the steps that I ended up taking:
1. Since we need to play around with the VM, make sure it’s powered off.
2. Now find the datastore where the `vmx` file of the VM is located
3. Remove the VM from the vCenter inventory
4. Create a temp folder in the VM folder on the datastore (I’ll do that via ssh)
5. Move all `vmsn` and `vmsd` files to tmp folder
6. Find the VMX file and add the VM back to vCenter
7. Acknowledge the ‘VM consolidation needed’ status and go drink some thee/coffee/water/anything else
8. When done, check to see if the snapshot is now removed and then boot VM.
9. VM booting alright? Continue with removing the temp folder for that VM off the datastore

I tried to reproduce this error in my lab on my vCenter 7.0 U1 but there error didn’t occur. I created the screenshots from the customer environment so I redacted quite some information so I can show them here. In my own lab, I am able to move snapshot files around so I created a clip with [asciinema](https://asciinema.org/) where I move the files via ssh on an ESXi host. When you just have one VM the `vmsn` and `vmsd` files can also be moved via the vCenter UI.

![A general system error occurred: Fault cause: vim.fault.GenericVmConfigFault](/assets/img/2021-12-01-removing-snapshot/0-Error.png)
_A general system error occurred: Fault cause: vim.fault.GenericVmConfigFault_

![Finding the datastore where the VM files are saved](/assets/img/2021-12-01-removing-snapshot/1-check-vmx-location.png)
_Finding the datastore where the VM files are saved_

![Remove the VM from inventory](/assets/img/2021-12-01-removing-snapshot/2-remove-from-inventory.png)
_Remove the VM from inventory_

![SSH to host and move the `vmsn` and `vmsd` files to tmp folder](/assets/img/2021-12-01-removing-snapshot/3-ssh-and-move-snapshots.png)
_SSH to host and move the `vmsn` and `vmsd` files to tmp folder_

<script src="/assets/js/asciinema-player.min.js"></script>
<link rel="stylesheet" href="/assets/asciinema/asciinema-player.css">
<div id="asciinema"> </div>
<script>AsciinemaPlayer.create('/assets/img/2021-12-01-removing-snapshot/tmpi78knj72-ascii.cast', document.getElementById('asciinema'), {poster: "npt:1:23", speed: "3", });</script>
_Asciinema recording of moving the `vmsn` and `vmsd` files to tmp folder_

![Register the VM back into vCenter](/assets/img/2021-12-01-removing-snapshot/4-add-vmx-to-vcenter.png)
_Register the VM back into vCenter_

![See the consolidation needed status on the VM](/assets/img/2021-12-01-removing-snapshot/5-consolidation-needed.png)
_See the consolidation needed status on the VM_

![Consolidate!](/assets/img/2021-12-01-removing-snapshot/6-consolidate.png)
_Consolidate!_

![vCenter task in progress](/assets/img/2021-12-01-removing-snapshot/7-task.png)
_vCenter task in progress_

![Snapshot has been removed!](/assets/img/2021-12-01-removing-snapshot/8-all-is-good.png)
_Snapshot has been removed!_

The images show the full process of moving the `vmsd` and `vmsn` files, these files are key to the process of getting the snapshot files consolidated. The `vmsd` is a database file that holds information about all the snapshots and this is the primary source of information for the snapshot manager in vCenter. Without this file, vCenter doesn’t know which snapshots are present were and can only recover by consolidating any present deltas disk files. The `vmsn` is a file that contains the active memory state of the VM at the time of the snapshot. This file is no longer needed as the VM is now in shutdown and there is no possibility to consolidate the ‘active’ memory anyway.

## Where did it misalign?
After changing my script to work with multiple vCenters I found that I forgot to update a line of code. This line would actually create the snapshot and my guess is that the script would still create the snapshot on the old vCenter, despite the script removing the VM there. I tried to replay this in my lab environment but vCenter 7.0 now has the ability to natively use [Advanced Cross vCenter Server Migration](https://blogs.vmware.com/vsphere/2020/12/advanced-cross-vcenter-server-migration-integrated-in-vsphere-7-u1c.html). So I suspect that it handles a VM with a snapshot better than the older vCenter version but, because of time, I haven’t been able to test that. For now, the error is solved and that is the most important!

## Closing thought
In the end, it was a lot of work to remove all the snapshots that were stuck. It showed me that automation can often help to make dull tasks quicker but when it goes wrong then it can cause some more work. This situation clearly is an example of that. All in all, I was happy to be able to migrate the VMs from one platform to the new one, and that I managed to remove the snapshots in the end.

Thanks for reading! Hopefully, you found it interesting and maybe you’ve even learned something new! Want to be informed about a new post? Subscribe! Any questions or just want to leave a remark? Please do so- I’m always very curious to hear what you think of my content. Enjoy your day!


