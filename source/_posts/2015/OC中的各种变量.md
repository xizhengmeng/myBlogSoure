---
title: OC中的各种变量
date: 2015-06-10 18:00:10
tags:
- iOS基础知识
categories: iOS
---

首先看一下几个概念分类：
- 成员变量
- 属性
- 局部变量
- 全局变量
- 静态变量 = 全局变量 + static声明的局部变量
<!--more-->

>首先声明：
- 全局变量和静态变量都存在于全局内存区，在app生命周期内只初始化一次
- 局部变量存在于栈内存区
- 这里所讨论的是变量，而不是变量的值，这里两个概念`int a = 10`，a是变量，而10是它的值，这是两个概念
- 地位内存是全局变量区，高位内存是堆或者栈`0x108eae338`9位， `0x7fff56d52ad4`12位

## 成员变量var
>定义：变量存活周期跟你定义的该类实体对象一样；作用域是整个实体对象；可以在h文件中声明或者在m文件中@implementation上面添加的

```
@interface CustomView1()
{
	UIView *_backView1;
}
@property (nonatomic, strong) UIView *backView;
@end
```
上边的两种写法都是对的，都代表是一个类的成员变量，只不过@property关键字做的事情比较多，上边类名称+()叫做extension，类扩展，如果有括号内加上名字叫做category，是分类。成员变量声明周期与类的对象相同。
我们可以通过将一个对象赋值给成员变量来提升他的生命周期。

## 属性
@property关键字标识的变量

## 全局变量
>全局变量指的是存在于全局内存区，这个app声明周期只初始化一次，在所有的文件中不允许重名，下面的情况均属于全局变量
.m文件中
```
int a;

@interface CustomView1()

@end
int b;
@implementation CustomView1
int c;
CustomView *_backView;

- (instancetype)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
        [self initSubViews];
    }
    return self;
}

@end

```
>说明：a，b，c均为全局变量

.h文件中
```
int d;
@interface CustomView1

@end
```
>说明：主要不是在interface关键字与end关键字之间都是全局变量，如果其他文件中有这个名字的变量，那么编译时不能够通过的

全局变量可以提供全部的外界访问，无论你定义在h文件或者是m文件中，并且无需引入头文件，但是我们需要再重新再定义一下这个变量，否则编译器是不会通过的，他怎么知道你有这个变量呢，你说是吧，所以我们可以像下边这样来使用

```
#import "ViewController.h"
#import "CustomView1.h"

typedef void(^MyBlcok)();
NSString * const waha = @"lalala";

@interface ViewController ()

@property (nonatomic, copy) MyBlcok myBlock;
@property (nonatomic, strong) UIView *backView;

@end


#import "AppDelegate.h"

@interface AppDelegate ()

@end

@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    extern NSString *waha;

    NSLog(@"appdelegate***%@---%p", waha, &waha);
    return YES;
}
```
>我们需要再重新定义一下这个变量，否则编译器是通不过的，不过要将这变量变为全局才可以，其实也就是改变一下它的内存位置，将其从栈移动到全局内存中去

### static + 全局变量
这种方式的话这个全局变量就会变成这个类所私有的，声明周期与该类相同，允许重名


### iOS中使用全局变量的方法
- 静态变量
- 单例
- 在某个现成的单例中增加属性例如:appdelegate

## 局部变量
除了全局变量剩下的都是局部变量，内存是在栈上的，如果用关键字static来修饰局部变量那么它就变成了一个静态变量，内存也到了全局变量区，只是生命周期发生了改变，但是作用域并没有发生变化。

- 如果用extern来修饰局部变量，那么它就会直接内存区变为全局区，生命周期和app一样了。如果后边再有其他的同名全局变量，编译器不会报错，但是后边生命的那个不会生效的，不过值还是允许改的。可以打印来验证一下，无论你在后边声明多少个该同名变量，他们的地址总是一样的。

