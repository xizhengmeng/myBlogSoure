---
title: xcode多版本自动化打包设置
date: 2017-10-10 12:29:12
tags:
- 工具
- xcode
---

如果你升级了xcode但是这个时候你又想降级回去，你发现appstore已经不要指望了，我们需要下载一个你需要版本的xcode。
下载各个Xcode版本的地址`https://developer.apple.com/download/more`
然后放置到我们的程序目录下，改个名字比如说叫Xcode2
但是现在问题又来了，加入你是个打包服务器，你使用的是xcodebuild来打包的，这个时候两个版本的xcode同时存在，它如何选择sdk进行打包呢？
- 首先查看sdk版本 	`xcodebuild -showsdks`
- 然后切换默认的xcode `sudo xcode-select -switch /Applications/Xcode2.app`
- 然后在执行`xcodebuild -showsdks`

你会发现sdk已经变了，这个时候你使用xcodebuild来打包可以选择你需要的这个版本的sdk了
