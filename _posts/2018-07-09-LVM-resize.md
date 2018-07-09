---
layout: post
title: "How to increase an LVM Partition"
date:   2018-07-09
categories: [linux, IT, LVM]
---

Do you have finished all the space in you VM?
Yes for sure you can add a new Virtual Hard Disk on you VM in VirtualBox or in Proxmox or in VMWare, and have a new separate partition (like sdb) on your VM, and for example you'll can mount the log folder in this second "disk"; but if you are using an LVM Partition you can easily, ancee with out rebooting you VM, increare the LVM Partition adding the new disk in it


Below I will show you the steps to follow to complete this operation:
Let's check with the `pvdisplay` command that currently we have only one physical volume
{% highlight bash %}
user@vm $pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               rep-voce-vg
  PV Size               7,52 GiB / not usable 0
  Allocatable           yes (but full)
  PE Size               4,00 MiB
  Total PE              1925
  Free PE               0
  Allocated PE          1925
  PV UUID               nACRAt-5Gy2-p1I8-az72-CQtG-zAry-qfKisp
{% endhighlight %}

Let's create the primary partition on the second disk with the command `cfdisk /dev/sdX`: the partition must be PRIMARY, NOT BOOTABLE and 8E type (Lunux LVM).
And then with `fdisk -l /dev/sdb` command, let's check if all is correct
{% highlight bash %}
user@vm $fdisk -l /dev/sdb
Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x59a21acb

Device     Boot Start      End  Sectors Size Id Type
/dev/sdb1        2048 41943039 41940992  20G 8e Linux LVM
{% endhighlight %}

Let's create the Physical volume with `pvcreate /dev/sdX1` command.
{% highlight bash %}
user@vm $pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
{% endhighlight %}

Let's re-check now the Physical volumes
{% highlight bash %}
user@vm $pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               rep-voce-vg
  PV Size               7,52 GiB / not usable 0
  Allocatable           yes (but full)
  PE Size               4,00 MiB
  Total PE              1925
  Free PE               0
  Allocated PE          1925
  PV UUID               nACRAt-5Gy2-p1I8-az72-CQtG-zAry-qfKisp

  --- Physical volume ---
  PV Name               /dev/sdb1
  VG Name               rep-voce-vg
  PV Size               20,00 GiB / not usable 3,00 MiB
  Allocatable           yes (but full)
  PE Size               4,00 MiB
  Total PE              5119
  Free PE               0
  Allocated PE          5119
  PV UUID               cdF06U-56lm-wyXN-Kgnj-4Ci5-P9gu-PuKvVi
{% endhighlight %}

Let's check the name of the volume group (VG Name) with `vgdisplay` command
{% highlight bash %}
user@vm $vgdisplay
  --- Volume group ---
  VG Name               rep-voce-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  7
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               7,52 GiB
  PE Size               4,00 MiB
  Total PE              1925
  Alloc PE / Size       1925 / 7,52 GiB
  Free  PE / Size       0 / 0
  VG UUID               jCHDDR-sXu0-rWoc-J1nh-XLzJ-IkWd-VdmhXr
{% endhighlight %}

Let's extend the volume group with the command `vgextend VGname /dev/sdX1` command
{% highlight bash %}
user@vm $vgextend rep-voce-vg /dev/sdb1
  Volume group "rep-voce-vg" successfully extended
{% endhighlight %}

Let's check now the path of the Logical volume (LV Path) with `lvdisplay` command
{% highlight bash %}
user@vm $lvdisplay
  --- Logical volume ---
  LV Path                /dev/rep-voce-vg/root
  LV Name                root
  VG Name                rep-voce-vg
  LV UUID                IFSP9d-Yixt-14Ya-SyDx-Z8vk-RIoC-hcgnad
  LV Write Access        read/write
  LV Creation host, time rep-voce, 2018-02-28 12:25:03 +0100
  LV Status              available
  # open                 1
  LV Size                5,52 GiB
  Current LE             1413
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
{% endhighlight %}

Let's extend the logical volume with `lvextend LVpath /dev/sdX1` command
{% highlight bash %}
user@vm $lvextend /dev/rep-voce-vg/root /dev/sdb1
  Size of logical volume rep-voce-vg/root changed from 5,52 GiB (1413 extents) to 25,52 GiB (6532 extents).
  Logical volume root successfully resized.
{% endhighlight %}

Let's check the name of the device mapper with `df -h` command
{% highlight bash %}
user@vm $df -h
Filesystem                      Size  Used Avail Use% Mounted on
udev                            972M     0  972M   0% /dev
tmpfs                           200M   30M  171M  15% /run
/dev/mapper/rep--voce--vg-root  5,4G  5,1G     0 100% /
{% endhighlight %}

Let's now resize file system with with last 3 command
{% highlight bash %}
user@vm $
sync
resize2fs /dev/mapper/rep--voce--vg-root
sync
{% endhighlight %}

And then with `df -h` you can verify the new partition size
{% highlight bash %}
user@vm $
Filesystem                      Size  Used Avail Use% Mounted on
udev                            972M     0  972M   0% /dev
tmpfs                           200M   30M  171M  15% /run
/dev/mapper/rep--voce--vg-root   25G  5,1G   19G  22% /
{% endhighlight %}

Thanks for reading!
