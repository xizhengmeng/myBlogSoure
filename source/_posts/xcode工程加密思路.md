---
title: xcode工程加密思路
date: 2017-10-10 19:45:46
tags:
- xcode
- git
---

现在有这么一个需求，如何做到如果不安装我们的开发环境，就无法查看iOS工程。
首先我们如果对所有的代码进行加密这肯定是一个浩大的工程，所以我们选择是对project加密，

```
/usr/bin/env python
#coding:utf-8

from Crypto.Cipher import AES  
from binascii import b2a_hex,a2b_hex  
  
class prpcrypt():   #定义一个类  
    def __init__(self,key):  
        self.key=key  
        self.mode=AES.MODE_CBC  
    def encrypt(self,text):  
        cryptor=AES.new(self.key,self.mode,self.key)  
        x = len(text) % 16  
        if x != 0:  
            text = text + '0' * (16 - x)     #不满16，32，64位补0  
        self.ciphertext=cryptor.encrypt(text)  
        return b2a_hex(self.ciphertext)  
    def decrypt(self,text):  
        cryptor=AES.new(self.key,self.mode,self.key)  
        plain_text=cryptor.decrypt(a2b_hex(text))  
        return plain_text.rstrip('\0')  
  
pc=prpcrypt('tecentbluewhaleA') #自己设定的密钥

f = open('project.pbxproj','r')
text = f.read()
liens = text.split('\n')

if len(liens) > 10:
    item = liens[10]
    print item
    # item = pc.decrypt(item)
    item = pc.encrypt(item)
    item = item.replace('000000000000000','')
    print item
    liens[10] = item

for i in range(0,len(liens) - 1):
    liens[i] = liens[i] + '\n'

strings = ''.join(liens)

f = open('project.pbxproj','w')
f.writelines(strings)
f.close()
```
这里我们使用到了pycrypto库，安装的时候需要先卸载
`sudo pip uninstall crypto`
`sudo pip uninstall pycrypto`

然后
`sudo pip install pycrypto`这样就安装好了需要的库

关于我们加密那哪些东西，其实主要是project文件里边的解析文件，将一些关键是被符号进行加密，其实如果查看git操作记录是可以发现该如何还原的，这个时候就需要靠多行替换来增大替换的工作量来人为设置门槛

另外需要配合改装的git，在commit的时候进行加密然后commit，在pull的时候也是解密然后commit一下，这样我们就做到了，在远端库里边是不能被直接有序查看的代码，在本地需要配合我们的git才能被顺畅的使用
