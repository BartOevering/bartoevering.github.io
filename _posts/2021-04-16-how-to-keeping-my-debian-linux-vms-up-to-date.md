---
title: "How to: keeping my Debian Linux VMs up to date"
author: bart
date: 2021-04-16 10:00:00 +0100
categories: [Tech, VMware, Scripting, PowerCLI, Wheatley]
tags: [Tech, VMware,  Scripting, PoshSSH, PowerCLI, Wheatley]

redirect_from:
  - /tech/scripting/how-to-keeping-my-debian-linux-vms-up-to-date/

published: true
toc: true

img_path: /assets/img/2021-04-16-debian-update/
image:
  path: meme_powershell.jpeg
  alt: Matrix Powershell is mandatory
---

Now that the [lab is up]({% post_url 2021-01-15-new-hardware %}) for a few months I noticed that I already installed quite a few Debian Linux VMs. Debian Linux is the default OS that I use when I just want a lightweight server to run some application as a learning/play environment. These VMs run quite autonomous so I don’t often logon and I found that they are often not up-to-date, in this blog I’ll take you along with the solution that I created for that.

## What goal do I want to accomplish?
The lab has now been running for about six months and in that time I installed 15 Debian Linux VM’s. Debian Linux is my go-to *Operating System* (OS) for many VMs because Debian is a lightweight server-grade OS that I’m familiar with and I find it easy to configure. These VM’s do all kinds of things like, one is a PiHole DNS server another one is for Home Assistant but also have some small hobby VMs that are connected straight to the internet.

When I now log in to a VM, I first update that VM by typing: `apt-get update && apt-get -y upgrade` before doing anything else. After the update has finished, which usually takes maximally about five minutes, I continue to do what I wanted to do on that VM. This is a lot of manual work and leaves an inconsistent state between the VMs as one might be more up-to-date than another.

Recently I noticed that I don’t log in to most of those VMs on a daily/weekly or sometimes even monthly basis and that means that they are not updated for a longer period. The problem with that is that there might be an update for a package because it’s vulnerable to an attack and it might not get updated on time or the VM is forgotten. Therefore, I decided that security is important and started thinking about options to better keep the VM’s up-to-date!

