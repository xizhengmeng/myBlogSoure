---
title: 基于Django的自动化平台构建笔记(1)
date: 2016-12-15 11:30:54
tags:

---
现在提供的服务主要有：
- 根据property创建UI代码
- JSON自动转模型
- 自动打包服务
- 其他效率工具的链接
<!--more-->

其中最重要的，也是最复杂的应该是自动打包的服务，这套服务用python语言来实现，基于python的django框架，其中django框架主要用来提供web服务，用来给用户访问，而服务的执行者，比如说UI的代码创建，需要用到字符串解析等功能，完全由python在服务器端来完成，我们通过网络将需要解析的数据发送给服务器，然后服务器做完处理之后，再通过网络将数据返回，然后通过django来完成展示。再到打包服务也是这种逻辑，通过web服务来发送指令给服务器，然后服务器调用命令行来完成打包的具体的动作

### Django使用简解
Django是一个python的项目，我们可以使用WebStorm来编辑该项目。当运行的时候会生成一个本地的web服务，我们可以通过该地址，在浏览器访问这个服务，当然我们也可以在命令行来达到同样的效果，进入到项目根目录，manager.py的所在目录，执行`python manager.py runserver`，然后点击给出的地址，我们仍然可以查看服务。

对于这个新建的django工程我们主要关注的是两个文件，一个是ProjectName/urls.py，另一个是appName/views.py。
#### urls.py
这个文件时整个工程的路由，将url与具体的服务或者要展示的页面相对应。
```
from django.conf.urls import url
from django.contrib import admin
from learn import views as learn_views  # new
from django.contrib.staticfiles.urls import staticfiles_urlpatterns


urlpatterns = [
url(r'next', learn_views.nextPage),  # new
url(r'^$', learn_views.home,name='home'),  # new
url(r'^admin/', admin.site.urls),
url(r'callPython', learn_views.compute),  # new
url(r'ajaxpack', learn_views.ajaxpack),
url(r'ajaxgetbranches', learn_views.ajaxgetbranchesfunc),
url(r'add', learn_views.compute),  # new
url(r'uicreate', learn_views.uicreate),
url(r'ajaxcreateui', learn_views.createui),
url(r'package', learn_views.package),
url(r'buildlist', learn_views.buildlist),
url(r'ajaxsendMail', learn_views.sendMails),
url(r'jsonFormatClick1', learn_views.jsonFormat1),
url(r'jsonFormatClick2', learn_views.jsonFormat2),
url(r'interfaceTestClick', learn_views.interfaceTest),
url(r'url1', learn_views.getUrl1),
url(r'url2', learn_views.getUrl2),  # new
url(r'url3', learn_views.getUrl3),
url(r'url4', learn_views.getUrl4),
url(r'url5', learn_views.getUrl5),  # new
url(r'url6', learn_views.getUrl6),
url(r'url7', learn_views.getUrl7)
]

# ... the rest of your URLconf goes here ...
urlpatterns += staticfiles_urlpatterns()
```
看上边这段代码，新加入的有`from learn import views as learn_views  # new`这个是引入了learn这个app中的views这个文件，然后在这个文件中我们就可以使用这个文件了，我们可以在列表中添加很多的url与服务的对应，url的写法的讲究在于，`next`意味着包含next就跳这个，`^next`意味着以next开头，`next$`意味着以next结尾，`^next$`意味着直邮next才跳这个，完全对应的意思，这个文件的作用主要就这些

#### views.py
具体的服务我们是由这个文件来提供的
```
from django.http import HttpResponse
from django.shortcuts import render
from DIY.compute import getModelFromJson
from DIY.createui import getCreatedStringWithProperties
from DIY.packServer import packaged,getbranches
from django.http import HttpResponseRedirect
from DIY.mail import sendMail

def home(request):
string = '这是'
return render(request,'home.html',{'string':string})


def nextPage(request):
return render(request,'next.html')

def compute(request):
string = request.POST['text']
string = getModelFromJson(string)
return HttpResponse(string)

def uicreate(request):
return render(request,'UICreate.html')

def createui(request):
string = request.GET['text']
string = getCreatedStringWithProperties(string)
return HttpResponse(string)

def package(request):
return render(request,'package.html')

def ajaxpack(request):

f = open('/Users/wxg/Documents/Build/building.txt','r')
text = f.read()
if text == '':
string = request.GET['text']
resurtString = packaged(string)
return HttpResponse('done')
else:
return HttpResponse('wait')   

def ajaxgetbranchesfunc(request):
return HttpResponse(getbranches('string'))

def buildlist(requets):
return HttpResponseRedirect('http://10.13.80.19:8000/Documents/Build')

def sendMails(request):
sendMail('')
return HttpResponse('hi')  

def jsonFormat1(request):
return HttpResponseRedirect('http://www.jsonparseronline.com')

def jsonFormat2(request):
return HttpResponseRedirect('http://www.sojson.com')    

def interfaceTest(request):
return HttpResponseRedirect('http://www.atool.org/httptest.php')

def getUrl1(request):
f = open('/Users/han/Desktop/text1.txt', 'r')
text = f.read()
f.close()
return HttpResponse(text)

def getUrl2(request):
f = open('/Users/han/Desktop/text2.txt', 'r')
text = f.read()
f.close()
return HttpResponse(text)

def getUrl3(request):
f = open('/Users/han/Desktop/text3.txt', 'r')
text = f.read()
f.close()
return HttpResponse(text)

def getUrl4(request):
f = open('/Users/han/Desktop/text4.txt', 'r')
text = f.read()
f.close()
return HttpResponse(text)

def getUrl5(request):
f = open('/Users/han/Desktop/text5.txt', 'r')
text = f.read()
f.close()
return HttpResponse(text)

def getUrl6(request):
f = open('/Users/han/Desktop/text6.txt', 'r')
text = f.read()
f.close()
return HttpResponse(text)

def getUrl7(request):
f = open('/Users/han/Desktop/text7.txt', 'r')
text = f.read()
f.close()
return HttpResponse(text)
```
我们拿具体的例子来说，上边的这些代码，就是我们能够提供的具体的服务，具体说来我们主要提供三种服务
##### 返回json
```
def compute(request):
string = request.POST['text']
string = getModelFromJson(string)
return HttpResponse(string)
```
比如这个，需要引入
from django.http import HttpResponse

