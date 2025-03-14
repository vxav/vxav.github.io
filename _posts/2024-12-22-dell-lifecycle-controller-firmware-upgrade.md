---
layout: post
title: Dell Firmware upgrade with Lifecycle Controller
date: 2024-12-22T11:42:20.305Z
nobanner: "yes"
fb-img: null
---

During my years as a VMware sysadmin I got to do quite a bit of hardware work with firmware, drivers, compatibility and so on. But since I transition to the cloud native world this part of IT has moved further away from me for obvious reasons. Now lately I am getting back into this as part of a project at work around bare-metal and I figured I would use my Dell R430 for the first POC. Which includes upgrading firmware on the machine.

I used to have a VM with Dell OpenManage Enterprise to upgrade everything quick and easy but I wiped my ESXi disk and I couldn't really be bothered reinstalling it since this machine will be wiped completely and I won't be using ESXi at all so I thought I'd just upgrade the firmware manually. 

## Dell firmware upgrade with Lifecycle Controller

There are [several methods](https://www.dell.com/support/kbdoc/en-us/000128194/updating-firmware-and-drivers-on-dell-emc-poweredge-servers) to upgrade firmware on a Dell server but I went with the Dell Lifecycle Controller option for the sake of simplicity. We simply boot on he lifecycle controller and pull all the firmware from the Dell website.

I just wanted to run through the steps here because the online documentation is a bit lacking and I'm pretty sure I will have forgotten all about it by the next time I want to do this again.

- First, let's boot in the lifecycle controller by pressing F10

![Dell lfc](/img/firmware1.png)

- The prerequisite for all this is to have internet access so you'll need to go into settings and make sure the server has an IP either statically or via DHCP.

- Select `Network Share`.

![dell lfc network shrare](/img/firmware2.png)

- Select `HTTPS` and type in the Dell website where the catalog is stored: `dl.dell.com`. 

Note that many online resource will mention the _downloads_ subdomain but this won't work as Dell seems to have sneakily changed it at some point. Full disclosure, I found this information from a [random post on the Dell forum](https://www.dell.com/community/en/conversations/poweredge-hardware-general/idrac-78-cannot-download-firmware-updates-via-lifecycle-lcc-using-online-catalog/647f7fe4f4ccf8a8dee793c6?commentId=647fa32cf4ccf8a8de8f9150) (thanks Dell for the doc...).

![dell download firmware website](/img/firmware3.png)

- Make sure it still work by using the `Check connection` feature and accept the invalid certificate.

![test connection lfc](/img/firmware4.png)

- If everything goes to plan you should now have a list of firmware you can upgrade.

![firmware list lfc](/img/firmware5.png)

- After you hit apply your firmware should start getting downloaded and installed onto your system.

![firmware install lfc](/img/firmware6.png)

---

On a side note, if you are looking for options on how to **protect your data**, [Nakivo Backup & Replication](https://www.nakivo.com/) offers capabilities to back up data from many types of workloads which can be recovered in case of a data loss event.

[![nakivo backup](/img/2022-10-26-13-45-41.png)](https://www.nakivo.com)
