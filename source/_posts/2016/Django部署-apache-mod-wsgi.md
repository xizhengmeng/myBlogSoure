---
title: Django部署+apache+mod_wsgi
date: 2016-11-09 14:25:31
tags:
- Python
categories: Python
---

## Django的部署(apache + mod_wsgi)

### 概念解析
#### Django
Django是一个框架，可以用python来开发web应用，但是并不是独立的开发web，在UI放面依然需要依靠html+css+js，用这些来搭建界面，不过我们可以使用django无缝的使用python的逻辑功能，比如使用python来处理数据，然后交给页面来进行展示 
>Django本身是有web服务器的，但是这个服务器只能在本机提供web服务，其作用主要是用来做测试，如果想其他机器也能访问你的web这个时候你还需要另外一个服务器，提供网络访问的功能
<!--more-->
#### apache
Apache是专门用了提供HTTP服务的，以及相关配置的（例如虚拟主机、URL转发等等）

#### mod_wsgi
mod_wsgi的目的是实现一个简单的使用Apache模块可以举办任何Python应用程序支持Python的WSGI接口。该模块将适用于主机的高性能生产的网站，以及一般的自我管理个人网站的网页寄存服务运行。）

#### 虚拟主机
在apache上配置虚拟主机，这将在你mac上提供一个简单管理多个网站的框架

#### 静态文件
django1.3新加入了一个静态资源管理的app，django.contrib.staticfiles。在以往的django版本中，静态资源的管理一向都是个问题。部分app发布的时候会带上静态资源文件，在部署的时候你必须手动从各个app中将这些静态资源文件复制到同一个static目录。在引入staticfiles后，你只需要执行./manage.py collectstatic就可以很方便的将所用到app中的静态资源复制到同一目录。
staticfiles的引入，方便了django静态文件的管理，不过感觉staticfiles的文档写的并不是太清楚，初次使用的时候还是让我有些困惑。
下面简单的介绍一下staticfiles的主要配置：

-  ` STATIC_ROOT`：运行manage.py collectstatic后静态文件将复制到的目录。注意：不要把你项目的静态文件放到这个目录。这个目录只有在运行collectstatic时才会用到。我最开始想当然的以为这个目录和MEDIA_ROOT的作用是相同的，致使在开发环境下一直无法找到静态文件。

- `STATIC_URL`：设置的static file的起始url，这个只可以在template里面引用到。这个参数和MEDIA_URL的含义差不多。

-  `STATICFILES_DIRS`：除了各个app的static目录以外还需要管理的静态文件位置，比如项目公共的静态文件差不多。和TEMPLATE_DIRS的含义差不多。
各个APP下static/目录下的静态文件django的开发服务器会自动找到，这点和以前APP下的templates目录差不多。
在urls.py中加入静态文件处理的代码
```
from django.contrib.staticfiles.urls import staticfiles_urlpatterns
# ... the rest of your URLconf goes here ...
urlpatterns += staticfiles_urlpatterns()
```

### 各种目录
- django项目目录
- 静态文件目录
- 错误日志目录

### 请求的处理过程
sitename.conf --> wsgi.py --> settings.py --> urls.py --> views.py
### 配置的过程
- python是系统自带不需要安装
- apache系统自带不需要安装
#### mod_wsgi安装
- `brew tap homebrew/apache`
- `brew install mod_wsgi`

