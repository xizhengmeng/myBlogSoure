---
title: Python多进程通信和调用
date: 2017-06-12 15:00:31
tags:
- Python
---

## python多进程通信
>这里解决的主要是，如果是启动一个命令行，然后执行一个python脚本，如果有多个python脚本就需要多个命令行程序，这不是很明智的选择，最好的做法是，一个命令行搞定所有的python脚本，这里就涉及到两个问题
>
>- 如何做到并发
>- 如何在一个文件中添加一个python的执行入口
<!--more-->
```
#coding:utf-8

import os,re,time
import multiprocessing,commands,shutil,subprocess
from pymongo import MongoClient

lock = multiprocessing.Lock()

def func(name, processName, pipe):
    p_info = 'Process[%s]  hello %s' % (processName, name)
    print 'pipe send: ', p_info
    pipe.send(p_info)
    
    print 'sub pid: %d, ppid: %d' % (os.getpid(), os.getppid())
    time.sleep(0.1)

def serverForPull(name,pipe):

    path = '/Users/jdjr/Desktop/branches.txt'

    while True:
        time.sleep(3)
        newpath = '/Users/jdjr/Desktop/gradleDir'
        if os.path.exists(newpath) == True:
            os.chdir('/Users/jdjr/Documents/Build/JDJRAPPAndroid')

            currentbranch = ''

            branch = os.popen('git branch').read()

            lines = branch.split('\n')
            for item in lines:
                if '*' in item:
                   currentbranch = item[2:]

            p = subprocess.Popen("gradle assembleDebug",shell=True,stdout=subprocess.PIPE)

            indexcount = 0

            returncode = p.poll()

            while returncode is None:
                  text = p.stdout.readline().strip()
                  f = open('/Users/jdjr/Desktop/buildlog.txt','a')
                  f.write(text)
                  f.close()
        
                  mc = MongoClient("localhost",27017)
                  db = mc.package
                  post_info = db.index

                  dbs = post_info.find({'branchname':currentbranch})
                  count = 0
                  itemc = {}
                  for itemn in dbs:
                      itemc = itemn
                      count = count + 1

                  if count == 0:
                     post_info.save({'branchname':currentbranch,'count':indexcount,'time':0,'lastcount':0})
                  else:
                     ctime = itemc['time']
                     lastcount = itemc['lastcount']
                     
                     if 'Total time:' in text:
                         indexnew = text.find(':')
                         newtime = text[indexnew + 1:]

                         post_info.update({'branchname':currentbranch},{'branchname':currentbranch,'count':indexcount,'time':newtime,'lastcount':lastcount})
                     else:
                         post_info.update({'branchname':currentbranch},{'branchname':currentbranch,'count':indexcount,'time':ctime,'lastcount':lastcount})


                     if 'BUILD SUCCESSFUL' in text:
                         post_info.update({'branchname':currentbranch},{'branchname':currentbranch,'count':0,'time':ctime,'lastcount':indexcount})
                     else:
                         post_info.update({'branchname':currentbranch},{'branchname':currentbranch,'count':indexcount,'time':ctime,'lastcount':lastcount})

                  indexcount = indexcount + 1
        
                  returncode = p.poll()
        
            f1 = open('/Users/jdjr/Desktop/buildlog.txt','r')
            texts = f1.read()
            f1.close()
            if 'BUILD FAILED' in texts:
                (status, output) = commands.getstatusoutput('gradle assembleDebug')
                f2 = open('/Users/jdjr/Desktop/buildlog.txt','w')
                texts = f2.write(output)
                f2.close()

            shutil.rmtree(newpath)

            time.sleep(2)

            mc = MongoClient("localhost",27017)
            db = mc.package
            post_info = db.index

            post_info.update({'branchname':currentbranch},{'branchname':currentbranch,'count':20,'time':0,'lastcount':indexcount})

        if os.path.exists(path):
            f1 = open(path,'r')
            lists = f1.readlines()

            if len(lists) == 2:
               dir = lists[0]
               branch = lists[1]
               
               if 'JDJRAPPAndroid' in dir:
                  dir = '/Users/jdjr/Documents/Build/JDJRAPPAndroid'

               if 'JDMobileNew' in dir:
                  dir = '/Users/wxg/Documents/JDMobileNew'

               if len(dir) > 0:
                   print dir
              
                   os.chdir(dir)         
                   (status, output) = commands.getstatusoutput('git pull origin %s' % branch)
                   f1 = open('/Users/jdjr/Desktop/buildlog.txt','r')
                   text = f1.read()
                   f1.close()

                   f1 = open('/Users/jdjr/Desktop/buildlog.txt','w')
                   text1 = text + '\n' + output + '%i' % status
                   f1.write(text1)
                   f1.close()

                   f2 = open(path,'w')
                   f2.write('')
                   f2.close()

            f1.close()

def monitorForBlog(name,pipe):
    lastTimeStr = ''
    while True:
      time.sleep(3)
      os.chdir('/Users/jdjr/Documents/Blog/blogsource')
      
#      os.chdir('/usr/local/var/www')

      (status, output) = commands.getstatusoutput('git pull origin blog')

      (status, output) = commands.getstatusoutput('git log')

      f = open('/Users/jdjr/Documents/Blog/logcommit.txt','w')
      f.write(output)
      f.close()

      f1 = open('/Users/jdjr/Documents/Blog/logcommit.txt','r')
      lines = f1.readlines()
      firstline = lines[0]
      f1.close()

      if lastTimeStr != firstline:
          print '推送一次博客'
          sourcepath = '/Users/jdjr/Documents/Blog/blog/source'
          
          if os.path.exists(sourcepath):
             shutil.rmtree(sourcepath)
          
          shutil.copytree('/Users/jdjr/Documents/Blog/blogsource/source','/Users/jdjr/Documents/Blog/blog/source')

          os.chdir('/Users/jdjr/Documents/Blog/blog')

#          os.popen('hexo clean')
          os.system('hexo clean')
          os.system('hexo g')
          
          filepath = '/usr/local/var/www'
          if os.path.exists(filepath):
             shutil.rmtree(filepath)

          publicfile = '/Users/jdjr/Documents/Blog/blog/public'
          while not os.path.exists(publicfile):
                time.sleep(1)
#                print 'not exist'

          shutil.move(publicfile,filepath)
          
          lastTimeStr = firstline
          print lastTimeStr


def main():
    print 'main pid: %d, ppid: %d' % (os.getpid(), os.getppid())
    
    # 注意： 此处是Pipe来自multiprocessing.Pipe(), 其来源于 multiprocessing.connection import Pipe
    pipe_parent, pipe_child = multiprocessing.Pipe(duplex=False)
    
    processList = []
#    for i in xrange(4):
#    pro = multiprocessing.Process(target=func, args=('ceshi', 'Process-' + str(10), pipe_child))
#    pro.start()
#    processList.append(pro)
#    
#    pro1 = multiprocessing.Process(target=func1, args=('waha',pipe_child))
#    pro1.start()
#    processList.append(pro1)
#    
#    pro2 = multiprocessing.Process(target=func2, args=('heihei',pipe_child))
#    pro2.start()
#    processList.append(pro2)

    pro3 = multiprocessing.Process(target=serverForPull, args=('heihei',pipe_child))
    pro3.start()
    processList.append(pro3)

    pro4 = multiprocessing.Process(target=monitorForBlog, args=('heihei',pipe_child))
    pro4.start()
    processList.append(pro4)
    
    for pro in processList:
        pro.join()      # 在此处阻塞子进程，可实现异步执行效果，直至子进程全部完成后再继续执行父进程
    
    pipe_child.send(None)   # 此处等全部子进程执行完成后，输入'None'标记，表示Pipe结束接收任务，可以退出

    while pipe_parent:
          p_info = pipe_parent.recv()
          print 'pipe get: ', p_info
        
          if not p_info:      # 如果接收到了'None'标记，退出Pipe
             print 'pipe get:  None, then exit out.'
             break

# 测试
if __name__ == '__main__':
    main()
    print('end.')


```
