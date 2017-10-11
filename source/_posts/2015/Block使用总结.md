---
title: Block使用总结
date: 2015-05-2 19:17:41
tags:
- iOS基础知识
categories: iOS
---

block用来保存一段代码，在需要的时候回调，是对闭包的实现
<!--more-->

所以，切勿将过程代码当做block的实际实现，切记切记！！！
## block的声明
```
//变量
void (^myBlock)();
int (^myBlock1)(int, int);

//属性1
typydef void(^Myblock)();//这样Myblock就代表一个block类型了
@property (nonatomic, copy) Myblock myblock; 

//属性2
@property (nonatomic, copy) void(^myBlock)(int);//这样仍然是一个block，而且名字就是myBlock

//参数
- (void) modifyUserInfoWithNickName:(NSString *) nickName
                           userLogo:(NSString *) userLogo
                                sex:(NSNumber *) sex
                           location:(NSString *) location
                            succeed:(void(^)())successBlock
                               fail:(void(^)(NSError *error))errorBlock;
                               //这个时候的格式就变成了(返回+(^)(参数))名字
                               
也可以这样

typedef void(^FXSuccessBlock)(id result);
typedef void(^FXFailBlock)(NSError *error);

- (void)onlyGetLoginUserInfo:(FXSuccessBlock)successHandler
                 failHandler:(FXFailBlock)failHandler;
```
block声明的格式：`返回值 + block名字 + 参数`

## block实现
block实现的格式：`^(参数){}`

注意block实现是要加上变量名字的
```
self.loginBlock = ^(BOOL complete) {
  NSLog(@"登录成功");
};
```
或者有种偷巧的写法，就是利用set方法
```
 [self setLoginSucBlock:^(BOOL complete) {
      NSLog(@"登录成功");  
  }];
```

## Block对外部变量的管理
### 基本数据类型
#### 1、局部变量
局部自动变量，在Block中只读。Block定义时copy变量的值，在Block中作为常量使用，所以即使变量的值在Block外改变，也不影响他在Block中的值。
```
int base = 100;
long (^sum)(int, int) = ^ long (int a, int b) {
     return base + a + b;
};
base = 0;
PRintf("%ld\n",sum(1,2));
// 这里输出是103，而不是3, 因为块内base为拷贝的常量 100
}
```
//进入block的是值，而不是地址，因为这个变量是定义在栈内存中的，随时可能被释放，所以我要将它的值copy进来，防止我使用的时候取不到
#### 2、STATIC修饰符的全局变量
因为全局变量或静态变量在内存中的地址是固定的，Block在读取该变量值的时候是直接从其所在内存读出，获取到的是最新值，而不是在定义时copy的常量。

加入一个变量定义在函数之外，那么这个变量是全局变量。

```
static int a = 10;
    
int(^sum)(int) =^(int b) {
    return a + b;
};

a = 0;

NSLog(@"----%d", sum(2));//2
```
//进入block的是地址而不是值，因为static修饰的变量在内存中存在于静态变量区，是不会被回收或者释放的，即使释放了，也会在爱disk中有备份，当我们使用的时候再创建然后赋值给我就好了，所以不用担心被销毁，所以用地址就好了

#### 3.__block修饰的变量
被__block修饰的变量称作Block变量。 基本类型的Block变量等效于全局变量、或静态变量。

```
 __block int a = 10;
    
int(^sum)(int) =^(int b) {
    return a + b;
};

a = 0;

NSLog(@"----%d", sum(2));//2
//进入block的是地址而不是值，这个道理与static是一致的
```

注意：如果我们用NSString来验证对象的block的内存管理是没有意义的，因为，某几种情况下NSString对象是存放在常量区的，相当于int这种变量

`补充：`
```
NSString *name = @"han";
NSLog("%p", name);//打印的是@"han"的地址
NSLog("%p", &name);//打印的是name这个指针的地址
```
### 对象
对象一定是存在于堆上的，所以决定要不要拷贝指针的关键在于引用的指针
```
@implementation ViewController
UIView *view;
- (void)viewDidLoad {
    [super viewDidLoad];
    
    view = [[UIView alloc] init];

    void(^stackBlock)() = ^() {
        NSLog(@"&&&&&%p", &view);
    };
    
    NSLog(@"****%p", &view);
    
    stackBlock();
}
@end
//2016-07-11 20:47:36.908 block[1906:517460] ****0x1000d1208
//2016-07-11 20:47:36.909 block[1906:517460] &&&&&0x1000d1208

```
现在是一个全局变量来引用UIView，我们发现指针直接就是原来的地址，并没有发生变化，

```
@implementation ViewController

void(^globalBlock)();
int a;
//UIView *view;
- (void)viewDidLoad {
    [super viewDidLoad];
    
     UIView *view = [[UIView alloc] init];

    void(^stackBlock)() = ^() {
        NSLog(@"&&&&&%p", &view);
    };
    
    NSLog(@"****%p", &view);
    
    stackBlock();
}

//2016-07-11 21:00:51.055 block[1916:519453] ****0x16fd69f28
//2016-07-11 21:00:51.056 block[1916:519453] &&&&&0x13563a480
```
很明显是两个指针

那如果是一个成员变量会是什么效果呢？经过验证也是同一个指针。
一个指针和两个指针有啥区别呢？答案就是，如果是两个指针的话，那么即使你外边的指针释放掉，这个对象也是释放不掉的，如果这个block不释放的话，所以如果这个block是一个全局的block这个对象也就没有释放的可能了。
解决的方案就是，传入一个弱引用的指针，这样起码这个被引用的对象是不会因为block而不能释放了。
从这个角度来理解，为什么会存在堆，栈，全局block，因为如果我引用了某个变量，而这个变量是在栈上的，那么不能因为被block引用而不释放这个变量，所以解决方案就是直接copy一份这个变量的值进来block结构体，用一个指针去引用，而如果引用了一个全局的变量，那么就不存在释放不掉的问题了，那么就不用再复制一份了。

>一个block值创建的时候是在全局区的
- 如果这个时候给我引用了一个全局的变量，那么我仍然在全局
- 如果这个时候给我引用一个局部变量，那么这个block就会被移动到栈上，这里要注意的是成员变量与局部变量同等对待，因为它毕竟不在全局变量区，但是捕获成员变量的时候，因为这个变量是不会被轻易释放的，所以不会copy这个指针
- 这个时候如果你在ARC情况下对这个block做一个赋值操作，如果之前这个block在栈上，那么一个strong引用操作之后这个block就被移动到堆上了


//todo
autorelease关键字起作用在arc下，需要这个对象是一个autorelease对象，默认情况下，UIView不是这样的对象，其他的是这样的对象，所以
```
@autorelease {
  UIView *view = [UIView alloc] init];//内存会一直增加，不会释放的，以为UIView不会被释放，如果这个换成UIImage或者其他的，那就没有问题
}
```