## 常量与const关键字
顾名思义，常量就是不能被修改的变量，说到这里我就有一个疑问，这个跟宏有啥区别？
- const简介：之前常用的字符串常量，一般是抽成宏，但是苹果不推荐我们抽成宏，推荐我们使用const常量。
- 编译时刻:宏是预编译（编译之前处理），const是编译阶段。
- 编译检查:宏不做检查，不会报编译错误，只是替换，const会编译检查，会报编译错误。
- 宏的好处:宏能定义一些函数，方法。 const不能。
- 宏的坏处:使用大量宏，容易造成编译时间久，每次都需要重新替换。
- 注意:很多Blog都说使用宏，会消耗很多内存，我这验证并不会生成很多内存，宏定义的是常量，常量都放在常量区，只会生成一份内存

我们确实可以选择使用宏来定义一个常量，然后给很多地方使用，当我需要改变这个变量的时候，我只要改变这个宏后边的值就可以了，使用的场景比如：
- 项目中的url
- 项目中的屏幕尺寸
- 项目中的某些特定颜色和尺寸

有些地方宏会体现出它的优势，比如一些代码块的替换，具体来说，比如屏幕的尺寸的获取，比如weakSelf的定义，但是使用常量的情况使用宏是有些问题的，就是因为宏是没有类型判断的，那么如果某个CGFloat的值，我们使用了宏，那么编译阶段就不会警告任何错误，哪怕你定义了一个NSString类型的宏，然后你又把它当做了CGFloat来使用，所以对于某些值得定义，我们要使用类型常量，而不用宏来定义，比如：
- 动画时间
- 按钮尺寸
- 文字尺寸
- 等等

具体来讲又可能分为三种情况：
- 全部文件都使用
- 只给自己使用

我们先来看看给自己使用的常量的定义：

```
#import "key1.h"

static const NSTimeInterval kAnimationDuration = 0.2;

@interface key1()

@end

@implementation key1

@end
```
要点有两个：
- 要用static和const同时来声明
- 声明在关键字之外

声明在关键字之外那么就是全局变量，又因为是给自己用的，所以要加上static做限制，而const是为了防止这个变量在后边被修改

再来看一下全局使用的常量，此类常量需要放在`全局符号表`,下面我们来看一下给全局使用的常量是如何来定义的：
```
//In the header file
extern NSString *const EOCStringConst;//这样的const的位置意味着这个指针是一个常量

//In the implementation file
NSString *const EOCStringConstant = @"VALUE";
```
>这种常量一般在头文件中声明，然后在实现文件中定义，注意const的位置，不允许更改指针的指向，另外编译器看到extern关键字，就明白在全局符号表中将会有一个名字叫EOCStringConstant的符号了，

## static和extern
>static作用：
- 修饰局部变量
  - 延长局部变量的生命周期
  - 局部变量只会生成一份内存，初始化一次
  - 改变局部变量的作用域
- 修饰全局变量
  - 只能在本文件访问，修改全局变量的作用域，生命周期不变
  - 避免重复定义全局变量

>extern
- 作用：获取全局变量的值，不能用来定义变量
- 工作原理：现在当前文件查找有木有全局变量，没有的话才去其他文件查找


## static和const
static与const作用:声明一个只读的静态变量

## extern与const联合使用
- 开发中使用场景:在多个文件中经常使用的同一个字符串常量，可以使用extern与const组合。

    - 原因:static与const组合：在每个文件都需要定义一份静态全局变量。
    - extern与const组合:只需要定义一份全局变量，多个文件共享。

- 全局常量正规写法:开发中便于管理所有的全局变量，通常搞一个GlobeConst文件，里面专门定义全局变量，统一管理，要不然项目文件多不好找。

