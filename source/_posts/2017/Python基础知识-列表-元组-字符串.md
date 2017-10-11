---
title: Python基础知识-列表/元组/字符串
date: 2016-04-09 10:53:55
tags:
- Python
categories: Python
---
## 列表和元组
在python中我们有两种很重要的数据结构，分别为序列和映射，对应到OC中就是数组和字典，序列又分为列表和元组，他们的区别在于，列表可修改，元组不能，列表用方括号[]，而元组用小括号（）
<!--more-->
```
list = ['hanshenghui','xiaoming']
list1 = ('hanshenghui','xiaoming')
```
<!--more-->
### 序列通用操作
列表和元组都能做的操作
#### 索引
```
>>> greeting = 'hello'
>>> greeting[1]
'e'
>>> greeting[2]
'l'
>>> 

>>> greeting1 = ('h','e','l')
>>> greeting1[0]
'h'
>>> greeting1[1]
'e'
>>> 
```
字符串可以理解为由一个个的字符拼成的列表
#### 分片
我们不但可以用一个索引来获取某一个元素，也可以获取某个范围的元素。
```
>>> numbers = [1,2,3,4,5,6,7,8,9,0]
>>> numbers[0:2]
[1, 2]
```
我们可以看到，左边的边界包含在其中，而右边的边界是不包含在其中的
```
>>> numbers[-1:2]
[]
>>> numbers[-1:-3]
[]
>>> numbers[-3:-1]
[8, 9]
>>> 
>>> numbers[-3:]
[8, 9, 0]
```
我们还可以用倒序来取元素，但是切记一定要左边小，右边的大，只标记一个就会从这个开始取所有，或者到这个为止

还可以给一个步长
```
>>> numbers[::3]
[1, 4, 7, 0]
```
#### 相加
```
>>> item1 = [1,2,3]
>>> item2 = [4,5,6]
>>> item1 + item2
[1, 2, 3, 4, 5, 6]

>>> string1 = 'hansheng'
>>> string2 = 'hui'
>>> string1 + string2
'hanshenghui'

>>> string1 + item1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: cannot concatenate 'str' and 'list' objects
```
但是只有同种类型才能相加
#### 乘法
```
>>> 'python' * 5
'pythonpythonpythonpythonpython'
>>> [10,11] * 3
[10, 11, 10, 11, 10, 11]
```
#### 成员资格
```
>>> han = 'hanshenghui'
>>> 'shenghui' in han
True
>>> 'wahha' in han
False
>>> 
```

#### 长度，最大值，最小值
```
han = 'hanshenghui'
item1 = [1,2,3]

>>> len(han)
11
>>> max(han)
'u'
>>> min(han)
'a'
>>> len(item1)
3
>>> max(item1)
3
>>> min(item1)
1
>>> 
```

### 列表操作
#### list与join
```
>>> name = list('hanshenghui')
>>> name
['h', 'a', 'n', 's', 'h', 'e', 'n', 'g', 'h', 'u', 'i']
>>> len(name)
11
>>> ''.join(name)
'hanshenghui'
>>> 'a'.join(name)
'haaanasahaeanagahauai'
```
#### 赋值
更改某个元素的值
```
>>> name[0] = 'i'
>>> name
['i', 'a', 'n', 's', 'h', 'e', 'n', 'g', 'h', 'u', 'i']
```
#### 删除del
```
>>> name = 'han'
>>> name
'han'
>>> name = list(name)
>>> del name[0]
>>> name
['a', 'n']
>>> 
```
#### 分片赋值
这是异常强大的一个特性，可以用来插入元素，更改元素，甚至删除元素
```
>>> name = list('han')
>>> name[1:] = list('python')
>>> name
['h', 'p', 'y', 't', 'h', 'o', 'n']
>>> name[2:]=[]
>>> name
['h', 'p']
>>> name[1:1] = list('wahha')
>>> name
['h', 'w', 'a', 'h', 'h', 'a', 'p']
>>> 
```
#### append追加
```
>>> name = list('han')
>>> name
['h', 'a', 'n']
>>> name.append('shenghui')
>>> name
['h', 'a', 'n', 'shenghui']
>>> ''.join(name)
'hanshenghui'
>>> 
```
#### count统计出现的次数
```
>>> name = 'hanshenghui'
>>> name.count('h')
3
>>> name.count('n')
2
>>> 
```
#### extend
一个列表后追加一个新的列表，同加法作用一致
```
>>> name
['h', 'a', 'n']
>>> name1 = list('sheng')
>>> name1
['s', 'h', 'e', 'n', 'g']
>>> name.extend(name1)
>>> name
['h', 'a', 'n', 's', 'h', 'e', 'n', 'g']
>>> 
```
#### index
找出列表中某个项的第一个匹配索引
```
>>> name
['h', 'a', 'n', 's', 'h', 'e', 'n', 'g']
>>> name.index('h')
0
>>> 
```
#### insert
```
>>> name
['h', 'a', 'n', 's', 'h', 'e', 'n', 'g']
>>> name.insert(0,'wahha')
>>> name
['wahha', 'h', 'a', 'n', 's', 'h', 'e', 'n', 'g']
>>> 
```
#### pop
移除某个值并且将该值返回，需要提供索引
```
>>> name
['wahha', 'h', 'a', 'n', 's', 'h', 'e', 'n', 'g']
>>> name.pop()
'g'
>>> name.pop()
'n'
>>> name.pop(0)
'wahha'
>>> 
```
#### remove
移除某个匹配项
```
>>> name
['h', 'a', 'n', 's', 'h', 'e']
>>> name.remove('s')
>>> name
['h', 'a', 'n', 'h', 'e']
>>> name.remove('s')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: list.remove(x): x not in list
```
如果该值不存在，那么就会报错
#### reverse
```
>>> name
['h', 'a', 'n', 'h', 'e']
>>> name.reverse()
>>> name
['e', 'h', 'n', 'a', 'h']
```
#### sort
改变原来的列表，返回一个空值
```
>>> name
['e', 'h', 'n', 'a', 'h']
>>> name.sort()
>>> name
['a', 'e', 'h', 'h', 'n']
>>>
```
需要返回值，sorted
```
>>> name = list('hna')
>>> name1 = sorted(name)
>>> name1
['a', 'h', 'n']
>>> 
```

