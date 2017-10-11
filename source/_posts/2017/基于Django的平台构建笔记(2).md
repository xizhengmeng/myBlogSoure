---
title: 基于Django的平台构建笔记(2)
date: 2017-05-18 09:57:42
tags:
- Python
- Django
- JS
---

### python中Json字符串的创建与javascript中json字符串的转换
<!--more-->
```
import json

def getindex(indexname):

    datas = []

    for item in namelist:
    
        pes.append(dic) 
        pes = json.dumps(pes)
        datas.append(pes)

    datas = json.dumps(datas)
    return datas
```
仔细看上边这段python我们能得出来一个结论，我们可以使用json这个功能，将一个python中的列表，转换为一段字符串，然后返回出去，这个字符串就是一段json字符串，我们可以将这段json字符串反过来转换成一个数组

下面我们来瞅瞅，如何将一个json字符串转换为一个json数组

>首先说明基本功能：

dumps是将dict转化成str格式，loads是将str转化成dict格式。

dump和load也是类似的功能，只是与文件操作结合起来了。


看代码实例：

```
In [1]: import json

In [2]: a = {'name': 'wang', 'age': 29}

In [3]: b = json.dumps(a)

In [4]: print b, type(b)
{"age": 29, "name": "wang"} <type 'str'>

In [11]: json.loads(b)
Out[11]: {u'age': 29, u'name': u'wang'}

In [12]: print type(json.loads(b))
<type 'dict'>
```

然后再看dump和dumps的区别，见代码：

```
In [1]: import json

In [2]: a = {'name': 'wang', 'age': 29}

In [3]: b = json.dumps(a)

In [4]: print b, type(b)
{"age": 29, "name": "wang"} <type 'str'>

In [5]: c = json.dump(a)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-5-92dc0d929363> in <module>()
----> 1 c = json.dump(a)

TypeError: dump() takes at least 2 arguments (1 given)
```

这里提示我们少一个参数，我们看一下帮助文件（iPyhton中可以直接使用help(json.dumps)来查看帮助文件）：



dumps(obj, skipkeys=False, ensure_ascii=True, check_circular=True, allow_nan=True, cls=None, indent=None, separators=None, encoding='utf-8', default=None, sort_keys=False, **kw)
Serialize ``obj`` to a JSON formatted ``str``.



dump(obj, fp, skipkeys=False, ensure_ascii=True, check_circular=True, allow_nan=True, cls=None, indent=None, separators=None, encoding='utf-8', default=None, sort_keys=False, **kw)
Serialize ``obj`` as a JSON formatted stream to ``fp`` (a
``.write()``-supporting file-like object).



简单说就是dump需要一个类似于文件指针的参数（并不是真的指针，可称之为类文件对象），可以与文件操作结合，也就是说可以将dict转成str然后存入文件中；而dumps直接给的是str，也就是将字典转成str。

例子见代码（注意文件操作的一些小细节）：

```
In [1]: import json

In [2]: a = {'name': 'wang'}

In [3]: fp = file('test.txt', 'w')

In [4]: type(fp)
Out[4]: file

In [5]: json.dump(a, fp)

In [6]: cat test.txt

In [7]: fp.close()

In [8]: cat test.txt
{"name": "wang"}
In [9]: json.load(fp)
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-9-0064dabedb17> in <module>()
----> 1 json.load(fp)

/usr/local/Cellar/python/2.7.11/Frameworks/Python.framework/Versions/2.7/lib/python2.7/json/__init__.pyc in load(fp, encoding, cls, object_hook, parse_float, parse_int, parse_constant, object_pairs_hook, **kw)
285
286     """
--> 287     return loads(fp.read(),
288         encoding=encoding, cls=cls, object_hook=object_hook,
289         parse_float=parse_float, parse_int=parse_int,

ValueError: I/O operation on closed file

In [10]: fp = file('test.txt', 'r')

In [11]: json.load(fp)
Out[11]: {u'name': u'wang'}
注：实际中dump用的较少。

```

### javascript中如何将一段json字符串转换为json对象


var obj = JSON.parse(data);

### web中上传文件，带进度条的做法
```
<div id="upload">

     <form>
        {% csrf_token %}
        <input type="file" id='file' class='file' style="margin-top: 5px;margin-left: 10px" name="file"><br>
        <div id='1' style="margin-top: 40px;margin-left: 10px;height:10px;width:200px;border-color: rgba(110,3,120,1);border:1px solid gray;float:left;margin-right:10px;">
            <div id='2' style="height:100%;width:0px;background:rgba(220,220,220,1);"></div>
        </div>
        <a style="position: absolute;top: 72px;;margin-right:25px" id='3'>0%</a>

        <button type="button" style="margin-top: 32px;margin-left: 50px;" onclick="upload();">上传</button>

    </form>
</div>
```
是要定义一个file,下边是js的代码，我们来实现上传功能

