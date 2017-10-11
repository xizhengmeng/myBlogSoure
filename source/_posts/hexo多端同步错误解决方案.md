---
title: hexo多端同步错误解决方案
date: 2017-10-11 22:00:15
tags:
- hexo
---
`hexo d`这个命令是相当于直接`git push`，如果只有一台机器那么是没有问题，但是如果是两台机器，那么还少一步`git pull`，所以如果是两台机器同时维护一个库的时候需要

- 进入到`.deploy_git`这个目录下
- 然后执行`git pull origin master --allow-unrelated-histories`
- 然后再push
- 执行`hexo d`就好了



