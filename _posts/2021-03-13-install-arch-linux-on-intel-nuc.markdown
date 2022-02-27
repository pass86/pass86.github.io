---
layout: post
title: Install Arch Linux on Intel NUC
---

Arch Linux是我喜欢的一个Linux发行版本，喜欢它的滚动发行方式和最新的软件版本。在此之前我用过CentOS、Fedora和Ubuntu，CentOS基本上是用不上最新版本的软件，经常得自己下载源码编译，如果想使用Linux桌面系统的的话，Fedora和Ubuntu还是不错的选择。

安装Arch Linux是全命令行操作，要想安装成功将经历一次历险，同时也能学到Linux的一些知识，能对系统进行非常精细化的定制，这一点让我很爽，系统的安装内容全是我真正想要的东西，每个启动进程我都能理解它负责的功能是什么。

之前我是在Thinkpad上安装了Arch Linux，有段时间我还常在上面写C++代码，折腾过Arch Linux的桌面，那机器配置已经过时，型号是T450，i5双核四线程，8GB内存，目前我对装Linux系统机器的定位是一个24小时开机的Server，如果拿笔记本来担当，它的键盘和屏幕实在是有点浪费，安放位置也比较尴尬，有电源又得有网线，要打开屏幕才能触摸到电源按钮，反正就不像一个稳稳的待在那里的一个服务器。

所以我有再组装一台PC的想法，最后了解到了Intel NUC，简直就是完美满足我的需求，功耗低，体积小，可自定义RAM和SSD。刚好NUC 11最近发布，我非常想买的是NUC11PAHi5，但是等了两个周，能买到i5只有NUC11PAKi5，i7有考虑过，但是价格多1400块，看了下YouTube上性能评测，实在觉得不值。最终我用直接一步到位1T SSD说服了自己不用再考虑扩展存储了。发现内存涨价好多啊，买得我肉疼。

配置是这样的: NUC11PAKi5(i5-1135G7, 4核8线程, 8MB缓存, 2.4GHz-4.2GHz)，Kingston RAM(16GB, DDR4, 3200MHz) x 2，西部数据SSD(1TB, NVMe)。

![NUC11PAKi5](/assets/nuc-11-pak-i5.jpeg)

Arch Linux虽然我之前安装过几次，但是这次安装并不顺利，因为之前分区都是用的BIOS with MBR的方式，在这个NUC上执行时grub-install时会报错，`error "cannot find EFI directory"`，于是我又得重来尝试UEFI with GPT的分区，期间又遇到了之前没有遇到过的问题，所以在此记录下详细的安装过程备忘。

* `制作安装U盘`

官网下载ISO后，推荐使用Rufus这个工具写入U盘，之前用过macOS的dd，很慢。

* `BIOS禁用Secure Boot`

启动时按F2可进入BIOS。

* `从U盘启动`

插入安装U盘，启动时按F10选择该U盘为启动设备。

* `同步网络时间`

确保连接网线并接入互联网。

```sh
ping archlinux.org
timedatectl set-ntp true
```

* `查看硬盘情况`

```sh
fdisk -l
```

SATA硬盘会显示/dev/sda，NVMe硬盘会显示/dev/nvme0n1。

* `创建EFI分区`

```sh
fdisk /dev/nvme0n1
g # create a new empty GPT partition table
n # add a new partition
<enter> # Partition number
<enter>  # First sector
+512M  # Last sector
t # change a partition type
1 # EFI System
p # print the partition table
w # write table to disk and exit
```

* `创建根分区`

```sh
fdisk /dev/nvme0n1
n # add a new partition
<enter> # Partition number
<enter> # First sector
<enter> # Last sector
p # print the partition table
w # write table to disk and exit
```

* `创建文件系统`

我之前有尝试过Btrfs，但是最后那套系统启动不了，跟Btrfs有关，所以这次我选择了默认的ext4。

```sh
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
```

* `挂载文件系统`

```sh
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
```

* `调整镜像列表`

把中国的镜像地址剪切到文件头部，提高下载速度。

```sh
vim /etc/pacman.d/mirrorlist
```

* `安装基础包`

```sh
pacstrap /mnt base linux linux-firmware base-devel
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

* `安装Vim`

后面需用Vim进行配置文件的编辑。

```sh
pacman -S vim
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
127.0.1.1 HOME-SERVER.localdomain HOME-SERVER
```

* `安装启动器`

```sh
pacman -S grub efibootmgr intel-ucode
grub-install /dev/nvme0n1p1
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

* `添加管理员`

```sh
vim /etc/sudoers
# 取消注释%wheel ALL=(ALL) ALL
useradd -m -G wheel user00
passwd user00
```

* `设置网络`

```sh
sudo vim /etc/systemd/network/20-wired.network
```

```
[Match]
Name=en*

[Network]
DHCP=yes
```

```sh
sudo systemctl enable systemd-networkd
sudo systemctl start systemd-networkd
```

```sh
vim /etc/resolv.conf
# 增加一行nameserver 192.168.2.1
```

* `启用SSH登录`

```sh
sudo pacman -S openssh
sudo vim /etc/ssh/sshd_config
```

```
Port 56666
StrictModes no
RSAAuthentication yes
PubkeyAuthentication yes
```

```sh
sudo systemctl enable sshd
sudo systemctl start sshd
```

```sh
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
# 增加远端系统里ssh-keygen生成的id_rsa.pub里的内容
```

* `更新系统`

```sh
sudo pacman -Syu
```

* `关机`

```sh
sudo poweroff
```
