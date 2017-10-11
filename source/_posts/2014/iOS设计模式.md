---
title: iOS设计模式
date: 2014-11-18 15:20:59
tags:
- iOS基础知识
categories: iOS
---
>设计模式的功能是在软件设计当中是解决一些重复的公共问题。他们是一些模板来帮助你更容易的书写代码和复用你的代码。他们还可能帮助你创建低耦合的代码，你可以很轻松的修改和替换其中的组件。
<!--more-->
## MVC
现在可能更流行的是MVVM，首先，你要保证在你的项目中任何一个类都有一个控制器，一个模型，一个视图，一个类中的函数不能有两种作用

## Singleton – 单例模式
单例设计模式确切的说就是一个类只有一个实例，有一个全局的接口来访问这个实例。当第一次载入的时候，它通常使用延时加载的方法创建单一实例。

在一些情况下，一个类只有一个实例是有意义的。例如，这里没有必要有多个登录实例，除非你一次想写入多个日志文件。或者，一个全局的配置类文件：它可以很容易的很安全的执行一个公共资源，这样的一个配置文件，要比同时修改多个配置类文件好很多。

推荐写法，拦截alloc和copy方法

```
static  MySingleton  *shareSingleton ＝ nil;

+( instancetype ) sharedSingleton  {

   static  dispatch_once  onceToken;

   dispatch_once ( &onceToken, ^ {

   shareSingleton  =  [[super allocWithZone:NULL] init] ;

} );

    return sharedSingleton;

}

+(id) allocWithZone:(struct _NSZone *)zone {

   return [Singleton shareInstance] ; 

}

-(id) copyWithZone:(struct _NSZone *)zone {

   return [Singleton shareInstance] ;

}

```

## Observer – 观察者模式
在观察者模式中，当状态发生改变的时候，一个对象会通知另一个对象。这个对象不需要知道另一个对象发生了什么改变─因此非常鼓励这种分离式的设计。这种模式经常用于，当一个属性发生改变时通知跟它相关的对象。

它通常需要一个观察者(observer)注册跟踪另外一个对象的状态。当状态发生改变的时候，所有的观察对象都会被通知改变。苹果的推送通知服务就是一个这样的例子。

如果你想要一直使用 MVC 模式（你确实需要），你如果想在模型和视图之间，不直接相互引用的情况下还要有通信。这时候就要用到观察者模式了。

Cocoa 有两个常用的方法来执行观察者模式：Notifications 和 Key-Value Observing (KVO)。

### 通知 Notifications
#### 实现原理


#### 注意
- 通知是同步操作，所以再通知的调用方法中做太多的事情是肯定会有问题的，有太多的通知也是有问题的


### KVO

#### KVO实现原理
![](http://7xrn7f.com1.z0.glb.clouddn.com/16-7-18/9770095.jpg)
- KVO是基于runtime机制实现的
- 当某个类的属性对象第一次被观察时，系统就会在运行期动态地创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的setter 方法。派生类在被重写的setter方法内实现真正的通知机制
- 如果原类为Person，那么生成的派生类名为NSKVONotifying_Person
- 每个类对象中都有一个isa指针指向当前类，当一个类对象的第一次被观察，那么系统会偷偷将isa指针指向动态生成的派生类，从而在给被监控属性赋值时执行的是派生类的setter方法
- 键值观察通知依赖于NSObject 的两个方法: willChangeValueForKey: 和 didChangevlueForKey:；在一个被观察属性发生改变之前， willChangeValueForKey: 一定会被调用，这就 会记录旧的值。而当改变发生后，didChangeValueForKey: 会被调用，继而 observeValueForKey:ofObject:change:context: 也会被调用。
- 补充：KVO的这套实现机制中苹果还偷偷重写了class方法，让我们误认为还是使用的当前类，从而达到隐藏生成的派生类
- KVO和通知一样，都是同步操作，所以使用的时候要注意




#### 手动触发KVO
```
- (void)viewDidLoad
{
    [super viewDidLoad];

    // “手动触发self.now的KVO”，必写。
    [self willChangeValueForKey:@"now"];

    // “手动触发self.now的KVO”，必写。
    [self didChangeValueForKey:@"now"];
}
```
