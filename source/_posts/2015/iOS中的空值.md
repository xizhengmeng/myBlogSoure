---
title: iOS中的空值
date: 2015-02-15 15:02:55
tags:
- iOS基础知识
categories: iOS
---
nil和NULL和Nil和NSNull的区别
<!--more-->
## nil
- nil 是 ObjC 对象的字面空值，对应 id 类型的对象，或者使用 @interface 声明的 ObjC 对象。
- 例如：
```
NSString *someString = nil;
NSURL *someURL = nil;
id someObject = nil;


if (anotherObject == nil) // do something
- 定义：
?
// objc.h
#ifndef nil
# if __has_feature(cxx_nullptr)
#  define nil nullptr
# else
#  define nil __DARWIN_NULL
# endif
#endif


// __DARWIN_NULL in _types.h


#define __DARWIN_NULL ((void *)0)
```

## Nil

- Nil 是 ObjC 类类型的书面空值，对应 Class 类型对象。
- 例如：

```
Class someClass = Nil;
Class anotherClass = [NSString class];
```

- 定义声明和 nil 是差不多的，值相同：
```
// objc.h
#ifndef Nil
# if __has_feature(cxx_nullptr)
#  define Nil nullptr
# else
#  define Nil __DARWIN_NULL
# endif
#endif
```
## NULL

- NULL 是任意的 C 指针空值。
- 例如：
```
int *pointerToInt = NULL;

- char *pointerToChar = NULL;
- struct TreeNode *rootNode = NULL;
- 定义：
?
// in stddef.h


#define NULL ((void*)0)
```

## NSNull
- NSNull 是一个代表空值的类，是一个 ObjC 对象。实际上它只有一个单例方法：+[NSNull null]，一般用于表示集合中值为空的对象。
- 注意，这里NSNull是一个单例，也就是说同一个数组中，如果我们要删除这个空对象，那么只要调用一次removeobjec:方法就好了。
- 例子说明：
```
// 因为 nil 被用来用为集合结束的标志，所以 nil 不能存储在 Foundation 集合里。
NSArray *array = [NSArray arrayWithObjects:@"one", @"two", nil];

-

// 错误的使用
NSMutableDictionary *dict = [NSMutableDictionary dictionary];
[dict setObject:nil forKey:@"someKey"];
-

// 正确的使用
NSMutableDictionary *dict = [NSMutableDictionary dictionary];
[dict setObject:[NSNull null] forKey:@"someKey"];
```

，对于像NSArray这样的类型，nil或NULL不能做为加到其中的Object，如果定义了一个NSArray，为其分配了内存，又想设置其中的内容为空，则可以用[NSNULL null返回的对对象来初始化NSArray中的内容，我的感觉有点像C语言中malloc一个内存空间，然后用memset初始化这段空间里的值为0。

由于服务器的数据库中有些字段为空，然后以Json形式返回给客户端时就会出现这样的数据：
"somevalue":null ，通过JsonKit 这个第三方库解析出来的数据就成了
somevalue=""; 这个数据类型不是nil 也不是 String。 解析成对象之后，如果直接向这个对象发送消息（eg：length，count 等等）就会直接崩溃。提示错误为：
-[NSNulllength]:unrecognizedselectorsenttoinstance0x388a4a70 ，这个就是NSNull类型的对象，代表空对象。
## NIL 或 NSNil

ObjC 不存在这两个符号！

小结

虽然 nil, Nil, NULL 的值相同，理解它们之间的书面意义才重要，让代码更加明确，增加可读性。
