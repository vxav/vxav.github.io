---
layout: post
title: Log4j workaround for vCenter automated script (VMware)
DATE: 
subtitle: ''
metadescription: Protect your vCenter appliance from Log4j with this easy to apply
  python script distributed by VMware.
image: ''

---
> _EDIT 17/12/2021: A new script is required to remediate **Analytics Service and CM Service**. You now need to download remove_log4jclass.py available in_ [_KB87081_](https://kb.vmware.com/s/article/87081?lang=en_US#vCenter67) _and run it at the end after running vmsa-2021-0028-kb87081.py._

If you don't know about the Log4j vulnerability yet (CVE score 10/10), it probably means you don't need to worry about it (or live in a cave). Anyway, I won't bother writing an introduction on Log4j in this blog since it has been literally everywhere on the internet. However, VMware products are impacted by it since they use Java quite a lot, this is detailed in [VMSA-2021-0028](https://www.vmware.com/security/advisories/VMSA-2021-0028.html). You'll find links to workarounds for each product, a patch will also be added when they release a fixed version.

Now, the Log4j vCenter workaround is described in [KB87081 ](https://kb.vmware.com/s/article/87081#vCenter67)but there is also a Python script that will automate all of that for you. And don't worry the script is distributed by VMware themselves.

* Head over to [KB87088 ](https://kb.vmware.com/s/article/87088)and download the script.

![](/img/log4j1.png)

* Copy the script to your vCenter appliance in /tmp. Either use WinSCP or copy/paste it in a file.
* Execute the script like so.

Note that you will have to confirm that you want to restart all the services so you may to ensure that this will not disrupt other ongoing operations.

    python /tmp/vmsa-2021-0028-kb87081.py

* This is the output on my vCenter 7 Update 3.

![](/img/log4j2.png)

* EDIT 17/12/2021: Download remove_log4jclass.py from KB87081 and run it to remediate the Analytics Service and CM Services.

![](/img/log4j3.png)

Regardless, I suggest you subscribe to those KBs and keep an eye out for new developments as things have been moving very fast over the last few days on this front.