```
function upload() {

            var apktextinput = document.getElementById('apkname')
            var apkname = apktextinput.value

            if (apkname.length == 0) {
                alert('请输入证书名称')
                return
            }

			var xhr = new XMLHttpRequest();
			var file = document.getElementById('file').files[0];   //取得文件数据，而.file对象只是文件信息
			var form = new FormData();   //FormData是HTML5为实现序列化表单而提供的类，更多细节可自行查询
            form.append('file',file);   //这里为序列化表单对象form添加一个元素，即file
			xhr.upload.addEventListener('progress',on_progress,false);     //xhr对象含有一个upload对象，它有一个progress事件，在文件上传过程中会被不断触发，我们为这个事件对应一个处理函数，每当事件触发就会调用这个函数，于是便可利用这个函数来修改当前进度，更多细节可自行查询
			xhr.open('POST','http://10.13.8.12:89/uploadAPK/',true);  //请将url改成上传url
            xhr.setRequestHeader('X-CSRFTOKEN','{{ request.COOKIES.csrftoken }}');   //此处为Django要求，可无视，或者换成相应后台所要求的CSRF防护，不是django用户请去掉
			xhr.send(form);   //发送表单
		}
```
重点是`xhr.upload.addEventListener('progress',on_progress,false);`这个方法会保证上传过程中不断的回调我们来实现on_progress

```
 function on_progress(evt) {       //看这个函数之前先看upload函数。这个函数可以接收一个evt(event)对象(细节自行查询progress)，他有3个属性lengthComputable，loaded，total，第一个属性是个bool类型的，代表是否支持，第二个代表当前上传的大小，第三个为总的大小，由此便可以计算出实时上传的百分比
			if(evt.lengthComputable) {
				var ele = document.getElementById('2');
				var percent = Math.round((evt.loaded) * 100 / evt.total);
                if (percent == 100 && strongapk == 1) {
                    strongapk = 0
                    var textcontent = document.getElementById("wating")
                    textcontent.style.display = 'block'

                    var apktextinput = document.getElementById('apkname')
                    var apkname = apktextinput.value

                     $.ajax({
                        url: "/startstrongapk/",    //后台webservice里的方法名称
                        data:{"apkname":apkname},
                        type: "post",
                        traditional: true,
                        success: function (data) {
                            downloadurl = data
                            alert(data)
                            $("#wating").css('display','none');
                            if (confirm("你确定要下载文件吗？")) {
                                self.location=('downloadsdk?url=' + data)
                            }
                        },
                        error: function (msg) {
                            alert(msg)
                            $("#wating").css('display','none');
                        }
                   });

                }
				ele.style.width = percent + '%';
				document.getElementById('3').innerHTML = percent + '%';
			}
		}
```
这里我们在percent达到100之后，调用ajax做一些事情

### django中接口调用的方式总结
网络方法调用大致有这么几种
#### 点击按钮跳转新页面
这种直接用<a>标签，然后用href就好

```
<a class="button" id="ios" style="position: absolute;top: 50px;" href="jinkensios" onmouseover="showDetailText(this)" onmouseout="clearTest()">● 打包(iOS)</a>
```
#### 点击按钮刷新局部数据
##### 这种最好是预先埋伏好ajax，然后用id绑定，点击之后直接调用

```
$(document).ready(function(){
      $("#btn").click(function(){
        string = $("#textarea1").val()
        $.get("/ajaxcreateui/", {'text': string},function(ret){
            $('#textarea2').html(ret)
        })
      });
    });
```
- `$(document).ready(function(){}`预先埋伏的写法要写在这句代码里边
- `$("#btn").click(function(){}`这个是预先埋伏
- `$.get("/ajaxcreateui/", {'text': string},function(ret){}`ajax的get调用

##### 直接使用XMLHttpRequest

```
var progresshttp;
function getprogress() {

    progresshttp=null;
    if (window.XMLHttpRequest)
      {// code for all new browsers
      progresshttp=new XMLHttpRequest();
      }
    else if (window.ActiveXObject)
      {// code for IE5 and IE6
      progresshttp=new ActiveXObject("Microsoft.XMLHTTP");
      }

    if (progresshttp!=null)
      {
          progresshttp.onreadystatechange=progressstate_Change;
          progresshttp.open("POST",'getpackprogress',false);
          var formData = new FormData();
          formData.append('branchname', branchname);
          progresshttp.send(formData);
      }
    else
      {
      alert("Your browser does not support XMLHTTP.");
      }
}

function progressstate_Change()
{
if (progresshttp.readyState==4)
  {// 4 = "loaded"
  if (progresshttp.status==200)
    {// 200 = OK

      num = this.responseText

      var ele = document.getElementById('12');
      var percent = num * 100;
      percent = percent.toFixed(2)
      if (percent >= 100.00) {
          percent = 100.00
      }
      ele.style.width = percent + '%';
      document.getElementById('13').innerHTML = percent + '%';
      if (percent >= 100) {
          window.clearInterval(waitinterval)
          ele.style.width = 100 + '%';
          var text = document.getElementById('building')
          text.innerHTML = ""
      }
    }
  else
    {
    alert("Problem retrieving XML data");
    }
  }
}
```

添加回调的方式
>- `xhr.upload.addEventListener('progress',on_progress,false);`进度条
>- `progresshttp.onreadystatechange=progressstate_Change`事件回调

#### 某个事件调用结束刷新局部数据
直接使用ajax或者XMLHttpRequest就好

### python中调用命令行随时输出的做法

```
p = subprocess.Popen("gradle assembleDebug",shell=True,stdout=subprocess.PIPE)

indexcount = 0

returncode = p.poll()

while returncode is None:
      text = p.stdout.readline().strip()
      f = open('/Users/jdjr/Desktop/buildlog.txt','a')#追加打开方式
      f.write(text)
      f.close()
        
      returncode = p.poll()
```



