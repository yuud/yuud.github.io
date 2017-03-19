---
layout: post
title: "fastboot and adb 工具"
tagline: ""
description: ""
category: 经验总结
tags: [Android, Fastboot, adb,]
last_updated: 
---

早之前曾经写过过一篇[文章](/post/2016/09/useful-adb-command.html)总结 adb 的常用命令，这次刷机也复习了一些命令，这里再补充一下刷机中必不可少的 fastboot 命令。

就像之前adb中说过的那样， adb 用来向开机的 Android 设备发送指令，而 fastboot 则是向开机的 Android bootloader 来发送指令，因此在刷机过程中，如果手机重启进入 bootloader 基本之后使用的 命令都离不开 fastboot。比如在安装第三方 recovery 时，要用到 `fastboot flash recovery recovery.img` 这样的指令。虽然不能依靠此命令来刷入ROM，但是 fastboot 也能够做一些 adb 命令无法实现的功能。

## bootloader
关于 bootloader 部分可以参考 [之前的文章](/post/2013/09/prevent-flash-android-rom-brick.html)

## fastboot

adb 和 fastboot 两个工具都是跟随着 Android SDK 分发的，但是由于这两个工具需求过高，Google 单独提供了两个工具集的[下载](https://developer.android.com/studio/releases/platform-tools.html#download)。

在下载之后，需要将文件路径添加到系统环境变量中，以便于在各个路径下访问。

假设加压后的路径为 `~/Android/Sdk/platform-tools/`， 那需要在 ～/.bashrc 或者 ~/.zshrc 中配置。

	export PATH="$PATH:/home/einverne/Android/Sdk/platform-tools"

WIndows 同理，在系统配置环境变量。

然后在终端下使用 `adb devices` ，就能够查看连接的设备。同样在 bootloader 下，也可以使用 `fastboot devices` 来查看连接设备，如果未出现已经USB连接的设备，查看驱动是否安装完好即可。

