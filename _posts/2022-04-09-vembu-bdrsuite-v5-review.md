---
layout: post
title: BDRSuite v5 review
metadescription: null
date: 2022-04-09T14:44:40.592Z
fb-img: null
---

BDRSuite is a full fledged Backup and disaster recovery solution aimed at protecting workloads across Virtual, Physical, on-premises, Cloud services, and SaaS applications. Following a change to a more rapid product releases cycle, BDRSuite announced the release of BDRSuite v5.1 which we talked about in a [previous blog](https://www.vxav.fr/2022-02-13-what-s-new-in-vembu-bdr-suite-v5.1/). I installed BDRSuite in my environment to play with it so today we are reviewing it and I will give you my opinion on it.

## BDRSuite setup

First off let's discuss the installation process of the software. For most cases BDRSuite is a Windows installable that supports Windows Server 2012 R2 onwards. There is also a Linux installer, although it is a bit peculiar. There is a Shell script that you can download for BDRSuite 5.0 to install it but BDRSuite 5.1 redirects you to contact support so I'm not sure about what this means. I did try it in an older version and it worked fine. However, like I mentioned I assume that most will use the Windows setup.

[![BDRSuite download](/img/vembu5-1.png)](https://www.bdrsuite.com/vembu-bdr-suite-download/?utm_source=vxav&utm_medium=blog&utm_campaign=download)

The system requirements for the BDR backup server are as follows. Note that they are the same for the Offsite DR server if you want to copy your backups to an offsite location.

| Resource | Minimum | Recommended |
|---------|----------|---------|
| RAM | 8GB | 16GB |
| CPU | 4 vCPU | 8 vCPU |
| Network | 1Gbps | 1Gbps |

Sizing your storage repositories can be tricky, and that's why BDRSuite offers a basic but useful [storage sizing tool](https://www.bdrsuite.com/vembu-storage-calculator/) in which you will get recommendations for backup or replication based on size, change rate, recovery points and number of full backups. I would have liked something a bit more customizable but the goal is to give a rough idea of requirements.

![BDRSuite storage sizing tool](/img/Vembu%20storage%20sizing%20tool.png)

I won't describe the installation process here as it is very much similar to any Windows software installation with an installation folder, a bunch of _Next_ to click, a service account, a BDR admin account and that's it. However, the embedded postgres database location is recommended to have a free space of at least 3% of the total size of the backup data and not reside on the OS disk so watch out for that.

Once you have installed the BDR Backup server, you will need in your BDRSuite Portal in which you need to create an account for your organization if you don't already have one. I quite like the idea of the BDRSuite Portal, it is an all-in-one hub spot for managing your registered BDRSuite products and services and where you can get started.

Note that you can also run you BDRSuite Backup servers in high availability (HA) as it supports load balancing and scalability by combining multiple servers, each running an instance of the BDR Backup Server application, that are connected to a common database. This will improve performance and reliability.

![BDRSuite Portal Account to register your BDRSuite BDR Backup Server ](/img/2022-04-09-10-30-08.png)

## VMware environment protection

Like any respectable VM backup software, protecting VMware VMs with BDRSuite of course uses agentless backup and replication. In a nutshell, it takes a snapshot of the VM, mounts the base virtual disk to the backup proxy, copy the changed blocks tracked with CBT, unmounts the disk and consolidates the snapshots on the VM. The BDRSuite documentation describe the BDRSuite backup anatomy as follows:

* When you complete configuring the backup schedule, the CBT reset process takes place. The VM will be validated for the backup process. The BDRSuite BDR backup server creates a snapshot and using the CBT, the disk information of the VM will be processed.
* Post this the Application Aware operations will be triggered directly from the BDRSuite BDR backup server.
* The backup proxy reads the backup data, one option is to let the BDRSuite BDR backup server choose the backup proxy automatically, or choose the backup proxy for your configured backup.
* The backup server provides details about the VM to the backup proxy so that the read process begins. The read process, compression, and encryption will happen after this. The data is transferred to the backup server and the connection is closed successfully.

![BDRSuite workflow](/img/2022-04-09-11-37-58.png)

Creating backup jobs is simplified with the use of Backup job templates. A feature that does what it says on the tin, you can create a template that can be used as a base to create backup jobs. That way you ensure that they are consistent across settings such as virtual disk exclusion, guest processing, schedules, retention, encryption, proxy selection and the likes.

![BDRSuite backup job template](/img/2022-04-09-13-50-00.png)

Another feature that I really like has been around for a little while is Instant boot VM. Although you don't want to have to use it, it is reassuring to know that you have it on hand. Instant boot VM is usually used in critical scenarios when you really need a VM ro be running. It lets you recover a VM directly from a backup, meaning the backup data will be mounted in the BDRSuite Virtual Drive and shared as an NFS datastore. The VM can then be registered straight from your backup repo without having to wait for the copy process. Performances will obviously not be as good (unless you are excentric enough to run full flash backup repositories) but the VM will be running.

Although I haven't tried it, the documentation states that you can restore to Hyper-V or KVM which would actually be really cool and would offer an alternative to the end of life of VMware converter for cross hypervisors migrations. This is made possible by [BDRSuite virtual disks](https://www.bdrsuite.com/guide/vembu-bdr-suite/5-1/en/manage-vembu-virtual-drive.html).

![Instant boot VM](/img/2022-04-09-11-53-25.png)

Performance has also been taken care of with fine tuning knobs for concurrent backup tasks in the backup server and per backup job. This will allow you to avoid any bottlenecks in your backup windows as no one wants to have backup jobs still running in the morning when everyone starts the day. Customizable data compression levels will also help choosing whether you want to spend more time on compression (CPU intensive) or copy (network intensive). And if you can't fit all your jobs outside of office hours, you can always configure Bandwidth throttling in a specific time window to avoid impacting prodution workloads performance.

![BDRSuite Bandwidth throttling](/img/2022-04-09-14-41-55.png)

## Supported backup locations and offsite capabitilies

Having backups is great to protect against data loss, having them in a second location is a must have to cover disaster scenarios.

### Backup repositories

BDRSuite 5.1 supports both block storage and Object storage as backup repositories backends. Block storage is a bit confusing to me as it includes network drives i.e NAS (NFS and CIFS) which are usually called _File storage_, but it also covers all the bases with local drives (DAS) and SAN (iSCSI and FC). Object storage mode supports Amazon S3 or any object storage with Amazon S3 compatible APIs.

![BDRSuite Backup repositories](/img/2022-04-09-10-43-02.png)

Note that you can also create backup copy jobs that will create a copy of your primary backup data and store it in a different backup repository. That way, in case the primary backup data got corrupted or lost, the backup copy can be used to recover data.

### Offsite backup capabilities

Like any good Backup software, your backups can be replicated to an offsite location to an on-premise Offsite DR Server or to the BDRSuite cloud to a CloudDR Server to cover you against data loss in case of disaster. The replication process is of course built on the same foundations as the backup process as it will leverage VM snapshots for recovery points and CBT for incremental capabilities.

The Cloud DR Server web console will be your pane of glass to manage your offsite backup copies in BDRSuite cloud. From there you can:

* Restore VMs & Windows Machines
* Restore File & Application Data (Exchange Server and mailboxex, Sharepoint, Outlook, SQL and MySQL)
* Delete the backup
* Check the status of the backup
* View Reports

![VMware backup offsite BDRSuite](/img/2022-04-09-11-42-10.png)

## licensing

It may take you a little effort to figure out the licensing model of BDRSuite and which one to go for. However, such level of granularity is a good thing in my opinion as it means you get to only pay for what you need instead of paying for extra capacity or added features you'll never use.

You can also decide whether you want to include BDRSuite in your OPEX or CAPEX budget with subscription or perpetual license models. Both let you choose between VM based licensing or CPU socket based pricing. The CPU socket based model includes free maintenance and support for the first year while the subscription model includes product updates, upgrades and standard technical support during the subscription period.

Independant from the licensing model, several editions are there to choose from. You can find feature comparisons in the [editions comparison](https://www.bdrsuite.com/vembu-bdr-suite-free-vs-paid-edition/) page. Note that the [free version](https://www.bdrsuite.com/vembu-bdr-suite-download/?utm_source=vxav&utm_medium=blog&utm_campaign=download) includes almost the same features as the Standard edition appart from replication capabitilies and it is limited to 10 VMs (which is great for small environments).

If you want to leverage backup offsite to BDRSuite Cloud DR, you can purchase capacity depending on your storage needs in multiples of 100 GB packages. For Example, you can buy 5 * 100 GB packages for 500 GB of Cloud DR storage requirements.

| Cloud Storage Package | 1 Year |
|---------|----------|
| Fee for a 100 GB package | $144 / 100GB / year |

## Conclusion

I have been following BDRSuite and playing with their products for several years now and it's always done the job very well. Note that they recently rebranded the product so we should say BDRSuite and not BDRSuite Backup & Disaster recovery. BDRSuite is a solid backup solution for VMware platforms with all the bells and whistles an IT department might need without going over-the-top with features no one will ever use. I particularly liked the slick and intuitive web interface to manage BDRSuite.

The BDRSuite Cloud DR offering is also a very interesting one as small organizations are looking for integrated solutions and might not want the added complexity of managing an Azure or AWS account dedicated to store offsite backups.

## Try it for free

If you want to try it for free just follow this [link](https://www.bdrsuite.com/vembu-bdr-suite-download/?utm_source=vxav&utm_medium=blog&utm_campaign=download) and click **Download Free Trial** and you can use it unrestricted for free during a **30 days trial period**. If you just want to evaluate the solution in a lab, you can simply install it in a Windows VM and target a test vCenter to start playing around with it.
