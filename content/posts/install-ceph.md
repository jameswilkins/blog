---
title: "Install Ceph"
date: 2020-10-12T19:39:44+01:00
draft: true
description: "Installation of Ceph"
categories: ["Lab","Ceph"]
tags: ['Lab','Ceph','Storage']
---

## Introduction

If you've followed along with the [previous]({{<relref "ceph-lab.md" >}}) guide then you'll have 4 machines ready to install Ceph onto.

|Hostname   |IP Address   |
|---|---|
|ceph-bastion   | 172.16.200.10  |
|ceph-01   | 172.16.200.20  |
|ceph-02   | 172.16.200.21  |
|ceph-03   | 172.16.200.22  |


{{< admonition type=tip title="Ceph Version" open=true >}}
This version of the guide focuses on the Red Hat version of Ceph Storage.  This requires an active subscription in order to access the necessary installation files.  A version of this lab that utilises the open-source ceph version will be available shortly.
{{< /admonition >}}


## Registration

We need to ensure our nodes are registered with the Red Hat Network in order to ensure access to the appropiate package repositories.

We'll use a short ansible playbook in order to streamline this part of the procedure.  The playbook will

* Register the 4 hosts with the Red Hat Network
* Subscribe to the appropiate subscription
* Ensure latest updates are applied 

The playbook will be run from our virtualisation host, so ensure you have ansible installed.



### Ansible Configuration

Configure up the vars.yml file with the appropiate information

rh_username - Your Red Hat Subscription Username
rh_password - Your Red Hat Subscription Password
rh_poolid - The PoolID for your subscription that has the Red Hat Ceph packages available

```bash
```


Foo Bar Foo
