---
layout: post
title: Troubleshooting broken K3s api access
date: 2024-12-26T10:42:20.305Z
nobanner: "yes"
fb-img: null
---

While I work with vanilla Kubernetes (K8s) in my day job, I run a single node K3s cluster at home for my personal services like Home Assistant, Z2M, Unifi controller and so on. I mentioned this setup a little bit in [this blog](2024-03-30-argo-events-and-renovate.md). Just like Kubernetes, the default port for the K3s API is 6443, which is exposed by the `k3s` service. 

Now while working on bare metal stuff, I wanted a quick way to load balance 6443 to another Kubernetes cluster so I created an endpoint with a LAN IP and created a load balancer service on 6443 targetting this endpoint... I then lost access to the K3s API and that was a big facepalm moment. The service took over 6443 in place of the K3s service.

I will describe here the troubleshooting steps as I found this to be an interesting excercise.

## How to gain access back to the Kubernetes API

I lost access to Kubernetes so I couldn't try to delete the service to revert this change. The k3s service was running but not replying on port 6443.

In order to gain access back temporarily, I changed the default port from 6443 to 6446 (arbitrary).

- Stop the `k3s` service.

```
systemctl stop k3s`
```

- Edit the K3s service to use a different port.

```
vi /etc/systemd/system/k3s.service

...
ExecStart=/usr/local/bin/k3s \
    server --https-listen-port 6446 \
```

- Restart the `k3s` service.

```
systemctl start k3s`
```

- Modify the kubeconfig on my machine to target the new port.

```
vi $KUBECONFIG

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ***
    server: https://192.168.1.233:6446
  name: default
...

At this point I could use kubectl again to try and troubleshoot further.

## Fix the root cause

My first thought was to delete the the service I had created, which I did. I noticed that the pod associated with that service did not get deleted, which led me to think that things were not going fine but I deleted it anyway. I then changed the port back to 6443 but no luck.

At this point, nothing was listening on port 6443. Even if I manually listened on 6443 with netcat, I couldn't get to it. Which led me to suspect iptables.

- Check `iptables` rules. K3s has its own embedded implementation of kube-proxy which manages iptables rules to deal with services and networking. Since iptables is configured in nftables mode (I discovered this in the process), I will skip the iptables commands.

Here I look for rules with 6443 and grep previous lines to find which [chain](https://unix.stackexchange.com/questions/506729/what-is-a-chain-in-iptables) they are in. 

```
> nft list ruleset | grep 6443 -B10

...
	chain KUBE-SEP-MKZ5TTJEI3R5D5SV {
		ip saddr 192.168.1.233  counter packets 2 bytes 120 jump KUBE-MARK-MASQ
		meta l4proto tcp   counter packets 53 bytes 3180 dnat to 192.168.1.233:6443
--
...
	chain CNI-DN-3499088e1adba268a4e0c {
		meta l4proto tcp ip saddr 10.42.0.0/24 tcp dport 6443 counter packets 0 bytes 0 jump CNI-HOSTPORT-SETMARK
		meta l4proto tcp ip saddr 127.0.0.1 tcp dport 6443 counter packets 272 bytes 16264 jump CNI-HOSTPORT-SETMARK
		meta l4proto tcp tcp dport 6443 counter packets 273 bytes 16328 dnat to 10.42.0.244:6443
```

The first rule in the `KUBE-SEP-MKZ5TTJEI3R5D5SV` chain didn't bother me but the second ones in the `CNI-DN-3499088e1adba268a4e0c` chain seem to refer to a dnat of 6443 traffic to `10.42.0.244`, which is a pod cidr IP.

- Delete all rules in the `CNI-DN-3499088e1adba268a4e0c` chain.

```
> nft flush chain nat CNI-DN-3499088e1adba268a4e0c
```

After I did that, the K3s API started responding again on 6443, awesome. Yet I thought I would reboot the machine just to make sure. After a reboot, the rule was back in and "blocking" traffic to 6443 so something is recreating it.

I quickly checked the nftables config just to make sure but, as expected, nothing is in it.

```
cat /etc/nftables.conf

#!/usr/sbin/nft -f

flush ruleset

table inet filter {
	chain input {
		type filter hook input priority 0;
	}
	chain forward {
		type filter hook forward priority 0;
	}
	chain output {
		type filter hook output priority 0;
	}
}
```

- I checked in Kubernetes for the pod associated with the service and it was back indeed. I tailed the logs of that service and the answer was right there. 

```
> kubectl logs -n kube-system                 svclb-kubeapi-loadbalancer-8e92d16f-htf6q

[INFO]  nft mode detected
+ trap exit TERM INT
+ BIN_DIR=/sbin
+ check_iptables_mode
+ set +e
+ lsmod
+ grep -qF nf_tables
+ '[' 0 '=' 0 ]
+ mode=nft
+ set -e
+ info 'nft mode detected'
+ set_nft
+ ln -sf /sbin/xtables-nft-multi /sbin/iptables
+ ln -sf /sbin/xtables-nft-multi /sbin/iptables-save
+ ln -sf /sbin/xtables-nft-multi /sbin/iptables-restore
+ ln -sf /sbin/xtables-nft-multi /sbin/ip6tables
+ start_proxy
+ + echo 0.0.0.0/0
grep -Eq :
+ iptables -t filter -I FORWARD -s 0.0.0.0/0 -p TCP --dport 6443 -j ACCEPT
+ grep -Eq :
+ echo 10.43.49.200
+ cat /proc/sys/net/ipv4/ip_forward
+ '[' 1 '==' 1 ]
+ iptables -t filter -A FORWARD -d 10.43.49.200/32 -p TCP --dport 6443 -j DROP
+ iptables -t nat -I PREROUTING -p TCP --dport 6443 -j DNAT --to 10.43.49.200:6443
+ iptables -t nat -I POSTROUTING -d 10.43.49.200/32 -p TCP -j MASQUERADE
+ '[' '!' -e /pause ]
+ mkfifo /pause
```

In the default implementation of K3s, when you create a service of type load balancer, a pod is created which installs iptables rules (implemented as nftables by default). So deleting the pod won't do anything. When I restarted the machine, that pod started and reinstalled the rules.

- Check where this pod is created from and delete the resource.

```
> kubectl describe pod -n kube-system svclb-kubeapi-loadbalancer-8e92d16f-htf6q | grep -i "controlled"
Controlled By:  DaemonSet/svclb-kubeapi-loadbalancer-8e92d16f
```

- Delete the `daemonset`.

```
> kubectl delete ds -n kube-system svclb-kubeapi-loadbalancer-8e92d16f
daemonset.apps "svclb-kubeapi-loadbalancer-8e92d16f" deleted
```

After that, I delete the nftables rule again, restarted the machine and everything was working as expected again.

## Wrap up

Obviously, creating a service of type LB on port 6443 was a mistake but let's call this a momentary brain freeze. After that, deleting the service should have led to k3s cleaning up everything but I suspect the non-default port and possibly other iptables rules may have blocked this somehow.

The easy fix I thought of originally was to restore etcd but I also learned that the default implementation of K3s uses SQLite as a datastore. Which can be backed up by copying the db but it was never done.

In the end it was a very positive problem to dig into because I learned a lot about K3s since I didn't look into it too deeply when I installed it (mostly for its lightweight nature compared to kubelet and friends).

Next steps to improve my setup:

1. Switch from SQLite to etcd for the K3s datastore.
2. Configure scheduled etcd snapshots.
3. Replace traefik with Cilium in ebpf mode.