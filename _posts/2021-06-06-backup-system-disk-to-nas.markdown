---
layout: post
title: Backup System Disk to NAS
---

又入手了一个NUC，这次买的是我本来最想买的NUC11PAHi5。上次那个NUC定位为一个稳定的Linux开发环境，这次的NUC定位是一个不稳定的体验测试环境，比如说玩下破解游戏，破解就意味着很大可能带有后门或者病毒。在我主力系统上我是不会安装破解软件的。

我把上个NUC的一条16GB内存拆到这台NUC上，老电脑的256GB的SATA SSD安装到NUC上，就可以安装系统了。

安装系统，驱动，设置账号等，这些操作每次重装系统都会浪费不少时间，而这台NUC重装系统的可能性是比较大，说不定哪天就拿来玩下Arch Linux桌面系统。所以我有了备份整个系统的需求。

最开始想到的是大学时代非常流行的Ghost工具，可以全盘备份，那时还是XP时代。能不能备份Linux系统我没试过，我希望的是一个通用的备份方案，不受限于安装的什么系统。

想了下这个过程，插上一个U盘，启动引导到U盘上系统，然后可以访问NAS的共享文件夹，把硬盘压缩备份到NAS的共享文件夹下。

这个用Arch Linux的安装U盘，进入shell然后mount到NAS的共享文件夹，再通过dd拷贝不就可以实现了吗。果然搜索到了Arch Linux的Wiki，已经有比较详细的教程了，所以我采取了这种方案。

* `Mount NAS Backup`

```sh
mkdir /mnt/backup
mount -t cifs //192.168.1.100/backup /mnt/backup -o username=foo
ls /mnt/backup
```

* `Install pigz`

为了利用多核，需要安装pigz(Parallel implementation of the gzip file compressor)

```sh
pacman -Sy pigz
```

* `Create Disk Image`

```sh
fdisk -l
dd if=/dev/sda bs=256K status=progress | pigz -c > /mnt/backup/disk.img.gz
```

* `Restore System`

```sh
fdisk -l
unpigz -c /mnt/backup/disk.img.gz | dd of=/dev/sda bs=256K status=progress
```

* `Stats`

35.3 GB used, 256 GB copied, 13.38 GB saved

使用dd是不会关心硬盘到底使用了多少空间的，它是整个硬盘拷贝，所以硬盘容量越大，花的时间越多，现在我倒庆幸这个老硬盘只有256GB，因为经过了压缩，所以最终的镜像会比你使用的空间小，比如我用了35.3GB，实际镜像只有13.38GB。

我尝试了直接使用gzip，不利用多核要慢很多。dd的bs选项也非常影响速度，默认的512非常慢，256K对这个环境来说个不错的选择。

备份

```
single core bs=64K  124 MB/s    34:27   12.71 GB
multi core  bs=256K 405 MB/s    10:35   13.38 GB
```

还原

```
single core bs=256  43.8 MB/s   1:37:25 12.71 GB
single core bs=256K 271 MB/s    15:46   12.71 GB
multi core  bs=256K 370 MB/s    11:32   13.38 GB
```
