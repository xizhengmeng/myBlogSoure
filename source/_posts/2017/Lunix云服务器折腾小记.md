---
title: Lunix云服务器折腾小记
date: 2017-01-17 17:33:07
tags:
- Lunix
---
### Mac登录云服务器
```
ssh root@127.987.883.888
```
这样再输入密码我们就进入了我们的服务器的命令行界面，这里执行的命令行与本机是一致的，不过你看到的是你服务器的一些情况

### 文件的上传和下载
- 上传
```
scp ceshi.txt root@127.987.883.888:~
```
这样文件会被传输到你的用户名下的文件夹下
- 下载
```
scp root@127.987.883.888:~／ceshi.txt ceshi.txt
```

### apache安装
-1. 运行 Terminal，输入命令：
```
ssh username@ip，然后输入密码。
```
2. 安装 Apache 软件：
```
yum install httpd
```
3. 设置 Apache 在服务器启动时运行：
```
chkconfig --levels 235 httpd on
```
4. 在 Apache 配置文件中配置域名：
```
vi /etc/httpd/conf/httpd.conf，找到 ServerName ，添加“域名:80”，保存并退出。
```
5. 重启 Apache：
```
service httpd restart
```
6. 浏览器中访问第4步配置的域名，如果出现“Apache 2 Test Page powered by CentOS”的页面，说明配置成功。

### django安装
- 安装工具
```
yum install setuptools
```
- pip安装
```
pip install Django
```

### 重新格式化云服务器之后登录不上怎么办
将本地的./ssh文件夹中的know_hosts里边对应的ip的记录删除掉，重新生成

### lunix中的复制文件以及文件夹
```
cp -r xxx xxx
```
-r代表我们复制其中的所有文件以及文件夹

### lunix删除文件
```
rm -rf xxx
```

### 在python 的django服务不可用
查看

### lunix升级python
```
首先下载源tar包

　　可利用linux自带下载工具wget下载，如下所示：

1# wget http://www.python.org/ftp/python/2.7.3/Python-2.7.3.tgz
　　下载完成后到下载目录下，解压

1	tar -zxvf Python-2.7.3.tgz
2	
　　进入解压缩后的文件夹

1	cd Python-2.7.3
　　在编译前先在/usr/local建一个文件夹python27（作为python的安装路径，以免覆盖老的版本）

1	mkdir /usr/local/python27
　　在解压缩后的目录下编译安装

1	./configure --prefix=/usr/local/python27
2	make
3	make install
　　此时没有覆盖老版本，再将原来/usr/bin/python链接改为别的名字

1	mv /usr/bin/python /usr/bin/python_old
　　再建立新版本python的链接

1
ln -s /usr/local/python27/bin/python2.7 
　　这个时候输入

1	python
　　就会显示出python的新版本信息

1	Python 2.7.3 (default, Sep 29 2013, 11:05:02)
2	[GCC 4.1.2 20080704 (Red Hat 4.1.2-54)] on linux2
3	Type "help", "copyright", "credits" or "license" for more information.
4	>>>
修改YUM   /usr/bin/yum

文件让yum能正常的工作：改成上面我们修改的PYTHON 2.6.6的名字
```

>lunix解压命令
```
1、*.tar 用 tar -xvf 解压

2、*.gz 用 gzip -d或者gunzip 解压

3、*.tar.gz和*.tgz 用 tar -xzf 解压

4、*.bz2 用 bzip2 -d或者用bunzip2 解压

5、*.tar.bz2用tar -xjf 解压

6、*.Z 用 uncompress 解压

7、*.tar.Z 用tar -xZf 解压

8、*.rar 用 unrar e解压

9、*.zip 用 unzip 解压
```

### 安装apex
```

lunix 安装apxs扩展

好像命令行安装的，自带了这个模块，手动编译的apache，同样需要手动编译这个模块，编译的时候可能会遇到这个错误：mod_wsgi.c Python.h：没有那个文件或目录,，解决方法：yum install python-devel，安装apache的apxs扩展。
python 2.7 No module named ‘zlib'

yum -y install zlib-devel openssl-devel
cd /wls/softwares/Python-2.7.10
./configure --prefix=/usr/local/python27 make make install

pip是python的包管理工具，我们通过pip来安装python所需要的一些模块，当然如果你有多版本存在，可能需要将pip这个模块copy到对应的python版本下边

在安装   Linux 系统是顺便把apache 服务装好了 ，这时这是装了一个服务不能进行二次开发，

所以很多的开发工具和文件在apache下找不到，比如模块编译工具apxs ，这时就要求安装 开发包，


命令 rpm -ql httpd-devel   ，然后会在  usr/local/下面多出个 apache2文件夹，里面有不少开发需要的文件。
如果提示没有apxs，那就先yum install httpd-devel
```

### lunix 安装git
```
1.2 CentOS6.6下

在CentOS5的时代，由于yum源中没有git，所以需要预先安装一系列的依赖包。但在CentOS6的yum源中已经有git的版本了，可以直接使用yum源进行安装。

$ sudo yum install git
但是yum源中安装的git版本是1.7.1，太老了，Github等需要的Git版本最低都不能低于1.7.2 。所以我们一般不用上面的方法。而是下载git源码，编译安装。

编译安装的步骤是【4】：

（1）首先先更新系统

sudo yum update
（2）安装依赖的包

sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
（3）下载git源码并解压缩

$ wget https://github.com/git/git/archive/v2.3.0.zip
$ unzip v2.3.0.zip
$ cd git-2.3.0
（4）编译安装

将其安装在“/usr/local/git”目录下。

make prefix=/usr/local/git all
sudo make prefix=/usr/local/git install
（5）此时你如果使用git --version 查看git版本的话，发现git仍然是1.7.1版本。这是因为它默认使用了"/usr/bin"下的git。

你可以用下面的命令查看git所在的路径：

$ whereis git
git: /usr/bin/git /usr/local/git /usr/share/man/man1/git.1.gz
（6）我们要把编译安装的git路径放到环境变量里，让它替换"/usr/bin"下的git。为此我们可以修改“/etc/profile”文件（或者/etc/bashrc文件）。

sudo vim /etc/profile
然后在文件的最后一行，添加下面的内容，然后保存退出。

export PATH=/usr/local/git/bin:$PATH
（7）使用source命令应用修改。

source /etc/profile

（8）然后再次使用git --version 查看git版本，发现输出2.3.0，表明安装成功。
```

### django错误
```
DisallowedHost at /

Invalid HTTP_HOST header: '10.211.55.6:8000'. You may need to add u'10.211.55.6' to ALLOWED_HOSTS.
Request Method:	GET
Request URL:	http://10.211.55.6:8000/
Django Version:	1.10.4
Exception Type:	DisallowedHost
Exception Value:	
Invalid HTTP_HOST header: '10.211.55.6:8000'. You may need to add u'10.211.55.6' to ALLOWED_HOSTS.
Exception Location:	/usr/lib/python2.7/site-packages/django/http/request.py in get_host, line 113
Python Executable:	/usr/bin/python
Python Version:

1，以上时我访问请求的时候出现的，原因在于Django框架中的创建的一个项目的时候，

2，跑下这个命令：Python manage.py runserver 10.211.55.5:8000

3，然后在我本机的浏览器中写入上述IP和端口请求过去：http://10.211.55.6:8000

4，于是就出现了最上面的那个问题；

5，于是就去django-admin.py startproject project-name创建的项目中去修改 setting.py 文件：

ALLOWED_HOSTS = ['*']  ＃在这里请求的host添加了＊


6，于是就成功的访问到了Django的项目了；
```
