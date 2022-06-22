---
layout: post
title: Data Lifecycle Management and Backup Retention
description: Let's talk about what Data Lifecycle Management and Backup Retention are all about in simple terms.
date: 2022-06-21T16:32:32.105Z
nobanner: "yes"
fb-img: null
---

It is without a doubt that the IT landscape has been dominated and driven by data for the past decade or so and showing no sign on slowing down with cloud sovereignty jumping on the roadmap of most cloud providers in the last couple of years. The explosion of data generated by companies builds the reliance on storage and backup systems which require to be managed one way or the other. Data backup and recovery like [Nakivo Backup and Replication](https://www.nakivo.com/resources/download/trial-download/) offer great tools to ensure your data is safe and secure, but methodologies still apply, and arguably common sense as well. The world of IT, especially marketing content, is full of those terms that could mean anything and everything which sometimes can get confusing to understand, at least for me! In this article, I will break down the terms data lifecycle management and backup retention to try and make sense of them from the point of view of a technical lemming like me.

## What is data lifecycle management?

Right, first of all, we all know what data is and how it is stored. On a macro level, it's a bunch of ones and zeroes that are either presented to the consumer as object, block or file. The problem with data is that it grows fast and it needs to be put somewhere in a managed fashion to avoid the mushroom effect with data everywhere. Data lifecycle management (or DLM) is an approach to managed data in a policy-based fashion. Most enterprise software solutions have policy-based capabilities to manage data such as vSphere storage policies, Kubernetes storage classes or [Nakivo's Policy-Based Data Protection](https://www.nakivo.com/features/policy-based-data-protection/).

For instance, NAKIVO Backup & Replicationlets you set up custom policy rules to automatically back up or replicate VMs and physical machines that match a certain set of criteria. This feature simplifies and automates data protection activities in large infrastructures as well as dynamically changing environments. This reduces the risk of shadow IT (unmanaged resources) and offers the ability to enforce the security policies of your organization.

Data management lifecycle links to the concept of the CIA triad, and I don't mean sunglasses wearing agents with a conceiled weapon. The CIA triad includes the three most important aspects to consider when managing data:

* **Integrity**: Data mustn't be modified or altered by unauthorized parties. Nakivo offers immutability features to prevent tempering with backup files. Hashing is probably the best method to ensure data integrity.
* **Confidentiality**: The data must be accessible only to those authorized to have access to it. It prevents unauthorized disclosure of information. Encryption is one of the options to enforce confidentiality.
* **Availability**: Users should always have access to the data they are authorized to query. This relates to Nakivo's fields of expertise such data backup, fault tolerance, disaster recovery...

![data cia triad](/img/2022-06-22-09-20-07.png)

## What is Backup Retention?

Now that we have a better idea about data lifecycle management, let's look at Backup retention. This should be familiar to more readers as backup vendors such as Nakivo have offered backup retention settings for years in their products. By backup retention we really mean "backup retention policy". Except in very specific use cases that don't require storing backups such as [blockchain](https://insidebitcoins.com/) implementations, a backup retention policy helps the organization to be ready in case of data loss or ransomware attack by enforcing techniques such as:

* **GFS Retention Policy**: The [Grandfather-Father-Son](https://www.nakivo.com/blog/gfs-retention-policy-explained/) retention policy is a rotation of daily backups as 'sons', weekly as 'fathers', and monthly as 'grandfathers'. The point of it is that, the more up to date the data, the more granularity you get for recovery purpose while saving storage space.

![Grandfather-Father-Son retention policy](/img/2022-06-22-10-28-05.png)

* **3-2-1-1 rule**: It could be argued whether this belongs here but the 3-2-1-1 rule is so important that I would specify it in the backup retention policy. You must keep 3 copies of your backed up data on 2 different media types with 1 offsite copy and air-gapped and immutable copy.
* **Policy based retention**: Backups of different data types have different requirements. For instance, critical production databases will require a more aggressive data retention policy than management servers for instance where the config doesn't change often. Having different backup policies helps in putting those different workload types in categories.
* **Immutability and encryption**: Backup retention policies should also dictate how the data is stored, whether it is encrypted or stored on a repository in an immutable state for instance.

On a larger and less specific scale, the role of a backup retention policy is also to help admins know what data can be deleted for instance in order to keep data sprawl in check, sort of the application of data lifecycle management.

## Wrap up

As an VMware engineer, data has always been somewhere on my plate by it was never the main dish. However, being involved in the community and following IT industry trends closely, it is getting closer to every IT professional regardless of their field of expertise. Whether you are a VMware admin, an MSP, a Kubernetes SRE or an account manager, being aware of basic data lifecycle management and retention policies concepts is necessary to the old toolbelt. Nakivo Backup and Replication offers all the bells and whistles to address these concerns.