```
GlobeConst.h

/*******************************首页****************************/
extern NSString * const nameKey = @"name";
/*******************************首页****************************/

GlobeConst.m

#import <Foundation/Foundation.h>
/*******************************首页****************************/
NSString * const nameKey = @"name";
/*******************************首页****************************/
```
我们现在来理解一下上面的代码，其实应该反过来看，首先我们在.m文件中声明并且对nameKey进行了赋值，所以我们在.h文件中才可以通过extern来取到nameKey的值，并且使用，其实我们完全可以在其他地方进行nameKey的取值，比如另外一个不相关的.m文件，只不过这样做会比较繁琐，需要重新声明这个变量，所以我们统一将这些个静态变量以extern的方式声明出来，放在.h文件中，让其他的文件调用，注意，这里是不能进行赋值操作的
>这里做一个辨析：全局变量并不是因为你用extern关键字，全局变量是由你变量声明的位置所决定的，而extern其实是一个取值的操作，同时告诉编译器这个变量无论声明在哪里，我知道这个全局变量是存在的，放心的使用就好，但是如果真的这个全局变量是不存在的，比如说你把它声明定义在了某个函数内部，那么这个时候肯定报错了
>另外我们使用`extern NSString *name = @"名字"`这种写法是没有意义的，因为extern是取值操作，这样写编译器也会报错的

关于extern关键字总结如下：
- 1. Declaration can be done any number of times but definition only once.
- 2. “extern” keyword is used to extend the visibility of variables/functions().
- 3. Since functions are visible through out the program by default. The use of extern is not needed in function declaration/definition. Its use is redundant.
- 4. When extern is used with a variable, it’s only declared not defined.
- 5. As an exception, when an extern variable is declared with initialization, it is taken as definition of the variable as well.

## __weak 与 __strong 
当我们定义一个变量的时候
```
UIView *view;
view = [UIView alloc] init];
```
其实这个view前边默认有一个关键字，那就是__strong，这是因为这个关键字，所以我们的=赋值的时候，是一个强引用，所以alloc创建出来的对象是不会被释放的，又因为view这个变量被定义在栈上，所以当代码执行到一个函数结束的时候，这个view变量会被释放，这样alloc出来的对象就失去了强引用，这部分内存也将被系统所回收。

现在我们来理解一下block的赋值操作，我们知道，当一个block被创建的时候，假如没有捕获变量，那么这个block是定义在全局的，对全局block做一个强引用的赋值操作是不会改变它的存储位置的，如果这个block捕获了一个栈上的变量，那么这个block就会被移动到栈上，这个时候如果我们对这个block做了一个强引用的赋值操作，这个block就被移动到了堆上。内存管理策略就变成了是否有强指针引用

## 使用常量替换宏定义
使用宏定义过多的话，随着工程越来越大，编译速度会越来越慢。
```
static CGFloat const kLogoImageWidth = 100; //logo宽度
static CGFloat const kLogoImageHeight = 100; //logo宽度static CGFloat const kLogoImageY = 110;
static CGFloat const kBtnHeight = 40;
static CGFloat const kPadding = 30;
static CGFloat const kWeixinTopPadding = 15;
static CGFloat const kWeiboLoginBottom = 230;
#define kScaleSpace(designSpace) ((designSpace)(SCREEN_HEIGHT/667.0)) //根据iphone6 的设计稿计算缩放高度
```

替换完成之后代码的编译速度确实上去了，现在编译快了。希望对正在为编译速度慢感到困惑的您有所帮助

>补充说明：以上的类型常量替换宏的情况，只是适用于单个文件的情况。如果是多个文件共享的常量，苹果推荐的这样的方式：

```
UserInfoModelConstants.h

extern NSString *const USER_AGE_KEY ;
extern NSString *const USER_TELPHONE_KEY ;
extern NSString *const USER_ADDRESS_KEY ;
extern NSString *const USER_BRIEF_KEY ;

UserInfoModelConstants.m

NSString *const BKUSER_AGE_KEY = @"XXXXX.userAge";
NSString *const BKUSER_TELPHONE_KEY = @"XXXXX.telphoneNO";
NSString *const BKUSER_ADDRESS_KEY = @"XXXXX.address";
NSString *const BKUSER_BRIEF_KEY = @"XXXXX.brief";
```
在需要使用共享常量的文件中引入UserInfoModelConstants.h即可。


