---
title: "VMware Cloud Director: Failed to remove Org VDC"
author: bart
date: 2024-02-01 10:00:00 +0100
categories: [VMware, VMware Cloud Director, Tech]
tags: [Tech, VMware Cloud Director, VMware]

published: true

img_path: /assets/img/failed-to-remove-orgvdc/
image:
  path: onedoesnotsimply.png
---

*[VCD]: VMware Cloud Director
*[Org VDC]: Organization Virtual Data Center

Past year, I've been busy with VMware Cloud Director (VCD) and in new versions some awesome collaborations with VMware NSX have been made, more about that in future blogs, but I think I might have caught a bug in this new version, so let's get into it!

## VMware Cloud Director
This will be the first blog on my page about VMware Cloud Director (VCD). I'm still busy writing a VCD introduction blog which will come online somewhere in 2024, but I really wanted to share this bug. Since the introduction blog is not online yet, I'll just assume that you know how VCD works and what it does.

## Some background
On November 30th version 10.5.1 of VCD got released (*[VMware Cloud Director 10.5.1 is now GA](https://blogs.vmware.com/cloudprovider/2023/11/vmware-cloud-director-10-5-1-is-now-ga.html)*) and I waited no time to upgrade my LAB VCD to this new version. Since my VCD is only used in test situations and there are no real Org VDC deployed the easiest upgrade was to just scrap the old VM and deploy a new one. And that's just what I did. All worked well and I started to play around with the "NSX Tenancy" setting. This setting will provide the translation from VCD organizations to VMware NSX Projects, providing an easy separation of tenants in their own 'project' space. The project will limit the scope of items shown and allows for a more cloud provider like setup. Also, a dedicated 'Short log identifier' will be set that will be used on all syslog messages from said organization, providing the possibility to split syslogs to a dedicated syslog collector for that tenant/project.

## The error
Since this VCD is used for playing around I wanted to reproduce what I've just build and try to build it again. If something is working straight out of the box it's either working very well or a fluke, and it will break the second time 😁. So I've tried to remove the Org VDC that I've created and VCD was able to delete most networking components but was not able to delete the Virtual Datacenter.

```shell
Operation: Deleted Virtual Datacenter 020_Rattmann(0d790692-d362-42db-8223-f005a717df53)
Type: vdc
Status: Failed
Organization: System
Service Namespace: com.vmware.vcloud
Details: [ 7faf1e0f-775f-4576-bd0b-a878b49c7c90 ] Internal Server Error - Bad Request: The object path=[/orgs/default/projects/06035aea-d0e9-4f1c-b5aa-ec8df0ada9c3] cannot be deleted as either it has children or it is being referenced by other objects path=[/orgs/default/projects/06035aea-d0e9-4f1c-b5aa-ec8df0ada9c3/infra/domains/default/gateway-policies/79790f77-e28c-4305-8746-f366b29465e6], error code 500030
Debug Information: com.vmware.vcloud.api.presentation.service.InternalServerErrorException: Internal Server Error at com.vmware.vcloud.api.presentation.service.impl.VdcServiceAdapterImpl.waitForFuture(VdcServiceAdapterImpl.java:2621)
```

![VCD Delete failed](vcd-delete-failed.png)

### Start with the investigation

The error here is letting us know that it was not possible to delete an item since it's still in use. Well that's not nice! So first a check what is still configured for that Org VDC, but I couldn't find any configuration items still in use. I took a second look at the error and saw that this error comes straight out of NSX since I recognize the API path (```/orgs/default/projects/<id>/infra/domains/....```). My second investigation focused on NSX, but I couldn't find any resources in use there either. NSX will show that there are some default rules in the Distributed Firewall, but they simply can't be deleted. The API call used shows that a project still had something with 'gateway policies' but there's no T1 Gateway linked to the project and the gateway firewall is empty, so why is this error showing?

| ![NSX Networking overview](nsx-networking.png) | ![NSX DFW overview](nsx-dfw.png) | ![NSX Gateway Firewall overview](nsx-gw-fw.png) |


Onwards to the API then! I'm using [Postman](https://www.postman.com/) simply because I don't know any other API client that is not *curl* or *wget* and is a bit more sophisticated. I've configured my Postman as [Rutger Blom](https://rutgerblom.com/2019/06/16/getting-started-with-the-nsx-t-api-and-postman/) shows in his blog except I didn't load the OpenAPI specifications so be aware that when you see ```{% raw  %} {{baseURL}} {% endraw  %}``` this means the FQDN of my NSX manager. The first request that I did was a ```GET``` of that API path ```{% raw  %} https://{{baseUrl}}/orgs/default/projects/06035aea-d0e9-4f1c-b5aa-ec8df0ada9c3/infra/domains/default/gateway-policies/79790f77-e28c-4305-8746-f366b29465e6 {% endraw  %}```, this will have a strange result since the output will have the title *VMware NSX \| Login*.

![Postman get gateway policies](postman-get-gateway-policies.png)

I've then dug a bit deeper and found that the error is not the full path of the request it's doing you will need to add ```{% raw  %} https://{{baseUrl}}/policy/api/v1/ {% endraw  %}``` as a prefix yourself. So a second attempt with the following API call ```GET {% raw  %} https://{{baseURL}}/policy/api/v1/orgs/default/projects/06035aea-d0e9-4f1c-b5aa-ec8df0ada9c3/infra/domains/default/gateway-policies/79790f77-e28c-4305-8746-f366b29465e6 {% endraw  %}```

This does show a result and indeed shows a *Gateway Policy* that VCD apparently cannot or forgot to delete.

```json
{
    "rules": [],
    "resource_type": "GatewayPolicy",
    "id": "79790f77-e28c-4305-8746-f366b29465e6",
    "display_name": "79790f77-e28c-4305-8746-f366b29465e6",
    "tags": [
        {
            "scope": "SYSTEM",
            "tag": "urn:vcloud:org:06035aea-d0e9-4f1c-b5aa-ec8df0ada9c3"
        },
        {
            "scope": "SYSTEM",
            "tag": "urn:vcloud:vdc:0d790692-d362-42db-8223-f005a717df53"
        },
        {
            "scope": "SYSTEM",
            "tag": "urn:vcloud:gateway:79790f77-e28c-4305-8746-f366b29465e6"
        }
    ],
    "path": "/orgs/default/projects/06035aea-d0e9-4f1c-b5aa-ec8df0ada9c3/infra/domains/default/gateway-policies/79790f77-e28c-4305-8746-f366b29465e6",
    "relative_path": "79790f77-e28c-4305-8746-f366b29465e6",
    "parent_path": "/orgs/default/projects/06035aea-d0e9-4f1c-b5aa-ec8df0ada9c3/infra/domains/default",
    "remote_path": "",
    "unique_id": "733b34c7-7dbc-4f35-aa68-e571d4d43e2b",
    "realization_id": "733b34c7-7dbc-4f35-aa68-e571d4d43e2b",
    "owner_id": "a3a5e538-1a5d-451c-8725-b3cf3b12eb8c",
    "marked_for_delete": false,
    "overridden": false,
    "sequence_number": 0,
    "internal_sequence_number": 54000000,
    "category": "LocalGatewayRules",
    "stateful": true,
    "tcp_strict": true,
    "locked": false,
    "lock_modified_time": 0,
    "rule_count": 0,
    "is_default": false,
    "_create_time": 1702465593549,
    "_create_user": "sa_vcd_admin",
    "_last_modified_time": 1702465593549,
    "_last_modified_user": "sa_vcd_admin",
    "_system_owned": false,
    "_protection": "NOT_PROTECTED",
    "_revision": 0
}
```

![Postman get gateway policies](postman-get-localgatewayrules.png)

## The fix

The fix is actually quite simple. Since we're already using the API to verify that there is indeed some logical construct that prevents the NSX project from being deleted we could try to use ```DELETE``` instead of ```GET``` with that exact Gateway Policy. 

```DELETE {% raw  %} https://{{baseUrl}}/policy/api/v1/orgs/default/projects/06035aea-d0e9-4f1c-b5aa-ec8df0ada9c3/infra/domains/default/gateway-policies/79790f77-e28c-4305-8746-f366b29465e6 {% endraw  %}```

![Postman DELETE Gateway Policy](postman-delete-gatewaypolicy.png)

Would it now be possible to delete the Org VDC? Yes! The task finishes successfully.

![VCD Delete Succeeded](vcd-delete-succeeded.png)

I've also tested deleting an Org VDC that doesn't use NSX Tenancy and this works perfectly every single time, so the impact is only related to Organizations and Org VDC that have NSX Tenancy enabled.

## Closing thoughts
Thanks for reading! Hopefully, you found this interesting, and maybe you've even learned something new! This blog was all about VMware Cloud Director, but more blogs are on my list to write and publish. Any questions or just want to leave a remark? Please do so, I'm always curious to read what you think of my content. Enjoy your day!