#### 高级排序
```
>>> name = list('hanshenghui')
>>> name.sort(reverse = True)
>>> name
['u', 's', 'n', 'n', 'i', 'h', 'h', 'h', 'g', 'e', 'a']
>>> 
```

## 字符串
### 字符串格式化
```
>>> python = '%s is a %s' 
>>> value = ('xiaohong','dog')
>>> print python % value
xiaohong is a dog
```
>符号说明:`s`代表字符串，后边的对象将以str转译，`%`代表占位，只有元组才会被分开解释，如果是一个列表，那么只能被解释为一个对象，这点要特别注意。d代表整型，f代表浮点数

```
>>> python = '%10.1fha'
>>> print python % 10
      10.0ha
```
f前边的小数点后的数字代表，小数点的保留位数，而前边的10代表字段宽。

```
>>> python = '%010.1fha'
>>> print python % 10
00000010.0ha

>>> python = '%+10.1fha'
>>> print python % 10
     +10.0ha

>>> print python % 10
10.0      ha
>>> 

```
在字段宽和精度前边还可以放置一个标志，这个标志可以是零，加号和减号或者空格，用法见上

### 字符串方法
#### 1、find
find方法可以在一个较长的字符串中查找子字符串。它返回子串所在位置的最左端索引。如果没有找到则返回-1。
```
>>> name = 'hanshenghui'
>>> name.find('h')
0
>>> name.find('h',2)
4
>>> name.find('h',2,6)
4
>>> name.find('h',2,5)
4
>>> name.find('h',2,3)
-1
```
#### 2、join && split
join方法是非常重要的字符串方法，它是split方法的逆方法，用来在队列中添加元素：
```
>>> s = ['1', '2', '3']
>>> '+'.join(s)
'1+2+3'
```
注意：需要添加的队列元素都必须是字符串。

split这是个非常重要的字符串方法，它是join的逆方法，用来将字符串分割成序列。
```
>>> '1+2+3+4'.split('+')
['1', '2', '3', '4']
>>> '1 2 3 4'.split()
['1', '2', '3', '4']
```
#### 3.lower && upper
lower方法返回字符串的小写字母版。
```
>>> name = 'A'
>>> name.lower()
'a'
>>> b = name.lower()
>>> b.upper()
'A'

S.lower() #小写 
S.upper() #大写 
S.swapcase() #大小写互换 
S.capitalize() #首字母大写 
```
#### 4.strip
strip方法返回去除两侧（不包含内部）空格的字符串
去两边空格：str.strip()
去左空格：str.lstrip()
去右空格：str.rstrip()
去两边字符串：str.strip('d')，相应的也有lstrip，rstrip

#### 5.replace && translate
translate方法和replace方法一样，可以替换字符串中的某些部分，但是和前者不同的是，translate方法只处理单个字符。它的优势在于可以同时进行多个替换，有些时候比replace效率高得多。

```
>>> name = 'hanshenghui'
>>> name.replace('h','z')
'zanszengzui'
```

在使用translate转换前，需要先完成一张转换表。转换表中是以某字符替换某字符的对应关系。因为这个表（事实上是字符串）有多达256个项目，我们还是不要自己写了，用string模块里面的maketrans函数就行了。

maketrans函数接收两个参数：两个等长的字符串，表示第一个字符串中的每个字符都用第二个字符串中相同位置的字符替换。
```
>>> from string import maketrans
>>> table = maketrans('cs', 'kz')
　　创建这个表后，可以将它用作translate方法的参数，进行字符串的转换：

>>> 'this is an incredible test'.translate(table)
'thiz iz an inkredible tezt'
translate的第二个参数是可选的，这个参数是用来指定需要删除的字符。

>>> 'this is an incredible test'.translate(table, ' ')
'thizizaninkredibletezt'
```

#### 6.字符串判断相关
字符串判断相关
是否以start开头：str.startswith('start')
是否以end结尾：str.endswith('end')
是否全为字母或数字：str.isalnum()
是否全字母：str.isalpha()
是否全数字：str.isdigit()
是否全小写：str.islower()
是否全大写：str.isupper()


#### 7.字符串比较
cmp方法比较两个对象，并根据结果返回一个整数。cmp(x,y)如果X< Y,返回值是负数 如果X>Y 返回的值为正数。
sStr1 = 'strch'
sStr2 = 'strchr'
print cmp(sStr1,sStr2)##-1

### 列表，元组，字符串的转换list与tuple
意义在于元组可以作为一个参数整体使用，而列表不可以
```
>>> name = 'hanshenghui'
>>> tuple(name)
('h', 'a', 'n', 's', 'h', 'e', 'n', 'g', 'h', 'u', 'i')
>>> list(name)
['h', 'a', 'n', 's', 'h', 'e', 'n', 'g', 'h', 'u', 'i']
>>> tuple(list(name))
('h', 'a', 'n', 's', 'h', 'e', 'n', 'g', 'h', 'u', 'i')
```
