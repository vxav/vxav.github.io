---
layout: post
title: Horizon View shambles
DATE: 
date: '2019-01-17T08:02:13.000+00:00'

---
I started working with Horizon View for a customer a little less than a year ago. I had never used the product before and boy was life sweet. I will consider this post a mix of self therapy and archives (I know, makes sense to me). Although before ranting at Horizon View it is worth mentioning that the environment in which it is deployed is pretty complex.

I will keep updating this post with new issues that worth the tale so I hope not a lot... If you encountered some of them or have information/ideas/theories **please share in the comments!**

### Issue #1: A taste of apocalypse

* **_Root cause:_** _Horizon View: 7.4.0 - 7400497 buggy version_

[The first issue](https://communities.vmware.com/thread/591103 "Call of duty: Horizon View") I encountered was really bad. Whenever the connection servers would lose connectivity to vCenter, even for a short amount of time, everything would go awol. Errors all over the place, no provisioning, a real battlefield with body parts everywhere. Proper messy. The only way to fix it was to shut down everything in the right order and power it back on. Not "ideal" in production... VMware later provided us with a hotfix (_7.4.0 8741716_) and it's been ok since then. Fingers crossed.

### Issue #2: Pickaboo

* **_Root cause:_** _Internal SAS storage controller driver/firmware issue_

Then we ran into hardware problems with customized VSAN ready nodes where they would just disconnect from vCenter causing a bunch of problems. What happens is that somehow the local ESXi boot disk goes missing (hence the pickaboo). This event would cause ESXi to try to bring the disk back online over and over again, eventually using up all the allocated memory causing hostd and vpxa to crash dramatically. At least that's my best theory that the VMware support confirmed with a (not convincing at all) "It makes sense".

Funnily enough VSAN would still work fine and the VMs on it too so there's that... ESXi could still run as it's loaded in RAM but no logs persist as they are store on local disk. The only way to troubleshoot was the real time vmkernel log in the DCUI (alt+F12). After reviewing the logs I found a lot of records pointing the finger at the storage controller of the 2 dedicated boot disks. (This controller is the "customized" bit of the ready node - certified by the vendor - or is it?)

VMware confirmed and recommended to upgrade the firmware driver of this controller to the latest version which I did. The issue hasn't come back since but it's not been long so I'm not putting my money on it - This is a case of wait and see.

**_EDIT:_** The upgraded firmware/driver fixed this issue.

### Issue #3: Because why not

* **_Root cause:_** _Unknown - Upgrade recommended by VMware support_

This third riot is rather fresh. I was happily updating the VMtools and compatibility of the connection server VMs in the second pod after our 6.5 migration. One by one. Every time I wait for it to come back to green in the admin console and I move on to the next one. I was laughing.

Then I moved to the other pod. Same process, everything goes smoothly. There's only one that I don't upgrade as there's a bunch of users connected in tunneling mode, no biggy. Thought to myself "Cool almost done" but then the "Problem vCenter VMs" count started incrementing along with error events but all components were green... oh oh. VMs provisioned would go in "Provisioning error" and VM deletion in "Error (missing)". Both suggesting an authentication problem, weird. I even got some of them in "Error" with a magnificent error message _'java.lang.IllegalArgumentException: cachedVM == null'_. Thanks, so what am I supposed to do with that exactly? I never found it on google, only a few hits that resembled it that suggested Invalid object in the ADAM database. I looked at it but couldn't find any oddity (although I didn't know what I was after so...). I checked the replication with [repadmin](https://kb.vmware.com/s/article/1021805 "KB1021805") which showed that everything was OK.

Luckily the second pod was fine so there was no doomsday kind of emergency but I didn't sleep well. This time we managed to bring everything back online by shutting down all the connection servers in the pod one by one, and then powering them back on. It is still a mystery to me why things got bad and why the "let's call it a reset" of the connection servers fixed it... I opened a case with VMware and will keep this post updated if/when new relevant information comes in.

**_EDIT:_** VMware's conclusion is to upgrade Horizon because 7.4.0 is a "build with problems".

### Issue #4: Pod racing upgrade (13/05/2019)

* **_Root cause:_** _Unknown - Fixed magically_

I'm trying to be creative with the title of my issues and I like Star Wars so... (you have to laugh or else you'll cry). Anyway, this issue occurred after upgrading a pod in our cloud pod architecture. It is actually not that bad and I was half expecting it to be honest.

Before doing an upgrade I like to roam the VMTN forums of the components involved for horror stories. I did just that in preparation for our upgrade from 7.4.0 to 7.8.0 and stumbled upon [this thread](https://communities.vmware.com/thread/604479). Some users reported not being able to display the global entitlements in the UI of the upgraded pod, it would just show a popup with "Server Error". Upgrading the second pod just brings the problem to both pods. It seems that the cli tool lmvutil still works though and people fixed the issue by creating a "fake" entitlement (no idea what that means, I don't call that a fix anyway).

At that point I wasn't over confident in upgrading so we asked our contacts over at VMware. They checked in depth internally with engineers and stuff and assured us they couldn't find anything regarding this issue so "good to go"... right, hmm ok, I guess? So we upgraded one pod. Guess what? "Server Error". Nice...

So far we held back upgrading the second pod and opened a ticket with VMware. I will update here when resolved.

**_EDIT:_** Several days after the upgrade of the first pod the GUI magically worked again. It was  a couple hours after I collected a support bundle on one of the connection servers but VMware support doesn't think it is related. We then upgraded the second pod and everything was ok straight away. No root cause found.