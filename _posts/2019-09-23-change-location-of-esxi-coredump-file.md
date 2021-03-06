---
layout: post
title: Change location of ESXi coredump file
DATE: 

---
A core dump, also referred to as crash dump or memory dump, contains the content of the memory at a given point in time. It is usually created when the system or program crashes. It is a way of saving evidence that could help troubleshoot the root cause of the issue. It would be awkward if the housekeeper mopped the floor at the crime scene before Gil Grissom and his pals arrived... Anyway, I digress.

ESXi has three coredump mecanisms to save crash information including a network location, a partition or a file (the topic of this article).

The coredump file is created by ESXi at boot time. The size (function of the memory installed in the host), location and names are defined automatically. Sometimes a local datastore is selected and sometimes it is a shared one. There are a few reasons why you may want to change the location of the file, including but not limited to :

* Is it preventing you from decommissioning a datastore?
* Are you running out of space on this datastore?
* Do you want to keep the file on a local/shared datastore?
* Do you want to dedicate a datastore for this purpose?
* ...

##### Identifying the coredump files

They are stored in a folder name "**vmkdump**" in the datastore and have a unique name like so.

![](/img/corefile1.png)

##### Identifying the datastore UID

We will need it later to identify the host owning the files. You can find it in "**Datastore**">"**Summary**">"**URL**".

![](/img/corefile2.png)

You can also use PowerCLI.

    PS> (get-datastore *XXXX*).ExtensionData.summary
    
    Datastore          : Datastore-datastore-120
    Name               : XXX-XXX-XXX-XXX
    Url                : ds:///vmfs/volumes/570e3e4a-a3cbd39f-5335-e41f13815e0b/
    ...

##### Identifying the hosts using the files

* The command below will list the coredump file information about each cluster host and store it in a variable.

      $corefile =  get-cluster MyCluster | Get-VMHost | ForEach-Object{
      
      $esxcli = Get-EsxCli -VMHost $_ -V2
      
      $esxcli.system.coredump.file.list.invoke() | where active -eq $true | select @{l="vmhost";e={$esxcli.VMHost.name}},*
      
      }
* We can then use this information to find which coredump file is stored in the incriminated datastore by filtering using its UID gathered previously.

      PS> $corefile | where path -match "/vmfs/volumes/570e3e4a-a3cbd39f-5335-e41f13815e0b/"
      
      vmhost     : ESX-Host-01
      Active     : true
      Configured : true
      Path       : /vmfs/volumes/570e3e4a-a3cbd39f-5335-e41f13815e0b/vmkdump/E3C23887-677B-8B46-A501-E4F9AD2877A4.dumpfile
      Size       : 6287261696

Now we know that ESX-Host-01 has its coredump file stored on the datastore we want to move it from.

Note that you could also identify the host that is holding a lock on the file by using the "vmkfstools -D xxx" in SSH but I wanted to stay in PowerCLI with the esxcli coredump commands.

##### Moving the coredump file to another datastore

* List current coredump file for the host.

      $esxcli = Get-EsxCli -VMHost ESX-Host-01 -V2
      
      PS> $esxcli.system.coredump.file.list.invoke()

![](/img/corefile3.png)

* Disable the current coredump file. The **Active** and **Configured** properties are now set to **False** if you run the list method again.

      PS> $esxcli.system.coredump.file.set.invoke(@{unconfigure=$true})
      
      true

![](/img/corefile4.png)

* Remove the coredump file (not mandatory). You see in the screenshot that the line disappeared (the file is also gone in the datastore).

      PS> $esxcli.system.coredump.file.remove.Invoke(@{file="/vmfs/volumes/570e3e4a-a3cbd39f-5335-e41f13815e0b/vmkdump/E3C23887-677B-8B46-A501-E4F9AD2877A4.dumpfile"})
      
      true

![](/img/corefile5.png)

* Create a new file and enable it. The new file is now active and configured on the new datastore.

      PS> $esxcli.system.coredump.file.add.Invoke(@{datastore="Datastore-02";enable=$true})
      
      true

![](/img/corefile6.png)