## Which options are possible?
What I try to do is not unique- Debian or at least Linux is widely used in the industry and they all need patches and updates. Many companies are using automation frameworks like [Ansible](https://www.ansible.com/) or [Terraform](https://www.terraform.io/) to keep their VMs up-to-date and I thought about doing the same. I started looking into how these frameworks work and figured out that they do way more than only automated updates of VMs. Both Ansible and Terraform can be used to, for example, create a VM in VMware vCenter and install Debian with a specified application. All these extra functions make it more complex than I want right now so continued my search.

During my search, I found that there is a package/application that can be set up on all VMs called ‘[unattended-upgrades](https://wiki.debian.org/UnattendedUpgrades)‘. This package needs to be installed and configured on every Debian VM and it will create a schedule to update that VM. This solution will fulfill my main need of having VM’s up-to-date but it’s not a process that I can influence other than changing the moment when it will install the updates. I’d like to have some more control and that only leaves the third option: creating my own solution. I got the inspiration during my course for becoming a *Cisco Certified Network Professional* (CCNP) where a Python script was used to log in to network devices via SSH.

## What does the script need to do?
Now, Python is not something I often use, however, I do often use Microsoft PowerShell. I started looking into a PowerShell module that would allow me to build an SSH connection. The SSH connection is important because that makes it possible to send CLI commands which will allow me to install the needed updates. After a short search, I found a PowerShell module called ‘[Posh-SSH](https://www.powershellmagazine.com/2014/07/03/posh-ssh-open-source-ssh-powershell-module/)‘ that will supply this functionality.

Also, important is that the script needs to be dynamic- I don’t want to feed in a static list of VM’s to update and find out after a couple of months that I missed VM’s. The script must therefore build a list with Debian VM’s that are powered on in VMware vCenter and install the updates on those VMs. The script will use that list of VMs to connect to every VM via SSH and install all the relevant updates similar to the command I would normally put when I log in. The script will now update one VM at a time but as a future improvement, I’d like to make it process the VM updates in parallel to be more efficient. With all these steps in mind, I decided it’s easier to create a flowchart with all that the script needs to do and which commands I need as you can find below.

![flowchart of the goal](flow.png)

With the flowchart in mind, the fun part starts, building the code.

## The nitty-gritty details
Just to be sure, I’m not a programmer. The script that I create can, probably, be improved in many ways, for example, with a more [Object-Orient](https://en.wikipedia.org/wiki/Object-oriented_programming) approach and this script can definitely have better [error handling](https://docs.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-exceptions). This is, however, the way that I write my scripts and usually, it gets the job done.

### Setting parameters and loading modules
The first step in the script is to set some parameters that are needed. Because the script retrieves information from vCenter the script needs to know where vCenter is located and which user the script needs to use to login. After the parameters are set the script start loading modules. In this case, there are two modules needed; [VMware PowerCLI](https://developer.vmware.com/docs/powercli/latest/products/) and the [Posh-SSH](https://www.powershellmagazine.com/2014/07/03/posh-ssh-open-source-ssh-powershell-module/) module.

The PowerCLI module will make it possible to manipulate vCenter in almost any way you can also do via the GUI. Load the module will make it possible for the script to retrieve a list of VMs registered in vCenter. An added benefit of the PowerCLI module is that I can make sure that the credentials for vCenter are stored safely by using the ‘[VICredentialStore](https://developer.vmware.com/docs/powercli/latest/vmware.vimautomation.core/commands/get-vicredentialstoreitem/#Default)‘ function. The CredentialStore saves the credentials fully encrypted in a file on the system and this file can only be decrypted on the same computer by the same user making it a secure way of storing these highly sensitive credentials.

### Build list of VMs
Now that there is a connection to VMware vCenter it’s possible to request a list of VMs present within vCenter. The ‘[Get-VM](https://developer.vmware.com/docs/powercli/latest/vmware.vimautomation.core/commands/get-vm/#Default)‘ command will provide a list based on the display name of the VM and the script will need to filter this list because not all VMs have Debian as OS. Also, the script should remove VMs from the list that has PowerState ‘poweredOff’ and VMs that are in the resource group ‘RvG’ as they don’t need to be updated by me. The Get-VM command shows the display name but for the SSH connection, an IP address or *Fully Qualified Domain Name* (FQDN) is needed. The script will get the registered IP address of VMware tools as this is the most reliable way of connecting to SSH.

At this point, the script can start to build an SSH connection to the first VM in the list. However, if anything goes wrong during updating I’d like to have a log of what happened. For that reason, a full transcript of any console output is saved every time the script is started.

### Connect to VM, check hostname and user rights
Now everything is prepared to finally connect to the first IP address in the list. The IP address of a VM could be either dynamically or statically assigned to a VM so when to SSH connection is established the first thing I want to show is a message showing the set hostname of that VM. The hostname is similar to the display name in vCenter for easy troubleshooting.

While setting up the SSH connection a username is required and that is filled in with ‘root’ because the SSH key that I now have is for that user. In the future, I want the script to use an own user on the Debian VMs and the script is prepared for that. For now, the script will check if the username is equal to ‘root’ by issuing the ‘[whoami](https://en.wikipedia.org/wiki/Whoami)‘ command. This way the script knows it has the rights to use ‘[apt-get](https://wiki.debian.org/apt-get)‘ to install the updates.

### Update the VM
If the script indeed has root rights it can continue with updating the OS by invoking the command `apt-get update && apt-get -y upgrade && apt-get -y dist-upgrade && apt-get -y autoremove`. After the `apt-get` command, the VM is completely up-to-date and that was the goal!

The script could now be finished but can do more. There are some packages that just need to be installed on every Debian VM because they are important, for example, ‘[open-vm-tools](https://github.com/vmware/open-vm-tools)‘ and ‘[openssh-server](https://www.openssh.com/)‘. To check there is a variable `$installPackages` which is an ‘[array](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_arrays)‘. The script will iterate through every package noted in the array and if it’s missing the script will install it with the use of [`apt-get`](https://wiki.debian.org/apt-get).

### Check extra work
Now with the VM up-to-date and all the extra packages installed the script could be done. Sadly I found that not all packages are updated. In a previous post, I mentioned that I have a Pi-hole running but it’s not possible to update Pi-hole with the `apt-get -y update` command because it just works differently. Therefore, the script will check the hostname of the VM it’s connected to and if this is equal to ‘DC-PIHOLE01’ it will also run the needed command ‘pihole -up‘ to update Pi-hole.

### Exit and close SSH session
Now that everything is fully up-to-date and it’s time to start with the clean-up. The script will exit out of the ssh session and because the SSH session is then no longer needed it session will be removed. During running the script I found out that the removal doesn’t always go smoothly, so there is a piece of code at the end of the script to make sure that all SSH sessions are removed successfully.

## The script itself
```powershell
<#  
.SYNOPSIS
Author: Bart Oevering
Version: 0.1 (01-01-2020) 

.DESCRIPTION
The script will get the Debian VM's from vCenter that are not in a specific resourcegroup.
These VM's are the input to create ssh session the ssh connection is created based on a ssh user, RSA key and key passphrase
If the SSH passphrase is not stored in a encrypted manner it'll ask for the phrase and save this.
A transscipt will be stored and the connection will be made to the first IP of the VM (based on vCenter).
With the ssh session, check user to be root and then update, upgrade, dist-upgrade and autoremove for any VM connected to.
Then check the VM for the packages defined in $installPackages and if not there install.
For any special servers this can be done based on $sshHostname (e.g. DC-PIHOLE01 and run pihole -up there).
Cleanup all sshSession that might be still there and logoff.

#>
# #----------------[Declarations]---------------------
[string]$sshPassFile        = "..\Data\ssh"
[string]$sshKeyFile         = "..Data\id_rsa"
[string]$sshBaseFile        = ".\Transcript"
[string]$sshUserName        = "root"
[string]$vCenter            = "vcenter.wheatley.local"
[string]$vCenterUser        = "scripting@vsphere.local"
[array]$installPackages     = @("open-vm-tools", "openssh-server")
[hashtable]$virtualServers  = @{}

#----------------[Load Modules]---------------------
Import-Module Posh-SSH
Import-Module VMware.VimAutomation.Core
#----------------[The Work]---------------------

#Because everythings in git, the documentlocation must be correct to find the needed files.
Push-Location $PSScriptRoot

#Connect to vCenter
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

if (!$((Get-PowerCLIConfiguration -Scope User).InvalidCertificateAction -eq "Ignore")) { Set-PowerCLIConfiguration -InvalidCertificateAction "Ignore" -Confirm:$false }
if (!$(Test-Path "$env:APPDATA\VMware\credstore\vicredentials.xml")) { New-VICredentialStoreItem -Host $vCenter -User $vCenterUser -Password ((get-credential).GetNetworkCredential().password) }
if ($null -eq $vcconn) { $vcconn = Connect-VIServer -Server $vCenter  }

#Password, first check the file and if exist get password from there otherwise, ask for the password to save securely
if (!$(Test-Path $sshPassFile)) {     
    $Secure = Read-Host -Prompt "No file found at: $sshPassFile`nEnter SSH passphrase for file: id_rsa" -AsSecureString
    $Secure | ConvertFrom-SecureString | Set-Content $sshPassFile
}
$sshPassPhrase = Get-Content $sshPassFile | ConvertTo-SecureString
$Credentials = New-Object System.Management.Automation.PSCredential($sshUserName, $sshPassPhrase)

#Now get a list of Debian VM's that are on and not in RvG resource group from vCenter
$listOfVMNames = ((Get-VM).where{$_.PowerState -eq 'PoweredOn' -and $_.Guest.OSFullName -match 'Debian' -and $_.ResourcePool -notmatch 'RvG'}).Name | Sort-Object
foreach ($vm in $listOfVMNames) {
    $virtualServers.Add($vm, (Get-VM -Name $vm | Select-Object Name, @{N="IPAddress";E={@($_.guest.IPAddress[0])}}).IPAddress)
}

#Make sure that we have a transcript file, could be handy at some point
$date = Get-Date -format "yyyyMMdd_hhmm"
Start-Transcript -Path $sshBaseFile"\"$date".txt"

foreach ($virtualServer in ($virtualServers).Values) {
    #Create SSH session
    try {
        $sshSession = New-SSHSession -ComputerName $virtualServer -Credential $Credentials -KeyFile $sshKeyFile -AcceptKey:$true -ErrorAction Stop
    }
    catch {
        Write-Host "Oeps! and error has occurred! `nNo connection to $virtualServer was made! `nPlease check manually!" -ForegroundColor DarkRed
    }
    if (($sshSession).Connected) {
        $sshHostname = (Invoke-SSHCommand -Index 0 -Command "hostname").output
        Write-Host "Connected to server: $sshHostname" -ForegroundColor DarkCyan

        #Now check if all whent well and we are indeed root
        $sshCommand = Invoke-SSHCommand -Index 0 -Command "whoami"
        if (($sshCommand).output -eq "root") {
            #We are root, so we can do some stuff :) 
            Write-Host "Connected to the server as $(($sshCommand).output)" -ForegroundColor DarkGreen
            Write-Host "Installing updates for Debian OS and running dist upgrade" -ForegroundColor Cyan

            #Check for any updates and install including dist-upgrades
            $sshCommand = Invoke-SSHCommand -Index 0 -Command "apt-get update && apt-get -y upgrade && apt-get -y dist-upgrade && apt-get -y autoremove"

            #Check if commom packages are installed
            foreach ($package in $installPackages) {
                Write-Host "Checking for $package" -ForegroundColor Cyan
                $command = 'dpkg-query -W -f=''${Status}\n'' ' + $package
                $sshCommand = Invoke-SSHCommand -Index 0 -Command $command
                if (($sshCommand).output -ne "install ok installed") {
                    Write-Host "Package: $package is not installed! `nInstalling the package now" -ForegroundColor DarkRed
                    $sshCommand = Invoke-SSHCommand -Index 0 -Command "apt-get install -y $package"
                }
                if (($sshCommand).output -eq "install ok installed") {
                    Write-Host "Package: $package is installed on the server!" -ForegroundColor DarkGreen
                }
            }

            if ($sshHostname -eq "DC-PIHOLE01"){
                #When we connect to DC-PIHOLE01, update pihole as well
                Write-Host "Installing updates for PIHOLE" -ForegroundColor Cyan
                $sshCommand = Invoke-SSHCommand -Index 0 -Command "pihole -up"
            }
        }
        else {
            Write-Host "Oeps, an error has occurred!`nNot connected as root but as: $(($sshCommand).output)" -ForegroundColor DarkRed
        }
        
        #Remove the session as all work has finshed
        Write-Host "All done! `nRemoving SSH session" -ForegroundColor DarkMagenta
        $sshCommand = Invoke-SSHCommand -Index 0 -Command "exit"
        $sshSession = Remove-SSHSession -Index 0
    }
}

#if something goes wrong and there are idk how many sessions, lets take them out otherwise the script doenst run well
if (Get-SSHSession) {
    Write-Host "Still some ssh sessions found, removing all ssh session..." -ForegroundColor DarkYellow
    foreach ($sshSession in (Get-SSHSession).SessionId){
        $sshSession = Remove-SSHSession -Index $sshSession
    }
}
Stop-Transcript
```

With the script fully functional there is a nice transcript file stored on the disk that shows me all VMs are successfully updated and if something went wrong it’s also in this file. To make the console look good I also added some colors to the output the script will show.

![Script Output](scripting_update_debian.png)

## Closing thought
I’m very happy with the script; I used it quite a few times now and it really made keeping the VM’s up-to-date way easier. Gone are the times where, when I login to a VM, I need to update VMs by hand. I just run the script, it runs for a few minutes and everything is fully up-to-date. As with everything in IT, there’s probably something out there making it even easier and more granular on what updates to install but this works like a charm for me!

Thanks for reading! Hopefully, you found it interesting and maybe you’ve even learned something new! Want to be informed about a new post? Subscribe! Any questions or just want to leave a remark? Please do so- I’m always very curious to hear what you think of my content. Enjoy your day!

