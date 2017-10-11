---
title: iOS知识小集合
date: 2016-11-15 14:34:32
tags:
- IOS
---

- 1.dealloc方法是不能被覆盖的的，我们在子类中写的dealloc会被调用，同时父类的dealloc也会被调用，无需在子类中写上[super dealloc]

### 获取实际的启动图
```
- (NSString *)getLaunchImageName
{
CGSize viewSize = self.window.bounds.size;
// 竖屏
NSString *viewOrientation = @"Portrait";
NSString *launchImageName = nil;
NSArray* imagesDict = [[[NSBundle mainBundle] infoDictionary] valueForKey:@"UILaunchImages"];
for (NSDictionary* dict in imagesDict)
{
CGSize imageSize = CGSizeFromString(dict[@"UILaunchImageSize"]);
if (CGSizeEqualToSize(imageSize, viewSize) && [viewOrientation isEqualToString:dict[@"UILaunchImageOrientation"]])
{
launchImageName = dict[@"UILaunchImageName"];
}
}
return launchImageName;
}

```

### 手动更改状态栏颜色
```
- (void)setStatusBarBackgroundColor:(UIColor *)color
{
UIView *statusBar = [[[UIApplication sharedApplication] valueForKey:@"statusBarWindow"] valueForKey:@"statusBar"];
if ([statusBar respondsToSelector:@selector(setBackgroundColor:)])
{
statusBar.backgroundColor = color;
}
}
```

### 禁止锁屏
默认情况下，当设备一段时间没有触控动作时，iOS会锁住屏幕。但有一些应用是不需要锁屏的，比如视频播放器。

[UIApplication sharedApplication].idleTimerDisabled = YES;
或
[[UIApplication sharedApplication] setIdleTimerDisabled:YES];

