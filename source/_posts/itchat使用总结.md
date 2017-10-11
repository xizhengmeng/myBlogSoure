---
title: itchat使用总结
date: 2017-06-14 09:23:30
tags:
- python
---

### 登录
```
itchat.auto_login()
```

### 查找好友

>获取任何一项等于name键值的用户
itchat.search_friends(name='littlecodersh')
这里的name是RemarkName', 'NickName', 'Alias


### 发送中文

要进行一个转换
```
f = open(path,'r')
text = f.read()
text = text.decode('utf-8')
```
