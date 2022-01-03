---
layout: post
title: How to copy file without cp command
---

前一阵子偶然翻出了我9年前的Android Pad，顺便拿来玩了一下，这个系统是Android 4.1.2的版本，我用adb shell可以连接进去，当时想在里面拷贝一个文件，执行了cp命令时才发现没有这个命令，这太让我意外了，该怎么办呢？

我用下面这条命令成功完成了cp的功能，觉得挺有意思的，所以记录一下。

```sh
cat foo.dat > bar.dat
```
