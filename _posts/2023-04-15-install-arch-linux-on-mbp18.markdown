---
layout: post
title: Install Arch Linux on MBP18
---

我有一台闲置的MacBook Pro 2018，一直都不知道拿来做什么，放在抽屉里，有一天拿来用居然开不了机了，去苹果店检查是主板坏了，苹果的做法是只换不修，换主板需要5K多，修好再卖个二手都不一定能赚钱。后面在淘宝上找到一家华强北的店可以修苹果电脑，最近去把它修好了，只花了几百块，问了下这台MBP能卖多少钱，说只能卖3K左右，真不想这样贱卖了。

但是又不想这样闲置下去，于是想了几个可以用它来做的事情。我一直没有一台电脑是拿Arch Linux作为主力桌面系统的，之前装在NUC上的Arch Linux是作为服务器运行，追求稳定长时间运行，不打算装桌面系统。那我为什么需要一台Arch Linux桌面系统呢，目前想到两个应用的地方：一是用来编译Open Wrt，我一直想自己编译调试Open Wrt，但一直没有合适让我折腾的环境；二是用来编译AOSP，之前编译AOSP的环境是安装在移动硬盘上的一个Ubuntu，需要编译时插到我的PC上，感觉也是很临时。这个MBP装上Arch Linux可以作为我在Linux下的主力开发机。

