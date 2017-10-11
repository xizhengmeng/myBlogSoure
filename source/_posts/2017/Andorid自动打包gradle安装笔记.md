---
title: Andorid自动打包gradle安装笔记
date: 2016-12-16 08:41:48
tags:
- 自动化
---

- 安装andorid studio
- copy一份sdk过来
- 设置andorid studio的sdk路径
<!--more-->
- 安装sdkman
  - 在控制台或者item2中输入
    ```
    curl -s https://get.sdkman.io | bash
    source "$HOME/.sdkman/bin/sdkman-init.sh"
    ```

    检验sdkman是否安装成功
    ```
    sdk version
    ```
- 安装andorid studio中认可的gradle版本
```
sdk install gradle 2.8
```
- 升级jdk到andorid studio认可的版本比如说1.7
- 到工程根目录，执行gradle build就可以了'gradle assembleDebug'
- 打包出来的文件在outputs中

