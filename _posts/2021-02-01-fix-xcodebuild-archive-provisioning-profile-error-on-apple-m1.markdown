---
layout: post
title: Fix xcodebuild archive Provisioning profile error on Apple M1
---

最近入手了Mac mini M1(16GB RAM, 256GB SSD)，买它的主要目的就是用来运行GitHub Actions的self-hosted runner。

然而在把之前的iOS项目在Mac mini上用命令行构建时遇到了这个错误: `error: Provisioning profile "iOS Team Provisioning Profile: *" doesn't include the currently selected device "Mac mini" (identifier ...`

之前我是用的x86 MBP来做self-hosted runner，没有出现过这个问题。我之前买过Apple Developer的Membership，但是后面没有续订已经过期了，因为发现直接用免费的Automatically manage signing也能满足我的需求。Membership过期后Apple Developer网站上证书相关操作页面都没有入口，为了把Mac mini的identifier加上去，我又续订了。

在Apple Developer的Devices页面增加了报错信息里的identifier后，我命令行构建iOS App可以成功。

然而在我配置好GitHub Actions后，通过GitHub Actions runner运行构建脚本时，却遇到同样的错误，仔细一看identifier和我配置的不一样。一阵Google后发现我之前我配置的是Provisioning UDID，现在报错信息里是Hardware UUID，这两个ID可以在System Information里看到，在我x86 MBP上这两个值是一样的，在Mac mini M1上这两个值是不一样的。

我把Hardware UUID加到Devices里，再更新Provisioning Profiles(`~/Library/MobileDevice/Provisioning Profiles`)，验证Hardware UUID是没有加入到`PROVISIONED DEVICES`的。

通过不断的Google，终于找到一个和我一样的案例，而且作者自己给出了解决方法，在xcodebuild参数里增加`-destination 'generic/platform=iOS'`就可以了。

所以其实不用买Membership也是可以解决问题，我知道在x86 Mac上是不用在Devices里加UDID，但鉴于M1是ARM架构可以运行iOS的App的，我以为这是个新的机制要求。这都是些黑盒的东西，xcodebuild的文档对`-destination`也没有更详细的说明，而且通过GitHub Actions运行报错的又是Hardware UUID。

推测在不加`-destination`参数时，xcodebuild默认会寻找一个当前支持的设备，iOS App在x86 Mac下是不支持的，所以没有设置当前设备，而在M1 Mac下则支持有了选择。

然而为什么Github Actions调用时取到的是Hardware UUID，这个就难以理解了。

总之，我目前能Make it work就达到我的目的了。
