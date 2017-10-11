---
title: Python基础知识-字典
date: 2016-04-10 10:55:29
tags:
- Python
categories: Python
---
字典这种数据结构，我们又称之为映射(mapping)，在java中也是这么称呼的，但是在python中我们称之为字典，与OC中的叫法一致。
<!--more-->
## 创建字典
```
>>> dicts = {'name':'han','tele':'136'}//直接创建

>>> xiaohan = ('xiaohan',42)//利用dict函数来创建
>>> xiaoming = ('xiaoming',45)
>>> items = [xiaohan,xiaoming]
>>> d = dict(items)
>>> d
{'xiaohan': 42, 'xiaoming': 45}
>>> 

>>> d = dict(name='han',age=43)//或者dict函数还可以这样来用
>>> d
{'age': 43, 'name': 'han'}
>>> 
```
## 基本操作
- len(d)返回d中键值对的数量
- d[k]返回关联到k上的值
- d[k]=v将值v关联到键为k的项上
- del d[k]删除键为k的项
- k in d检查d中是否含有键为k的项
- 键类型可以是任意类型，比如元组，浮点数或者字符串
- 自动添加，及时键起出在字典中并不存在，也可以赋值，这样字典就会建立起新的项
- 成员资格:用k in d查找的是键而不是值，而在列表中，v in l是用值在查找，而不是索引

## 字典方法
### clear
```

```
### 

