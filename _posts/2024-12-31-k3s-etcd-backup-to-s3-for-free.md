---
layout: post
title: K3s etcd backup to S3 for free
date: 2024-12-31T10:00:00.305Z
nobanner: "yes"
fb-img: /img/backblaze-free-real-estate.png
---

The [incident I had recently](/2024-12-26-k3s-stuck-load-balancer) highlighted to me that I never set up regular backups of the Kubernetes datastore of my home K3s single-node cluster. As a result I converted K3s to use embedded etcd instead of the default SQLite as datastore backend, which allows me to configure backups (snapshots) of etcd to an S3 bucket. 

If your K3s doesn't use etcd as a datastore yet, you can convert it simply by adding the `--cluster-init` flag to the config file. [Documentation here](https://docs.k3s.io/datastore/ha-embedded#existing-single-node-clusters).

in this blog I will higlight how to set up an S3 bucket for free (no credit card required) on Backblaze and configure K3s to push snapshots to it. 

## Set up S3 bucket on Backblaze B2

When looking for S3 compatible endpoint providers, I looked at a few different options but I wanted one with a decent free tier and without the requirement for a credit card. Since I will only use it to store etcd backups I ended up settling for Backblaze B2 which includes the first 10GB for free, daily download cap of 1 GB and free data egress (also known as download) up to 3x average monthly storage amount.

- First create an account on [Backblaze B2 cloud storage](https://www.backblaze.com/sign-up/cloud-storage?referrer=getstarted). I chose the "EU Central" region since I am in France.

![backblaze account](/img/backblaze-1.png)

- Once you are logged into your account, go to `B2 Cloud Storage > Buckets > Create a Bucket`.

![backblaze bucket](/img/backblaze-2.png)

- Give it a sensible name, and configure it as you see fit. I made it private, encrypted and immutable since it will store etcd snapshots.

![backblaze bucket config](/img/backblaze-3.png)

- Then head to `Application Keys > Add a New Application Key`.

![backblaze app key](/img/backblaze-5.png)

- Give it a sensible name, select the name of the bucket we created with `Read and Write` access.

![backblaze app key create](/img/backblaze-6.png)

- Save the following fields somewhere safe (lastpass, onepassword...) as it will only appear once.
  - `keyID` is the S3 `access key`
  - `applicationKey` is the S3 `secret key`

![backblaze secrets](/img/backblaze-7.png)

## Create the S3 secret in Kubernetes

When configuring the backup of the [etcd snapshots to S3](https://docs.k3s.io/cli/etcd-snapshot#s3-configuration-secret-support), the S3 properties can be set either in the the K3s config file (`/etc/rancher/k3s/config.yaml` by default) or placed in a secret in the cluster itself. The latter is my preferred method as it means no secrets stored in plain-text.

- Create the following secret in the cluster.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: backblaze-etcd-backup-config
  namespace: kube-system
data:
  etcd-s3-access-key: <B2 bucket keyID>
  etcd-s3-bucket: "k3s-etcd-backups"
  etcd-s3-endpoint: <Find it in the properties of the bucket - e.g. s3.eu-central-003.backblazeb2.com>
  etcd-s3-region: "auto"
  etcd-s3-secret-key: <B2 bucket applicationKey>
```

## Configure K3s for etcd snapshots to S3

Now we need to tell K3s to enable S3 backups and to use that secret as source of information.

- Edit the K3s config file (`/etc/rancher/k3s/config.yaml` by default). Here is my configuration with comments:

```yaml
etcd-snapshot-dir: /var/lib/rancher/k3s/db/snapshots  # Default location, explicit for quick access
etcd-snapshot-retention: 10  # Keep only the last 10 snapshots
etcd-snapshot-schedule-cron: "0 4 * * *"  # Daily at 4AM
etcd-snapshot-compress: true  # Compress files as .zip (~5x smaller)
etcd-s3-config-secret: backblaze-etcd-backup-config  # Use Kubernetes secret we created previously
etcd-s3: true  # Enable backups to S3
```

- Restart K3s.

```sh
systemctl restart k3s
```

Now you should start seeing new files in your bucket every day.

![backblaze files](/img/backblaze-8.png)

You can also test manually from the `k3s` command line:

```sh
k3s etcd-snapshot save --s3 --etcd-s3-endpoint=s3.eu-central-003.backblazeb2.com --etcd-s3-bucket=k3s-etcd-backups --etcd-s3-access-key=*** --etcd-s3-secret-key=*** --etcd-s3-region=auto
```

---

On a side note, if you are looking for options on how to **protect your data**, [Nakivo Backup & Replication](https://www.nakivo.com/) offers capabilities to back up data from many types of workloads which can be recovered in case of a data loss event.

[![nakivo backup](/img/2022-10-26-13-45-41.png)](https://www.nakivo.com)