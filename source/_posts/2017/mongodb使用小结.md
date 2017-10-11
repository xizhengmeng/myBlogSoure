---
title: mongodb使用小结
date: 2017-05-17 14:46:54
tags:
- mongo
---

### 创建一个mongo数据库
<!--more-->
```
from pymongo import MongoClient
import json,time

connection = MongoClient("localhost",27017)
mydb = connection.mydb # new a database
myser = mydb.allindex # new a table
```

### 查

```
dicn = {}
dicn['indexname'] = item
dicn['date'] = dateString
dbs = myser.find(dicn)

for item in dbs:

  dic = {}
  datestring = item['date']
  datestring = datestring.encode('utf-8')
  dic['date'] = datestring
  dic['earyeild'] = item['earyeild']
  dic['indexname'] = item['indexname']


```

> 

MongoDB提供了一组比较操作符：$lt/$lte/$gt/$gte/$ne，依次等价于</<=/>/>=/!=。
-  下面的示例返回符合条件age >= 18 && age <= 40的文档。
```
> db.test.find({"age":{"$gte":18, "$lte":40}})
{ "_id" : ObjectId("4fd58ecbb9ac507e96276f1a"), "name" : "stephen", "age" : 35,"genda" : "male", "email" : "stephen@hotmail.com" }
```

- 下面的示例返回条件符合name != "stephen1"
```
> db.test.find({"name":{"$ne":"stephen1"}})
{ "_id" : ObjectId("4fd58ecbb9ac507e96276f1a"), "name" : "stephen", "age" : 35,"genda" : "male", "email" : "stephen@hotmail.com" }
```

- $in等同于SQL中的in，下面的示例等同于SQL中的in ("stephen","stephen1")
```
> db.test.find({"name":{"$in":["stephen","stephen1"]}})
{ "_id" : ObjectId("4fd58ecbb9ac507e96276f1a"), "name" : "stephen", "age" : 35,"genda" : "male", "email" : "stephen@hotmail.com" }  
```

- 和SQL不同的是，MongoDB的in list中的数据可以是不同类型。这种情况可用于不同类型的别名场景。
```
> db.test.find({"name":{"$in":["stephen",123]}})
{ "_id" : ObjectId("4fd58ecbb9ac507e96276f1a"), "name" : "stephen", "age" : 35,"genda" : "male", "email" : "stephen@hotmail.com" } 
```

- $nin等同于SQL中的not in，同时也是$in的取反。如：
```
> db.test.find({"name":{"$nin":["stephen2","stephen1"]}})
{ "_id" : ObjectId("4fd58ecbb9ac507e96276f1a"), "name" : "stephen", "age" : 35,"genda" : "male", "email" : "stephen@hotmail.com" }
```

- $or等同于SQL中的or，$or所针对的条件被放到一个数组中，每个数组元素表示or的一个条件。
下面的示例等同于name = "stephen1" or age = 35
```
> db.test.find({"$or": [{"name":"stephen1"}, {"age":35}]})
{ "_id" : ObjectId("4fd58ecbb9ac507e96276f1a"), "name" : "stephen", "age" : 35,"genda" : "male", "email" : "stephen@hotmail.com" } 
```

- 下面的示例演示了如何混合使用$or和$in。
```
> db.test.find({"$or": [{"name":{"$in":["stephen","stephen1"]}}, {"age":36}]})
{ "_id" : ObjectId("4fd58ecbb9ac507e96276f1a"), "name" : "stephen", "age" : 35,"genda" : "male", "email" : "stephen@hotmail.com" } 
```

- $not表示取反，等同于SQL中的not。
```
> db.test.find({"name": {"$not": {"$in":["stephen2","stephen1"]}}})
{ "_id" : ObjectId("4fd58ecbb9ac507e96276f1a"), "name" : "stephen", "age" : 35,"genda" : "male", "email" : "stephen@hotmail.com" }

```

### 增
```
myser.save({'指数名称':item,'市盈率':pe,'盈利收益率':roe,'日期':date,'市净率':pb,'股息率':rate})
```

### 时间的用法
```
current = time.localtime(time.time())
year = current.tm_year
month = current.tm_mon
day = current.tm_mday

yearStr = '%i' % year
yearList = list(yearStr)
yearStr1 = yearList[-1]
yearStr2 = yearList[-2]
yearStr = yearStr2 + yearStr1

monthStr = months[month - 1]

targetStr = 'idx_%i%s%s' % (day,monthStr,yearStr)

```