2018年的MBP有T2芯片，Arch Linux官方的ISO没有支持好，我装了官方的ISO测试，可以安装到MBP的SSD上去，系统可以启动，但是笔记本键盘，无线网卡，USB有线网卡都不能用，只能用USB键盘。这样的系统装上去也没啥用处了。在MBP上装Linux这个事情，看来也是有不少人有这需求，于是有了[Linux for T2](https://github.com/t2linux)，从这里[archiso-t2](https://github.com/t2linux/archiso-t2/releases)下Arch Linux的ISO，再按照[Installation wiki](https://wiki.t2linux.org/distributions/arch/installation/)，结合我之前安装Arch Linux经验，我已经能在MBP18上运行Arch Linux，笔记本键盘，无线网卡，USB有线网卡，USB键盘都测试可用，不过还没有装桌面系统，这个后面再做。其中也是遇点坑，有必要把整个过程记录下来，下次装的时候也能快点，希望也能帮助到有同样需求的网友。

* `制作安装U盘`

这次我参考了[t2linux文档](https://wiki.t2linux.org/guides/preinstall/)使用dd的方式，之前用dd一直填的`of=/dev/diskX`，速度很慢，这次用了`of=/dev/rdiskX`，U盘是满速写入，比之前快了很多。

macOS中使用dd需要给终端增加权限，系统设置=>隐私与安全性=>完全磁盘访问权限=>+=>终端。

```sh
# 查看U盘是哪个disk
diskutil list
# 假如是/dev/disk2，先卸载它
diskutil unmountDisk /dev/disk2
# 写入archlinux-t2的ISO
sudo dd if=archlinux-t2-2023.03.23-t2-x86_64.iso of=/dev/rdisk2 bs=1m
```

* `关闭文件保险箱`

系统设置=>隐私与安全性=>文件保险箱=>关闭...

* `磁盘分区`

打开[磁盘工具](https://support.apple.com/guide/disk-utility/dskutl14027/mac)，左边选中APPLE SSD，右边点击分区=>点击加号=>添加分区=>名称和格式随意（后面会被抹掉），大小我给Macintosh HD留了50GB，其他全给Arch Linux=>应用。

* `提取Wi-Fi和蓝牙的固件`

相当于Wi-Fi和蓝牙的驱动是用苹果官方的，完整文档在[这里](https://wiki.t2linux.org/guides/wifi-bluetooth/)，下载提取脚本[firmware.sh](https://wiki.t2linux.org/tools/firmware.sh)。

```sh
# 做的事情就是把固件打包放到EFI分区
bash ~/Downloads/firmware.sh
# 可以检查下是否有firmware.sh和firmware.tar.gz
sudo diskutil mount disk0s1
ls /Volumes/EFI
```

* `从U盘启动`

这里有个坑，U盘要用USB Type-A到Type-C的转接头，直接插在MBP上，试了插在贝尔金的Dock和绿联的Hub上都会导致启动卡死，后面用了Google Pixel 2包装盒里带的转接头才顺利进入U盘系统。

安装时需要联网下载软件包，所以要连接有线网络，我用的是绿联那个带网线接口的Hub。

插入安装U盘，MBP启动时按住option，直到出现选择启动磁盘的画面，选择`EFI Boot`。

* `查看硬盘情况`

```sh
fdisk -l
```

* `创建文件系统`

```sh
mkfs.ext4 /dev/nvme0n1p3
```

* `挂载文件系统`

```sh
mount /dev/nvme0n1p3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
```

* `设置代理`

懂的都懂。

```sh
export HTTP_PROXY=http://192.168.1.1:8080
export HTTPS_PROXY=http://192.168.1.1:8080
```

* `安装基础包`

又一个坑，不要使用t2strap那种方式，我最开始使用这种方式，结果/mnt/etc/pacman.conf直接被覆盖了，arch-chroot后，连个vim都装不上。

这里也要安装上python，文档里并没有说，后面运行firmware.sh时会用到python。

```sh
pacstrap /mnt base linux-t2 apple-t2-audio-config apple-bcm-firmware linux-firmware iwd grub efibootmgr vim python
```

* `增加软件源`

```sh
vim /mnt/etc/pacman.conf
```

```
[Redecorating-t2]
Server = https://github.com/Redecorating/archlinux-t2-packages/releases/download/packages

[arch-mact2]
Server = https://mirror.funami.tech/arch-mact2/os/x86_64
SigLevel = Never
```

* `生成文件系统挂载信息`

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

* `切换到安装的系统`

相当于后面的操作都是针对于挂载到/mnt的系统。

```sh
arch-chroot /mnt
```

* `设置时区`

```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

* `设置文本编码格式`

```sh
vim /etc/locale.gen
# 取消注释en_US.UTF-8 UTF-8
locale-gen
vim /etc/locale.conf
# 增加一行LANG=en_US.UTF-8
```

* `设置主机名`

```sh
vim /etc/hosts
```

```
127.0.0.1 localhost
::1 localhost
127.0.1.1 mbp.localdomain mbp
```

* `生成initial RAM disk`

```sh
# 增加apple-bce到MODULES里
vim /etc/mkinitcpio.conf
mkinitcpio -P
```

* `安装启动器`

```sh
# GRUB_CMDLINE_LINUX里增加intel_iommu=on iommu=pt pcie_ports=compat
vim /etc/default/grub
# 苹果的EFI boot loader会去读取/EFI/BOOT/BOOTX64.EFI，这里就把GRUB写到那里
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --removable
grub-mkconfig -o /boot/grub/grub.cfg
```

* `设置root密码`

```sh
passwd
```

* `返回到U盘系统`

```sh
exit
```

* `重启`

```sh
reboot
```

* `设置有线网络`

```sh
vim /etc/systemd/network/20-wired.network
```

```
[Match]
Name=en*

[Network]
DHCP=yes
```

```sh
systemctl enable systemd-networkd
systemctl start systemd-networkd
```

```sh
# 增加一行nameserver 192.168.1.1
vim /etc/resolv.conf
```

* `使用Wi-Fi和蓝牙的固件`

```sh
umount /dev/nvme0n1p1
mkdir /tmp/apple-wifi-efi
mount /dev/nvme0n1p1 /tmp/apple-wifi-efi
bash /tmp/apple-wifi-efi/firmware.sh
# 检查日志
journalctl -k --grep=brcmfmac
```

* `设置无线网络`

```sh
pacman -S networkmanager iw wpa_supplicant
systemctl enable NetworkManager
systemctl start NetworkManager
# 选择Wi-Fi，输入密码
nmtui
```

* `查看电池状态`

```sh
pacman -S acpi
acpi
```

* `运行效果`

![MBP18-ArchLinux](/assets/mbp18-archlinux.jpeg)
