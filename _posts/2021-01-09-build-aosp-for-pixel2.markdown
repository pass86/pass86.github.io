---
layout: post
title: Build AOSP for Pixel 2
---

我是从2019年底才开始接触AOSP(Android Open Source Project)的，因为当时在研究Android内存这块，了解到Android 10推出的Perfetto框架，能从系统底层记录堆内存的分配情况，听起来太酷了。但是实际使用过程中发现它的一个严重bug，比如记录王者荣耀的堆内存分配，heapprofd会在王者荣耀启动后没多久就退出，我改Perfetto的代码修了这个bug，quick and dirty，于是我就有构建AOSP的需求。同时也和Google工程师沟通，他在master修复了这个bug，当然这个过程是漫长的。

从Android 9开始AOSP默认只支持Pixel系列，所以买了一台测试手机Pixel 2，花了不到1000块，性价比还是挺高的。

当时我是在macOS下构建的AOSP，因为AOSP需要文件系统区分大小写，所以在macOS的下要么买一个新的SSD格式化为区分大小写的，要么创建一个区分大小写的disk image。用iMac(i7-6700K, 16GB RAM, 512GB SSD)构建花了3个多小时，用MBP(i7-7920HQ, 16GB RAM, 250GB disk image)花了6个多小时。

最近我又准备构建AOSP，在macOS下却遇到了一些莫名其妙的错误，Google许久没解决掉，加上我新组装了一台PC，想利用它的高配置，构建更快点，我尝试了在Windows 10上利用WSL构建AOSP，也是遇到各种错误。我反思了一下，为什么不用官方推荐的Ubuntu呢，浪费时间在这些莫名其妙的错误上，不太值。于是专门购买一块SSD来安装Ubuntu，官方推荐的是18.04，我安装了20.04，还好也可以构建成功。在PC(i7-10700K, 32GB RAM, 1TB SSD)上花了1小时10分钟，Nice。

下面列出在Ubuntu上构建AOSP各个步骤，最简化的流程。

* `安装依赖包`

官方列出

sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig

实践解决error while loading shared libraries: libncurses.so.5

sudo apt-get install libncurses5

* `下载代码管理工具repo`，这是一个python脚本，如果你的默认python不是python3的话，需要打开文件把第一行的python改成python3

curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

chmod a+x ~/bin/repo

vim .bashrc

export PATH=~/bin:$PATH

在中国大陆需要设置http_proxy和https_proxy，原因应该都懂的，AOSP推荐用bash，设置好环境变量后，需要重开一个bash

* `创建代码目录`，可用空间官方推荐需要250GB，下载后代码库70GB左右，构建完成后占用170GB左右，

mkdir ~/aosp

cd ~/aosp

* `设置代码URL和版本`，版本可以在这里找到[https://source.android.com/setup/start/build-numbers](https://source.android.com/setup/start/build-numbers)

repo init -u https://android.googlesource.com/platform/manifest -b android-10.0.0_r41

* `下载代码`，5M的出口带宽，峰值500KB/s左右，我下了一天

repo sync

* `下载设备驱动`，[https://developers.google.com/android/drivers](https://developers.google.com/android/drivers)

Pixel 2的代号是walleye，下载解压后会得到extract-google_devices-walleye.sh和extract-qcom-walleye.sh，移动到AOSP的代码目录，分别执行两个sh脚本

* `设置构建参数`，userdebug版本可以拥有root权限，这当然是我想要的

source build/envsetup.sh

lunch aosp_walleye-userdebug

* `启动构建`

m

* `刷机`，需要在Pixel 2里设置OEM unlocking开启

adb reboot bootloader

fastboot flashall -w

如果想清除构建的内容的话，可以执行make clobber。

虽然我是Apple的粉丝，但是能使用从源码构建出的系统，感觉是一件很酷的事情，拥有root权限也能更方便的去调试App，这台Pixel 2成为了我的Playground。
