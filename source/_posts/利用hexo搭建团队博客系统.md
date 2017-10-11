---
title: 利用hexo搭建团队博客系统
date: 2017-06-12 15:08:39
tags:
- python
- hexo
---

### hexo搭建个人博客的原理
- 将md渲染为html
- 将html以及资源文件上传到github，利用githubpages功能进行展示
- 解析的过程是将目录下source/_post文件中的东西，渲染为publick中的html文件，然后将该文件夹下的文件复制到.deploy_git文件下，然后利用git操作将该文件夹下的文件push到对应的git库
<!--more-->

### 使用hexo制作团队博客的困难
- 内网不能直接依赖于github
- 团队开发的权限管理问题
- 团队开发的作者识别问题
- 团队开发的文件冲突问题


### 问题的解决方案




