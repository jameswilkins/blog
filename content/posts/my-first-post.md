---
title: "My First Post"
date: 2020-10-07T07:57:23Z
draft: true
toc: true
asciinema: true
images:
tags:
  - ceph
---

This is some test


```bash

root@workstation ceph]# ssh root@ceph-osd01 "systemctl --all"|grep ceph-osd|awk '{print $1}'|xargs ssh root@ceph-osd01 systemctl start
[root@workstation ceph]# md5sum /etc/yum.repos.d/open.repo
986981f26bf36f5b5e8d6f0ce5701f39  /etc/yum.repos.d/open.repo
[root@workstation ceph]# rados put -p test-pool open.repo /etc/yum.repos.d/open.repo
[root@workstation ceph]# ceph osd map test-pool open.repo
osdmap e523 pool 'test-pool' (15) object 'open.repo' -> pg 15.44fb73c5 (15.5) -> up ([5,1], p5) acting ([5,1], p5)

```






test 2

* test.png


