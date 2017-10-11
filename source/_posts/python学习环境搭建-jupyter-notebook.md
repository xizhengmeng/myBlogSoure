---
title: python学习环境搭建-jupyter notebook
date: 2017-10-11 11:35:30
tags:
- python
---

### 先说一下好处：
我们学习python可以在命令行输入python直接进入python环境进行练习，但是问题是多行运行比较繁琐，这里我们推荐使用jupyter来运行代码

### 再说一下环境 ：

MAC台式机

python:mac下自带Python 2.7.10

- 1.先升级了pip安装工具：sudo python -m pip install --upgrade --force pip

- 2.安装setuptools 工具：sudo pip install setuptools==33.1.1

- 3.安装 Python-dateutil:sudo pip install python-dateutil==2.2

- 4.安装six:sudo pip install --ignore-installed six

- 5.安装jupyter：sudo install jupyter

启动命令：直接在终端输入jupyter notebook

折腾了半天，才搞好，以上命令是在安装的时候遇到各种问题时使用的，下面写记录一下遇到的问题 以及对应的解决办法

1.ImportError: cannot import name _thread  报这个错误 解决办法：

sudo pip uninstall python-dateutil
sudo pip install python-dateutil==2.2
2.File "/Library/Python/2.7/site-packages/dateutil/tz/_common.py", line 2, in <module>

from six.moves import _thread
ImportError: cannot import name _thread

解决办法：安装six 命令再上方

解决了上面的问题 启动的时候 还是会报错这是个顽固的错误：

  File "/Library/Python/2.7/site-packages/packaging/requirements.py", line 59, in <module>

    MARKER_EXPR = originalTextFor(MARKER_EXPR())("marker")

TypeError: __call__() takes exactly 2 arguments (1 given)

解决办法：根据错误信息直接找到路径中的文件，打开文件将 59 行中的函数修改

59：#MARKER_EXPR = originalTextFor(MARKER_EXPR())("marker")
60：MARKER_EXPR = originalTextFor(MARKER_EXPR("marker"))

修改好之后直接保存文件 再次运行jupyter notebook 命令 即可启动jupyter 

关于six那个问题，还有一点 需要删除我们默认路径下的six 否则 即使更新成功了 也不会使用最新的six

方法：先查看一下默认的six路径

 import six


### 最后说一下怎么使用

file-->new-->python2