---
title: "Ceph Lab"
date: 2020-10-07T17:09:19Z
draft: false
asciinema: false
description: "Building a Ceph test-lab"
categories: ["Lab","Ceph"]
tags: ['Lab','Ceph','Storage']
---
## Introduction

I'll quite often need to spin up a quick ceph cluster in order to test a particular piece of functionality

Here i've broken down the steps towards setting up a quick Ceph cluster utilising Virtual Machines on a libvirt capable host.  If you don't want to follow through step by step and just want to deploy the cluster - jump to [QuickStart]({{<relref "#QuickStart" >}})

## Target 

We're aiming for a fully functional Ceph cluster with the ability to utilise block, object and file services from.

* 1 Virtual-machine to act as our ceph bastion and test-client box
* 3 Virtual-machines running the appropiate ceph-daemons
* All of the above running a seperate network to keep things tidy and organised
* We'll end up with the following at the end

|Hostname   |IP Address   |
|---|---|
|ceph-bastion   | 172.16.200.10  |
|ceph-01   | 172.16.200.20  |
|ceph-02   | 172.16.200.21  |
|ceph-03   | 172.16.200.22  |


## Pre-requisites

* A host - either physical or virtual that is capable of running virtual machines (N.B You'll need nested virtualisation if using a VM).  I'm using a 64GB VM with nested virtualisation.  You can verify virtualisation is enabled by checking for extensions from the cmdline.  By checking the output for the exsitence of svm or vmx - as below you can confirmation virtualisation is enabled.  The below output is from my VM showing the vmx capability.

```bash
root@dev ceph-lab]# grep -E 'svm|vmx' /proc/cpuinfo
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl xtopology cpuid tsc_known_freq pni pclmulqdq vmx ssse3 cx16 pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cpuid_fault pti ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust smep erms xsaveopt arat umip arch_capabilities
```

* Libvirt installed and available to your user
* RHEL 8.2 QCOW Image.  This is available from the Red Hat download portal [here](https://access.redhat.com/downloads/content/479/ver=/rhel---8/8.2/x86_64/product-software).  I'm specifically using rhel-8.2-x86_64-kvm.qcow2.  If you don't have a Red Hat account - you can sign-up for a free developer account for non-production usage - more information at this page - [Red Hat Developer Subscription](https://developers.redhat.com/articles/getting-red-hat-developer-subscription-what-rhel-users-need-know).
  Download the image and store it in */var/lib/libvirt/images*

```bash
# ls -l /var/lib/libvirt/images
total 1131968
-rw-rw-r--. 1 qemu qemu 1159135232 Sep 30 18:19 rhel-8.2-x86_64-kvm.qcow2

# sha512sum /var/lib/libvirt/images/rhel-8.2-x86_64-kvm.qcow2 
1f3cf0cb62931cb13b509da1ee044c1aa07506eaf0dfb77bb673b6d3f56a38e1a3359475611437d82c4fd18c60805b22ae08ab4146ac86cbbd018e4273066bb3  /var/lib/libvirt/images/rhel-8.2-x86_64-kvm.qcow2
```


## Create SSH Key

Since we're working on a new dev box - if you don't already have an ssh-key generate a default one.  We'll use this later to access the lab we build.  Feel free to swap in an existing key - it's used later on when we inject paramters into the VM.

```bash
# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:zfFsmOnL2+OvG9Cx4XfEH6pSjsT7BrR8tPhe76qTBEo root@dev.lab.flumpy.net
```

## Checkout Lab repository

I've created a series of static configuration files to cover the hosts file and networking configuration.  We'll use these to embed into the virtual machines as we build them.

Clone the git repository into a local directory for usage later.  Let's also set an environment variable so we can reference this location later.

```bash
# git clone https://github.com/jameswilkins/lab-parts.git
Cloning into 'lab-parts'...
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 18 (delta 3), reused 18 (delta 3), pack-reused 0
Unpacking objects: 100% (18/18), done.

# GIT_LAB=/tmp/lab-parts
```

## Define custom network within libvirt

We'll make use of a custom network to seperate everything away nicely

Custom networks within lib-virt can be defined via an xml file 

```xml
<network>
  <name>ceph-lab</name>
  <bridge name="virbr666"/>
  <forward mode="nat"/>
  <ip address="172.16.200.1" netmask="255.255.255.0">
    <dhcp>
      <range start="172.16.200.10" end="172.16.200.249"/>
    </dhcp>
  </ip>
</network>
```

This will create a new bridge interface (virbr666) with an ip address 0f 172.16.200.1.  We allocate .10 to .249 for usage by DHCP.


Configure within libvirt by doing
```bash

[root@dev tmp]# virsh net-define $GIT_LAB/ceph-lab/network/ceph-lab.xml
Network ceph-lab defined from /tmp/lab-parts/ceph-lab/network/ceph-lab.xml

[root@dev tmp]# virsh net-start ceph-lab
Network ceph-lab started

[root@dev tmp]# virsh net-autostart ceph-lab
Network ceph-lab marked as autostarted

```

And then we can verify the bridge is up and running

```bash
[root@dev tmp]# ip ad sh virbr666
35: virbr666: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:4a:e1:da brd ff:ff:ff:ff:ff:ff
    inet 172.16.200.1/24 brd 172.16.200.255 scope global virbr666
       valid_lft forever preferred_lft forever
```

For good measure we can ping the default-gateway to confirm connectivity

```bash
[root@dev tmp]# ping -c 2 172.16.200.1
PING 172.16.200.1 (172.16.200.1) 56(84) bytes of data.
64 bytes from 172.16.200.1: icmp_seq=1 ttl=64 time=0.083 ms
64 bytes from 172.16.200.1: icmp_seq=2 ttl=64 time=0.088 ms
```

## Create our VMs

Next we'll configure a series of VM's to house our ceph lab.  We'll be storing the VM's in /var/lib/libvirt/images.

We're using virt-customize from the libguestfs project in order to inject paramters into our virtual machines.  We also use some of the config files stored in git to assist with configuration.

### Ceph-Bastion - 172.16.200.10

Create empty base-OS disk

```bash
# qemu-img create -f qcow2 /var/lib/libvirt/images/ceph-bastion.qcow2 60G
Formatting '/var/lib/libvirt/images/ceph-bastion.qcow2', fmt=qcow2 size=64424509440 cluster_size=65536 lazy_refcounts=off refcount_bits=16
```

Copy the RHEL82 template into our new base-OS disk.  This will also resize /dev/sda3 (/) to fit our newly sized base-OS disk.

```bash
# virt-resize --expand /dev/sda3 /var/lib/libvirt/images/rhel-8.2-x86_64-kvm.qcow2 /var/lib/libvirt/images/ceph-bastion.qcow2
```

Expected Output:
```bash
[   0.0] Examining /var/lib/libvirt/images/rhel-8.2-x86_64-kvm.qcow2
**********

Summary of changes:

/dev/sda1: This partition will be left alone.

/dev/sda2: This partition will be left alone.

/dev/sda3: This partition will be resized from 9.9G to 59.9G.  The
filesystem xfs on /dev/sda3 will be expanded using the ‘xfs_growfs’
method.

**********
[   6.1] Setting up initial partition table on /var/lib/libvirt/images/ceph-bastion.qcow2
[  17.8] Copying /dev/sda1
[  17.8] Copying /dev/sda2
[  18.2] Copying /dev/sda3
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[  72.7] Expanding /dev/sda3 using the ‘xfs_growfs’ method

Resize operation completed with no errors.  Before deleting the old disk,
carefully check that the resized disk boots and works correctly.
```

Now we move onto injecting the network & ssh configuration information.

```bash
# virt-customize -a /var/lib/libvirt/images/ceph-bastion.qcow2 \
   --root-password password:password \
   --uninstall cloud-init \
   --hostname ceph-bastion.ceph.lab \
   --copy-in $GIT_LAB/ceph-lab/config/hosts:/etc/ \
   --copy-in $GIT_LAB/ceph-lab/config/ceph-bastion/ifcfg-eth0:/etc/sysconfig/network-scripts/ \
   --ssh-inject root:file:/root/.ssh/id_rsa.pub \
   --selinux-relabel
```

Expected Output

```bash
[   0.0] Examining the guest ...
[   7.6] Setting a random seed
[   7.6] Setting the machine ID in /etc/machine-id
[   7.6] Uninstalling packages: cloud-init
[  13.1] Setting the hostname: ceph-bastion.ceph.lab
[  13.1] Copying: /home/james/sites/lab-parts/ceph-lab/config/hosts to /etc/
[  13.1] Copying: /home/james/sites/lab-parts/ceph-lab/config/ceph-bastion/ifcfg-eth0 to /etc/sysconfig/network-scripts/
[  13.1] SSH key inject: root
[  15.2] Setting passwords
[  17.1] SELinux relabelling
[  32.7] Finishing off
```

Then we'll use virt-install to define the parameters for our VM and store this in an xml file.  

{{< admonition type=tip title="Expected Output" open=true >}}
This command won't generate any console output - we're creating an xml file at this stage
{{< /admonition >}}

```bash
# virt-install --name ceph-bastion\
   --virt-type kvm --memory 2048 --vcpus 2 \
   --boot hd,menu=on \
   --disk path=/var/lib/libvirt/images/ceph-bastion.qcow2,device=disk,bus=scsi,discard='unmap' \
   --graphics none \
   --os-type Linux --os-variant rhel8.2 \
   --network network:ceph-lab \
   --noautoconsole \
   --dry-run --print-xml >ceph-bastion.xml
```

The XML file contains the neccessary information about our virtual machine

```xml
<domain type="kvm">
  <name>ceph-bastion</name>
  <uuid>47c5bf95-3d8a-40e1-91a2-e3cfb34bdebd</uuid>
  <metadata>
    <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
      <libosinfo:os id="http://redhat.com/rhel/8.2"/>
    </libosinfo:libosinfo>
  </metadata>
  <memory>2097152</memory>
  <currentMemory>2097152</currentMemory>
  <vcpu>2</vcpu>
  <os>
    <type arch="x86_64" machine="q35">hvm</type>
    <boot dev="hd"/>
    <bootmenu enable="yes"/>
  </os>
  <features>
    <acpi/>
    <apic/>
  </features>
  <cpu mode="host-model"/>
  <clock offset="utc">
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="pit" tickpolicy="delay"/>
    <timer name="hpet" present="no"/>
  </clock>
  <pm>
    <suspend-to-mem enabled="no"/>
    <suspend-to-disk enabled="no"/>
  </pm>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>
    <disk type="file" device="disk">
      <driver name="qemu" type="qcow2" discard="unmap"/>
      <source file="/var/lib/libvirt/images/ceph-bastion.qcow2"/>
      <target dev="sda" bus="scsi"/>
    </disk>
    <controller type="usb" index="0" model="qemu-xhci" ports="15"/>
    <controller type="scsi" index="0" model="virtio-scsi"/>
    <interface type="network">
      <source network="ceph-lab"/>
      <mac address="52:54:00:92:ab:e0"/>
      <model type="virtio"/>
    </interface>
    <console type="pty"/>
    <channel type="unix">
      <source mode="bind"/>
      <target type="virtio" name="org.qemu.guest_agent.0"/>
    </channel>
    <memballoon model="virtio"/>
    <rng model="virtio">
      <backend model="random">/dev/urandom</backend>
    </rng>
  </devices>
</domain>

```

We then inject the XMl file into libvirt and start our first VM.

```bash
# virsh define ./ceph-bastion.xml
Domain ceph-bastion defined from ./ceph-bastion.xml

# virsh start ceph-bastion
Domain ceph-bastion started
```

After a short period of time the ceph-bastion box will come online

```bash
# ping 172.16.200.10
PING 172.16.200.10 (172.16.200.10) 56(84) bytes of data.
64 bytes from 172.16.200.10: icmp_seq=1 ttl=64 time=0.321 ms
64 bytes from 172.16.200.10: icmp_seq=2 ttl=64 time=0.239 ms
```

### Ceph Nodes
Now we rinse and repeat for the next three virtual machines in our list.

```bash
for i in {01,02,03}; 
do
  qemu-img create -f qcow2 /var/lib/libvirt/images/ceph-$i.qcow2 60G
  qemu-img create -f qcow2 /var/lib/libvirt/images/ceph-$i-diska.qcow2 10G
  qemu-img create -f qcow2 /var/lib/libvirt/images/ceph-$i-diskb.qcow2 10G
done


for i in {01,02,03}; 
do
  virt-resize --expand /dev/sda3 /var/lib/libvirt/images/rhel-8.2-x86_64-kvm.qcow2 /var/lib/libvirt/images/ceph-$i.qcow2
  virt-customize -a /var/lib/libvirt/images/ceph-$i.qcow2 \
    --root-password password:password \
    --uninstall cloud-init \
    --hostname ceph-$i.ceph.lab \
    --copy-in $GIT_LAB/ceph-lab/config/hosts:/etc/ \
    --copy-in $GIT_LAB/ceph-lab/config/ceph-$i/ifcfg-eth0:/etc/sysconfig/network-scripts/ \
    --ssh-inject root:file:/root/.ssh/id_rsa.pub \
    --selinux-relabel
  virt-install --name ceph-$i \
    --virt-type kvm --memory 4096 --vcpus 2 \
    --boot hd,menu=on \
    --disk path=/var/lib/libvirt/images/ceph-$i.qcow2,device=disk,bus=scsi,discard='unmap' \
    --disk path=/var/lib/libvirt/images/ceph-$i-diska.qcow2,bus=scsi,discard='unmap' \
    --disk path=/var/lib/libvirt/images/ceph-$i-diskb.qcow2,bus=scsi,discard='unmap' \
    --graphics none \
    --os-type Linux --os-variant rhel8.2 \
    --network network:ceph-lab \
    --noautoconsole \
    --dry-run --print-xml >ceph-$i.xml
  virsh define ./ceph-$i.xml
  virsh start ceph-$i


done

```

## Verification

Once complete, let's verify we have the expected machines.

```bash
# virsh list
 Id    Name                           State
----------------------------------------------------
 96    ceph-bastion                   running
 100   ceph-01                        running
 104   ceph-02                        running
 108   ceph-03                        running
```

You should also find that ssh works to the bastion server.

```bash
[root@dev tmp]# ssh root@172.16.200.10
Activate the web console with: systemctl enable --now cockpit.socket

This system is not registered to Red Hat Insights. See https://cloud.redhat.com/
To register this system, run: insights-client --register

Last login: Mon Oct 12 14:24:40 2020 from 172.16.200.1
[root@ceph-bastion ~]#
```
## QuickStart / Lazy Method {#QuickStart}

If you don't want to follow the instructions above, i've included a simple bash script within the repository that will perform all of these steps for you.

Please read the script (as you should with anything you download from the internet!) then make modifications at the top to match your expected paths.

Installation with script:

<script id="asciicast-z4CPB8xsv6pTtiyGUuemaczGO" src="https://asciinema.org/a/z4CPB8xsv6pTtiyGUuemaczGO.js" async></script>

## Next Steps

Now you're ready to move onto the next part of the instructions and [install Ceph]({{<relref "#install-ceph.md" >}})