#### django安装及工程创建
- 安装pip`sudo easy_install pip`
- 利用pip安装django`（sudo) pip install Django`
- 检验是否安装成功
```
>>> import django
>>> django.VERSION
(1, 7, 6, 'final', 0)
>>> 
>>> django.get_version()
'1.7.6'
```
- 创建工程
```
django-admin.py startproject project-name
```
- 创建app
到工程的目录下，找到namage.py这个文件
```
python manage.py startapp app-name
```
#### apache配置
/etc/apache2/httpd.conf
这个文内
- `LoadModule wsgi_module /usr/local/Cellar/mod_wsgi/4.5.6/libexec/mod_wsgi.so`这个目录是安装mod_wsgi这个文件的时候，生成的，找到它
- `Include /private/etc/apache2/extra/httpd-vhosts.conf`放开这句话，意思是我要使用虚拟主机
- 配置虚拟主机在httpd-vhosts.conf这个文件内导入文件
```
include /private/etc/apache2/extra/vhosts/localhost.conf
include /private/etc/apache2/extra/vhosts/dev.mysite.com.conf
```
意思是导入这两个文件，作为虚拟主机的配置，但是这两个文件需要自己创建
- 虚拟主机配置`dev.mysite.com.conf`
这个是最关键的地方
```
Listen 89#虚拟主机的监听端口
<VirtualHost *:89>

LogLevel info

ServerName dev.mysite.com#虚拟主机的名字
ServerAdmin my@mysite.com

ErrorLog "/Users/han/Sites/logs/mysite.com-error_log"
CustomLog "/Users/han/Sites/logs/mysite.com-access_log" common#这两个一定要配置，因为一旦有错误，我们需要依靠这两个文件的内的日志来找到错误


WSGIScriptAlias / /Users/han/Documents/mysite/mysite/wsgi.py#这个是整个设置最重要的地方，让apache可以知道使用哪个wsgi

<Directory "/Users/han/Documents/mysite/mysite">
<Files wsgi.py>
Require all granted
</Files>
</Directory>#设置wsgi的文件权限

#################下边的都不是重点了##################
# Static files
DocumentRoot "/Users/han/Sites/mysite.com"
<Directory "/Users/han/Sites/mysite.com">
Options FollowSymLinks Multiviews
Require all granted
</Directory>

Alias /static/ /Users/han/Sites/mysite.com/static/

<Directory "/Users/han/Sites/mysite.com/static">
Require all granted
</Directory>

# WGSI configuration
#WSGIDaemonProcess dev.mysite.com processes=2 threads=15 display-name=%{GROUP} python-#path=/Users/han/Documents/mysite/:/Users/han/Documents/VirtualEnvs/python2.7-django/lib/python2.7/site-#packages

#WSGIProcessGroup dev.mysite.com

<Directory "/Users/han/Documents/mysite">
Options FollowSymLinks Multiviews
Require all granted
</Directory>


</VirtualHost>
```

#### django工程配置
设置wsgi文件，让mod_wsgi知道，使用哪个django项目
```
import os

from os.path import join,dirname,abspath

PROJECT_DIR = dirname(dirname(abspath(__file__)))#3
import sys # 4
sys.path.insert(0,PROJECT_DIR) # 5

from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")

application = get_wsgi_application()
```
最重要的是3，4，5三句，让mod_wsgi知道去哪里找django项目，剩下的就是django的事情了

### 错误说明
>其实所有的static文件都是非重点，因为我们最终调用的是django的服务，所以没有静态文件，只是不展示而已，不过如果我们需要一些小的icon等等，我们还是需要这个静态文件的，这是后话，先实现服务这是大事

如果我们遇到了权限问题，
>Permission denied: access to / denied (filesystem path '/Users/han/Documents/mysite use search permissions are missing on a component of the path

像下边这样的，说明我们没有合适的权限，这是所有文档都没有说明白的地方，这里之所以回访问这个是因为在mod_wsgi中进行了设置，mod_wsgi去这个文件中寻找django的服务，但是我们并没有在所有的设置中设置mod_wsgi的访问权限，这是其一，其二我们并不能直接设置mysite的权限，这样到最后我们发现还是一直报这个错误，我们需要设置上层目录的权限，不知道为啥会这么坑爹
到～目录下，然后
`chmod o+rx Documents`
最后我们输入`dev.mysite.com:89`的时候显示的就是django的首页了

### 常用命令行
- `sudo apachectl restart`重启命令
- `apachectl confittest`修改配之后测试一下看看哪里有错误，只出现一个OK才是没有问题

