---
title: python发送邮件代码记录
date: 2017-10-10 20:38:05
tags:
- python
---

```
#!/usr/bin/env python
#coding:utf-8

from email import encoders
from email.header import Header
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.mime.application import MIMEApplication
from email.utils import parseaddr, formataddr
import smtplib,os

def _format_addr(s):
    name, addr = parseaddr(s)
    return formataddr(( \
        Header(name, 'utf-8').encode(), \
        addr.encode('utf-8') if isinstance(addr, unicode) else addr))

def sendMail():
    username = 'hanshenghui'
    password = '11111111111'

    from_addr = 'hanshenghui@jd.com'
    to_addr = 'hanshenghui@jd.com'
    # to_addrList = ['hanshenghui@jd.com','gaixutian@jd.com','wangzhenyang1@jd.com']
    to_addrList = ['hanshenghui@jd.com','wangxiugang@jd.com','dingchao1@jd.com']
    smtp_server = 'smtp.jd.local'

    msg = MIMEMultipart()
    part = MIMEText("这是一封测试邮件\n\n现在主要有以下几个待确定点:\n1.我们可以通过对数据进行提取，做一些分析和简介作为邮件的正文，或者是一些其他什么内容\n2.现在邮箱我添加了名字为JDJRMobile，实际上是我自己的邮箱，用哪个邮箱来发送比较合适\n3.检测哪个分支\n4.检测频率\n\n")
    msg.attach(part)
    
    basePath = os.environ['HOME']
    reportPath = basePath + '/Documents/Build/JDJRAPPAndroid/infer-out/report.csv'
    if os.path.exists(reportPath):    
       part1 = MIMEApplication(open(reportPath,'rb').read())
       part1.add_header('Content-Disposition', 'attachment', filename="report.csv")
       msg.attach(part1)

    # part2 = MIMEApplication(open('cesh.png','rb').read())
    # part2.add_header('Content-Disposition', 'attachment', filename="cesh.png")
    # msg.attach(part2)

    msg['From'] = _format_addr(u'JDJRMobile <%s>' % from_addr)
    msg['To'] = _format_addr('JDJR')
    msg['Subject'] = Header(u'JDJR安卓源码检测报告', 'utf-8').encode()

    server = smtplib.SMTP(smtp_server,25)
    server.set_debuglevel(1)
    server.sendmail(from_addr, to_addrList, msg.as_string())
    server.quit()
```