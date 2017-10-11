---
title: iOS中的屏幕旋转
date: 2016-02-12 22:14:43
tags:
- iOS基础知识
categories: iOS
---

- 旋转事件只传递给主window
- 当我们使用webView播放视频的时候，它会创建一个UIViewController，然后创建一个window，让这个控制器成为这个window的根控制器，然后再将视频显示的view添加到这个UIViewController的控制器view上
- 屏幕选中的本质是当前主控制器的view跟随屏幕的旋转而旋转，并且调整大小至当前的宽高，所以我们要做的就是控制当前控制器要不要旋转
- 再本质一些就是，要对当前keywindown的根控制器进行控制＋对你要旋转的界面进行控制


### 详述
#### 第一层设置
![Alt text](./1444701872103.png)
在这里选择是否支持横竖屏
#### 第二层设置
```
- (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window {  
    return UIInterfaceOrientationMaskAll;
}
```
在Appdelegate中进行设置，实现这个方法，在这里返回你要支持的方向，这个设置会覆盖上边在general中的设置

#### 第三层设置
在window的`根控制器`中实现这两个方法，来加以控制，是否允许旋转以及支持的方向
```
//是否允许旋转
- (BOOL)shouldAutorotate {
    return YES;
}
//旋转的时候支持的方向
- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    
    return UIInterfaceOrientationMaskAll;
}
```

#### 第四层设置
在你要控制是否旋转的这个控制里边实现上边的这两个方法


#### 小结
>- 当屏幕旋转一次就会调用一次这个方法，来询问支持的方向
>- 这里返回的window是Appdelegate这个对象持有的window，也就是说，即使中间你换掉了keywindow这里返回的仍然只是最开始创建的那个window，也就是说，对你最后创建的window这个方法并不能影响
>- 一个控制器到底是否支持横屏，自己说了是不算的，主要看两个东西
>  - AppDelegate中的这个方法返回支持的方向
>  - window的根控制器中是否允许旋转，以及支持的方向
>  - **取交集**




上边的的四层设置起作用的情况，我们要分三种情况来说
>- 要控制的控制器间接依附最开始创建的window
>	- 该控制器是通过navigationcontroller管理的/直接添加childcontrollers
>	- present...
>- 要控制的控制器依附于新创建的window

##### 原始window
>- 受到Appdelegate中的方法的影响
>- 如果控制器是modal出来的，那么该控制器中实现的这两个方法会生效，但是依然遵循交集的原则
>- 如果这个控制器是navigationController管理或者直接添加childcontrollers，该控制器中的方法是没有作用的，它只受到appdelegate和window根控制器的影响

##### 依附于新创建的window
>- 不受到APPdelegate方法的影响
>- 其它都一样，但是分析的起点就变成了当前window的根控制器
	- 比如，webView播放视频的时候，它会新创建window，并且以UIViewController为根控制器，所以我们应该在这个控制器中添加控制代码，但是这个是系统的类，我们就只能靠分类来添加代码了

### demo
#### 整体是支持横屏，但是某个界面禁止
根据上边叙述，我们可以得到以下方案：
- 通过present...来管理控制器

> 在这个控制器中添加控制器代码，禁止旋转或者只支持竖着
```
  @implementation HSViewController

- (void)viewDidLoad {
   [super viewDidLoad];
}

- (BOOL)shouldAutorotate {
    return YES;
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskPortrait;
}

@end
```
- 不通过present来管理界面
> 通过在Appdelegate或者根控制器中加判断代码，实现在不同的当前控制器，返回不同的支持的方向
```
 - (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window {
    

   HSNavigationContronller *nav = (HSNavigationContronller *)[UIApplication sharedApplication].keyWindow.rootViewController;
    
    if ([nav.topViewController isKindOfClass:[HSViewController class]]) {
        return UIInterfaceOrientationMaskPortrait;
    }
    
    return UIInterfaceOrientationMaskAll;
    
}
```

#### 整体是支持竖屏，但是某个界面可以横屏
这种需求，只能通过加判断代码来做到

#### 控制通过webView加载的视频全屏时的横竖屏
webView的视频在全屏的时候，会新创建一个window，所以你在Appdelegate或者当前根控制器下的控制代码统统不会生效，并且以UIViewController为根控制器，所以我们要把控制代码加到UIViewController的分类中，又因为其它的控制器全部继承自这个控制器，所以我们要在这个里边加判断
```
#import "UIViewController+Extension.h"

@implementation UIViewController (Extension)

#pragma mark - 下面的两个方法主要用于控制webView播放视屏的时候，该是否是否应该横屏
/**
 * 如果当前keywindow的rootController类型为UIViewController，就返回YES，否则返回NO
 *
 *  @return 返回是否
 */
- (BOOL)shouldAutorotate {
    //拿到当前主窗口的根控制器类型名称
    NSString *className = NSStringFromClass([[UIApplication sharedApplication].keyWindow.rootViewController class]);
    //如果是UIViewController就返回YES，让这个控制器view可以跟随屏幕旋转
    if ([className isEqualToString:@"UIViewController"]) {
        return YES;
    } else {
        return NO;
    }
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    
    return UIInterfaceOrientationMaskAll;
}
@end

```
