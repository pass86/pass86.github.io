---
layout: post
title: Home Network Set Up
---

重新装修房子时，弱电箱拉了网线到卧室、电视墙。虽然大部分时间都是在用WiFi，但是高速传输还是得靠有线网络，所以留下扩展的可能性。

不过这次没想周全，每个接入点只拉了一根网线，实际使用时发现每个接入点有两根网线是比较灵活的，可以一根来一根回。

比如说你想把路由器放在电视柜的话，就得一根来的接入互联网，一根回的到弱电箱的交换机分发到各个房间。所以目前我的路由器只能在弱电箱附近。

如果是全屋地毯的话，把网线埋在地毯下面应该也是个很灵活的解决方法。

以下就是我目前在家里的网络拓扑图。

![Home Network](/assets/home-network.png)

许多路由器的默认IP都是192.168.1.1，在多路由时会冲突，这里需要设置下。电信网关使用192.168.1.1，Linksys WRT1900ACS使用192.168.2.1，Linksys WRT32X使用192.168.3.1。

我个人使用的网络在Linksys WRT32X后面有两个好处，一是我重启路由不会影响其他人，二是我是可以连接到Linksys WRT32X的管理页面的，不用切换网络。


接下来说说路由器，之前用过塑料壳、铁壳的TP-Link，使用体验没啥差别。不知什么时候了解到OpenWrt，能自己安装路由器的系统，而且还是开源的，这点就让我感到兴奋。比较各个兼容硬件后，发现Linksys的WRT系列兼容性是比较好的。于是我就之前就买了一台Linksys WRT1900ACS，刷了OpenWrt。

![Linksys WRT1900ACS](/assets/linksys-wrt1900acs.jpg)

折腾OpenWrt系统时，常重启路由器，导致网络不稳定，被女朋友吐槽网络好差。这次为了让我折腾OpenWrt时，不影响到其他人，我另外购置一台路由器作为家里的主路由，Linksys WRT32X，使用原装的系统。

![Linksys WRT32X](/assets/linksys-wrt32x.png)

再来聊聊NAS，最开始我不知道有NAS这个东西，只是有低功耗下载的需求，当时去华强北的赛格买了个准系统，不带风扇，装的Windows，硬盘忘了有多大，反正没有500G，远程桌面很卡，倒是能满足我当时的需求。

自从进了PT圈后，小硬盘满足不了小水管了。有试过e-SATA的外接硬盘盒，感觉这个方案很临时。然后买了个4盘位的U-NAS，原装的系统用过一段时间后，爱折腾的我把它先后装了CentOS，Arch Linux，自定义Samba，Transmission等等，软件折腾后开始折腾硬件，拆机换了系统盘后，开始不稳定，时不时就死机了。期间两次丢了所以NAS硬盘上的数据，删库跑路这种事情在家里常干...

经历了这么多折腾后发现也没啥好结果，于是我把目光转向了群辉，体验过他的DSM后，瞬间不想折腾了，于是这次买了群辉的DS718+，之前U-NAS的4盘位其实最多也就插了3个硬盘，4盘位还多个风扇，耗电和噪音都更多，所以这次选择2盘位。

![Synology DS718+](/assets/synology-ds718+.jpg)

买之前很在意视频在NAS上硬解码，这样就可以在iPad上看动辄10G的1080p电影了，实际情况是，DS video因为版权问题不支持很多格式的内置字幕。

NAS目前对我来主要有两个用处，一是挂PT存放高清电影🎬，二就是Time Machine备份我的MBP。

其实我倒不是怕MBP正常使用的硬盘质量问题，我是怕MBP常背出去，万一哪天摔坏了、弄丢了、被偷了，没有Plan B的话，心里有点不安。总有些东西没有在iCloud或者GitHub上。比如各种环境搭建，软件安装，也是比较折腾的过程。

使用Time Machine有点心得，如果用WiFi备份的话，速度很慢，经常一晚上都无法完成一次备份，所以在家都把MBP关闭WiFi，连接到Belkin Thunderbolt 3 Dock上，用有线网络，千兆网很快完成一次备份。

![Belkin Thunderbolt 3 Dock](/assets/belkin-thunderbolt3-dock.jpeg)

如果你是个高清电影爱好者，喜欢自己下1080p或者4K的电影看，有大电视的话，Apple TV 4K绝对是最佳伴侣。

![Apple TV 4K](assets/apple-tv-4k.jpeg)

在Apple TV 4K上使用Infuse播放NAS里的电影，整个体验非常舒服顺畅，Infuse能很好的呈现你的收藏，封面，简介，字幕。

至于电视我选择了Sony A9G OLED，画面效果没有让我失望，看4K电影，有身临其境的感觉。

![Sony A9G](/assets/sony-a9g.webp)

使用OpenWrt的好处之一就是科学上网我可以在路由器上做，我的方案使用了：SSH tunnel、Squid、Proxy auto-config、Auto Proxy Discovery。

Nintendo Switch直接设置网络代理到Squid（192.168.2.1:8080），浏览商店时顺畅多了，看介绍视频也能顺畅看完了😺。

![Nintendo Switch](/assets/nintendo-switch.jpg)

同样也可以设置PS4。可惜Apple TV 4K没有找到这样的设置。

这些事情其实在去年年底就做完了，使用了快一年，目前处于稳定状态。