##### 返回一个html文件
```
def home(request):
string = '这是'
return render(request,'home.html',{'string':string})
```
中间的参数`home.html`就是一个html的文件，那么问题来了，这个文件是从哪里来的，与views.py同级有个templates文件夹，里边是我们要存放的html文件，这个需要html和css的一些知识来编写

##### 返回跳一个其他的url，比如跳baidu或者条服务器的某个文件夹都是可以的

```
def jsonFormat1(request):
return HttpResponseRedirect('http://www.jsonparseronline.com')
```

```
def buildlist(requets):
return HttpResponseRedirect('http://10.13.80.19:8000/Documents/Build')
```

还有个问题需要说明就是，我们这里需要很多python的服务，如何引入呢，直接引入函数的名称就可以了，就像是`from DIY.packServer import packaged,getbranches`

Django安装和配置具体可以参考[Django部署+apache+mod_wsgi](http://hanson647.com/2016/11/09/2016/Django部署-apache-mod-wsgi/)

### 自动打包服务的构建过程
这里说两点，第一个是我们实现的一些比较不常见的功能，一个是踩到的坑。
先来说一下功能点：
- 线上和预发的切换
- 清除本分支做的操作
- .app转换为.ipa
- xcode XCBuildConfiguration配置文件自动设置

#### .app转换为.ipa
当我们执行了`xcodebuild build`的命令之后，形成的是一个.app文件，这个时候我们需要做的压缩这个文件，这个是ipa文件生成的原理
具体步骤：
- Step1: 新建“Payload”文件夹，注意名字要一字不差；
- Step2: 将你的.app包放到Payload中，注意app的名字不做任何更改，就用xcode生成的app名称；
- Step3: 在Payload文件夹上右键压缩成zip，然后将生成的.zip文件后缀改成.ipa即可
具体代码:
```
import os,shutil

os.mkdir('Payload')

shutil.copytree('JDMobile.app','Payload/JDMobile.app')

os.system('zip -r Payload.zip Payload')

files=os.listdir(".")

for filename in files:
    li=os.path.splitext(filename)
    if li[1]==".zip":
        newname=li[0]+".ipa"
        os.rename(filename,newname)

shutil.rmtree('JDMobile.app')
shutil.rmtree('Payload')
```


#### xcode XCBuildConfiguration配置文件自动设置
这里使用mod-pbxproj，这个是一个python解析库，用法如下
```
from mod_pbxproj import XcodeProject
import commands

def configProject():

    project = XcodeProject.Load('/Users/wxg/Documents/JDMobileNew/JDMobile_2.0/JDMobile.xcodeproj/project.pbxproj')

    for item in project.objects.values():
        nameIsa = item.get('isa')

        if (nameIsa == 'XCBuildConfiguration'):
            setting = item.get('buildSettings')
            nameReference = item.get('baseConfigurationReference')
            if (nameReference):
               idStr = setting.get('PRODUCT_BUNDLE_IDENTIFIER')
               print '---->' + idStr
               setting.__setitem__('PRODUCT_BUNDLE_IDENTIFIER','com.jd.jinrong2016')
               setting.__setitem__('PROVISIONING_PROFILE','')
               setting.__setitem__('PROVISIONING_PROFILE_SPECIFIER','')
               setting.__setitem__('DEVELOPMENT_TEAM','5TKVHWTT79')
               setting.__setitem__('CODE_SIGN_IDENTITY[sdk=iphoneos*]','iPhone Developer')
               item.__setitem__('buildSettings',setting)
            else:
               codeSign = setting.get('CODE_SIGN_IDENTITY')
               profile = setting.get('PROVISIONING_PROFILE')
               print codeSign
               print profile
               setting.__setitem__('CODE_SIGN_IDENTITY','iPhone Distribution: Beijing Jingdong Century Trading Co., Ltd. (TQZTTUQ9ZE)')
               setting.__setitem__('PROVISIONING_PROFILE','0d8cd55a-c922-4f27-b1aa-df6a2f277ea5')
               item.__setitem__('buildSettings',setting)

        elif (nameIsa == 'PBXProject'):
            attributes = item.get('attributes')
            targetAttributes = attributes.get('TargetAttributes')
            targets = item.get('targets')
            tar = targets[0]
            attr = targetAttributes.get('%s' % tar)
            developmentTeamName = attr.get('DevelopmentTeamName')
            developmentTeamName = attr.get('ProvisioningStyle')
            print developmentTeamName
            attr.__setitem__('DevelopmentTeamName','Beijing Jingdong Century Information Technology Co., Ltd.')
            attr.__setitem__('ProvisioningStyle', 'Automatic')
            attr.__setitem__('DevelopmentTeam', '5TKVHWTT79')

    project.save()

```


这里的坑主要是由权限引起的，因为我们从web服务去调用一个命令行的指令的时候，这个时候的并不是登录状态的权限，而是一个非登录状态的权限，这个时候有两个问题，一个是很多服务是没有被加载的，第二个是一些文件没有权限去调用，下面分别来说

- gradle无法调用
- 苹果的开发者证书不能读取
- git pull命令需要用户名和密码

#### gradle无法调用
这里涉及到一个问题是mac的环境变量，通过在终端输入`$PATH`来查看当前用户下加载的路径有哪些，如果返回的路径中包含我们的服务的路径，那么肯定这个服务当前用户是可以调用的，我们在web服务中调用这个命令发现返回的只有`usr/bin`等着几个路径，那么思路就来了，如果gradle想要被使用，那么就需要加到这路径下，解决的办法就是加一个软连接到/usr/bin文件下，sudo ln -s xxxxx xxxxx就可以了，那么如果找到这个服务的安装路径呢？以gradle为例`which gradle`，如果是git的话就是`which git`
这里还有个问题是mac升级系统后，这个文件是不允许更改的，解决办法:

>对于Mac OS X 10.11 El Capitan用户，由于系统启用了SIP(System Integrity Protection), 导致root用户也没有权限修改/usr/bin目录。按如下方式可恢复权限。
屏蔽方法：重启Mac，按住command+R,进入recovery模式。选择打开Utilities下的终端，输入：csrutil disable并回车，然后正常重启Mac即可

#### 苹果的开发者证书不能读取
User interaction is not allowed这个问题是因为证书不能被读取，这是因为证书在登录下，我们将证书移动到系统下就可以了
#### git pull命令需要用户名和密码
这个不能被执行也是因为权限的问题，最后的解决方案，是在登录状态下(命令行)直接开一个新的线程，死循环不断的检测一个文件下是否有对应的文件夹，如果有这个文件夹就执行`git pull`命令，执行完毕之后写入日志，然后删除文件内容，继续进入到下一个循环，这个与iOS的runloop是一个道理

### django中使用ajax的post方法
```
 $("#insert").click(function(){
            text =  $('#textarea1').val()
            $.post("/writecontent/", {'content': text,'foldername':foldername,'filename':filename},function(ret){
                alert(ret)
            })
        });
```
之所以使用post这里是因为无法突破apache对于get方法的整体参数长度限制，而使用post方法有个问题就是会强制CSRF校验，解决方法就是关闭django的验证，具体来说是到setting文件中注释一行代码
>django.middleware.csrf.CsrfViewMiddleware

### 安装pymongo
首先要安装pip，然后用pip安装pymongo
```
sudo easy_install pip
```
然后使用pip安装pymongo
```
sudo pip install pymongo
```
这个时候进入python环境使用命令行去访问pymongo应该是没有问题的，但是你使用web调用的方式去调用不一定能够访问到mongo，这可能是因为你使用的python安装路径的问题
```
which python
```
看一下当前加载的python，不出意外应该是/usr/local/bin/python，这个是登录状态下才能加载的一个环境和路径，而web是非登录的，所以很多功能访问不到正常，我们要做的就是让系统直接加载/usr/bin/python，这就涉及到一个加载优先级的问题
```
sudo emacs /etc/paths
```

>/usr/local/bin
/usr/bin
/bin
/usr/sbin
/sbin

>tips:
現在要把 /usr/local/bin 移到上面去

control + k：把一行字剪下來

control + y：把字貼上

control + x + s：存檔

control + x + c：關掉 emacs

然后再执行下边这句
```
echo $PATH
```
这个时候你再打开
发现已经变成了下边这样
>/usr/bin
/usr/local/bin
/bin
/usr/sbin
/sbin

这个时候你再执行
```
which python
```
发现已经变成了`/usr/bin/python`
现在看来，系统加载某个功能的逻辑，就是直接在这些个加载列表中找这些个功能，如果逐个加载了一遍发现没有这个功能，那么就会报错，如果第一路径下有这个功能，第二路径下也有这个功能，那么就会用第一路径下的，所以我们更改第一路径是有意义的
