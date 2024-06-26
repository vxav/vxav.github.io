---
layout: post
title: What's new in BDRSuite v5.1
DATE: 
subtitle: ''
metadescription: Learn about the new features and improvements released with BDRSuite
  BDRSuite v5.1.
image: ''

---
The world of data protection has been in motion for the past several years with the rise of cloud services and SaaS in general. BDRSuite carved a place for the company among the leading backup software vendors for the SMB sector with a robust virtual machine backup and recovery solution.

While the company keeps improving the core components to answer customer feedback, BDRSuite brought extensive cloud integration features in the latest versions to maintain the products place on the podium.

Following a change to a more rapid product releases cycle, BDRSuite announced the release of BDRSuite v5.1 Only four months after the release of [BDRSuite v5.0](https://www.vxav.fr/2021-10-11-vembu-bdr-suite-v5.0-ga-release/). BDRSuite v5.1 brings, among other things, the long awaited Object Storage Backup Repository and Backup Copy capabilities. Let's dig into it and find out what's new in BDRSuite v5.1.

[![](/img/vembu-download.png)](https://www.bdrsuite.com/vembu-bdr-suite-download/#)

BDRSuite v5.1 is a Windows or Linux installer and you can use it unrestricted for free during a **30 days trial period**.

#### Object Storage Backup Repository

As you know most backup solution often rely on local storage or some flavour of shared storage such as NFS, SMB, iSCSI and other protocols to present a volume or a share to the backup proxy. But how cool would it be to map cheap cloud storage as your destination to ensure flexibility and resilience? BDRSuite v5.1 now brings capabilities to use object storages such as AWS S3 & S3 compatible storage solutions as a storage destination for the backup data. A great use would be to use it to enforce the 3-2-1 rule with an offsite copy stored on a different medium.

![](/img/vembu5-11.png)

#### Backup Copy

Like mentioned before, the 3-2-1 rule states that you should have 3 copies of your data, on 2 different mediums and 1 offsite copy. The Backup Copy feature in BDRSuite v5.1 will now allow you to create a copy of your primary backup data and store it in a different backup repository. Note that you can leverage Object Storage backup repos.

![](/img/vembu5-12.png)

#### BDR Cluster Deployment

Many aspects of the datacenter often raise the question of wether to scale-up or scale-out. While scaling up may produce the same outcome, it induces some downtime while resizing the server and isn't particularly flexible. This is why BDRSuite v5.1 introduces BDR Cluster Deployment, a feature that lets you easily scale out your BDRSuite Backup Server by adding instances. You can then make them work together for load balancing and high availability.

Note that you can convert a standalone installation into a cluster setup as you can see on the screenshot of my server below.

![](/img/vembu5-13.png)

#### Additional product enhancements

A number of additional improvements are brought into BDRSuite v5.1 on top of the features presented above. Among which we find:

* Backup database optimized for better performance
* Reconnection attempts for the backup replication to Offsite DR Server
* Option to run the tape backup jobs immediately after saving it
* Option to run the tape backup jobs on-demand from the BDR & ODR Server
* The tree listing of the tape backup recovery page has been updated
* Automatically backup the tape agent database daily for disaster recovery scenarios
* Restore OneDrive/Group OneDrive files & folders to local storage drives
* Download Microsoft 365 Calendar & Contact items directly from the BDRSuite Backup Server UI
* Restore Google Drive files & folders to local storage drives
* Download Google Workspace Calendar & Contact items directly from the BDRSuite Backup Server UI
* AWS EC2 Instance level Recovery

#### BDRSuite v5.1 update

Updating your BDRSuite server is very easy. However, it is recommended to review the [Software Update Guide](https://www.bdrsuite.com/pdf/release-notes/vembu-bdr-automatic-software-update.pdf) to make sure your scenario is covered and the [Upgrade Checklist](https://www.bdrsuite.com/pdf/release-notes/vembu-bdr-suite-v5-0-ga-pre-and-post-checklist.pdf) before upgrading to BDRSuite v5.1.

#### Wrap up

Product management and including customer feedback to the roadmap is important for any company to answer real world problematic and not only cover the angles of the industry standards. BDRSuite has been doing it brilliantly so far in BDRSuite v5.1 with a lot of improvements in a relatively short span of time. I will be sure to keep my eyes peeled in the next few months to find out what the next versions of the product will bring in terms of features and improvements.