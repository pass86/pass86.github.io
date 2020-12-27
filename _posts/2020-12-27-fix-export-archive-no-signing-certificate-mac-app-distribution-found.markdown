---
layout: post
title: 'Fix exportArchive: No signing certificate "Mac App Distribution" found'
---

这个错误提示出现在Unity项目整合平台SDK后使用xcodebuild -exportArchive时。明明是导出的iOS平台工程，为什么会出现Mac App的关键字呢?而iOS的发布证书是有正确配置的，之前是可以正常工作的。这让我非常疑惑，甚至周末都来解决这个问题，但是经过多次的Google查找是否有类似问题，但是没有遇到iOS工程输出这种错误的。所以周末我并没有解决问题。

周一和平台相关的同事沟通后，发现有此类案例，原因是Copy Bundle Reources里有拷贝Static Library。我在Unity导出的Xcode工程里手动删除了相关的项，再次执行之前的导出脚本，成功了!

那为什么会有这些Static Library在Xcode工程的Copy Bundle Reources里呢？链接这些库使用的是XUPorter插件，经过Review后发现代码里有两份XUPorter，一份是之前加入的，另一份是新加入SDK里的，被加入了额外的namespace所以没有编译错误。

而之前的XUPorter会在OnPostProcessBuild里查找Assets目录下所有.projmods文件加入XCProject使用。问题就出在这里，新加入的SDK里也有很多.projmods文件，所以被误加入了XCProject，导致生成出来的Xcode工程不符合预期，然后导出的行为也就不符合预期。

解决方法就是把之前的XUPorter搜索.projmods的目录限定到更明确的Mods目录。跑整个构建流程，没有问题了。

我觉得这个问题非常的罕见，而且错误提示也非常离谱，和问题的源头离得很远，也浪费了我的周末时间，所以我希望记录下来帮助到再次遇到此类问题的程序员，通过Google搜索快速解决此类问题。
