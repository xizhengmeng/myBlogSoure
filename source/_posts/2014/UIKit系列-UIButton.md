---
title: UIKit系列---UIButton
date: 2014-12-3 20:03:53
tags:
- iOS基础知识
categories: iOS
---

### 1.一个界面内有多个button要防止同时点击
每个button都设置该属性
```
[btn setExclusiveTouch:YES];
```
<!--more-->
### 2.button不同状态下图片切换失效
如果你设置了normal状态下的button的图片，然后你设置disable状态下图片为nil，你会发现当你切换状态的时候，图片并不会变成nil，你许雅做的事，做一张像素为空的图片，然后在disable状态下，把按钮的图片设置为这个像素为空的图片

### 3.iOS7的UIButton第二次调用settiltle方法失效
这种情况下需要自定义button，替换button本身的方法，典型的应用场景是，按钮显示倒计时，或者切换关注和未关注状态
```
#define iOS7  ( [[[UIDevice currentDevice] systemVersion] compare:@"7.0"] != NSOrderedAscending )

@(11-经典bug)interface SongShowReleaseButton()
@property (nonatomic, strong) UILabel *theNewTitleLable;
@end

@implementation SongShowReleaseButton
/**theNewTiltleLable*/
- (UILabel *)theNewTitleLable {
    if (_theNewTitleLable == nil) {
        _theNewTitleLable = [[UILabel alloc] init];
        _theNewTitleLable.font = [UIFont boldSystemFontOfSize:16];
    }
    return _theNewTitleLable;
}

- (void)setTitle:(NSString *)title forState:(UIControlState)state {
    DLog(@"----%@", title);
    if (iOS7) {
        self.theNewTitleLable.text = title;
        [self.theNewTitleLable sizeToFit];
        self.theNewTitleLable.center = CGPointMake(self.width / 2, self.height / 2);
        [self addSubview:self.theNewTitleLable];
    }else {
        [super setTitle:title forState:state];
    }
}

- (void)setTitleColor:(UIColor *)color forState:(UIControlState)state {
    if (iOS7) {
        self.theNewTitleLable.textColor = color;
    }else {
        [super setTitleColor:color forState:state];
    }
}
```

### 4.target的方法被多次调用
检查是否添加target的方法被执行了多次

### 5.button实现双击调用
```
  [button addTarget:self action:@selector(onTouchUpInside:withEvent:) forControlEvents:UIControlEventTouchUpInside];

   -(void)onTouchUpInside:(id)sender withEvent:(UIEvent*)event 
{
    UITouch* touch = [[event allTouches] anyObject];
    NSLog(@"onTouchUpInside tagCount:%d",touch.tapCount);

    //判断点击次数
    if (touch.tapCount == 1) 
    {

       //todo
    }
}
```

### 5.button长按事件
```
UIButton *aBtn=[UIButton buttonWithType:UIButtonTypeCustom];  
    [aBtn setFrame:CGRectMake(40, 100, 60, 60)];  
    [aBtn setBackgroundImage:[UIImage imageNamed:@"111.png"]forState:UIControlStateNormal];  
    //button点击事件  
    [aBtn addTarget:self action:@selector(btnShort:)forControlEvents:UIControlEventTouchUpInside];  
    //button长按事件  
    UILongPressGestureRecognizer *longPress = [[UILongPressGestureRecognizer alloc] initWithTarget:selfaction:@selector(btnLong:)];   
    longPress.minimumPressDuration = 0.8; //定义按的时间  
    [aBtn addGestureRecognizer:longPress];  
  
-(void)btnLong:(UILongPressGestureRecognizer*)gestureRecognizer{  
    if ([gestureRecognizer state] == UIGestureRecognizerStateBegan) {  
        NSLog(@"长按事件");  
        UIAlertView *alert=[[UIAlertView alloc]initWithTitle:@"消息" message:@"确定删除该模式吗？" delegate:selfcancelButtonTitle:@"取消" otherButtonTitles:@"删除", nil nil];  
        [alert show];  
    }  
}  
```

### 6.button在被点击完之后如果是耗时操作可以禁止点击，防止多次触发，等事件返回之后再放开按钮点击事件


### 7.在UIButton中addSubview的问题

>UIView的userInteractionEnabled值默认为YES，必须设置UIButton所有的subview的userInteractionEnabled为NO，才能让UIButton正常响应点击。
但是如果设置了UIView的setUserInteractionEnabled为NO，其子view都将得不到响应。

### 8.扩大按钮点击范围
```
import <UIKit/UIKit.h>

@interface UIButton (EnlargeTouchArea)

- (void)setEnlargeEdgeWithTop:(CGFloat) top right:(CGFloat) right bottom:(CGFloat) bottom left:(CGFloat) left;

@end



#import "UIButton+EnlargeTouchArea.h"
#import <objc/runtime.h>
@implementation UIButton (EnlargeTouchArea)


static char topNameKey;
static char rightNameKey;
static char bottomNameKey;
static char leftNameKey;

- (void) setEnlargeEdgeWithTop:(CGFloat) top right:(CGFloat) right bottom:(CGFloat) bottom left:(CGFloat) left
{
    objc_setAssociatedObject(self, &topNameKey, [NSNumber numberWithFloat:top], OBJC_ASSOCIATION_COPY_NONATOMIC);
    objc_setAssociatedObject(self, &rightNameKey, [NSNumber numberWithFloat:right], OBJC_ASSOCIATION_COPY_NONATOMIC);
    objc_setAssociatedObject(self, &bottomNameKey, [NSNumber numberWithFloat:bottom], OBJC_ASSOCIATION_COPY_NONATOMIC);
    objc_setAssociatedObject(self, &leftNameKey, [NSNumber numberWithFloat:left], OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (CGRect) enlargedRect
{
    NSNumber* topEdge = objc_getAssociatedObject(self, &topNameKey);
    NSNumber* rightEdge = objc_getAssociatedObject(self, &rightNameKey);
    NSNumber* bottomEdge = objc_getAssociatedObject(self, &bottomNameKey);
    NSNumber* leftEdge = objc_getAssociatedObject(self, &leftNameKey);
    if (topEdge && rightEdge && bottomEdge && leftEdge)
    {
        return CGRectMake(self.bounds.origin.x - leftEdge.floatValue,
                          self.bounds.origin.y - topEdge.floatValue,
                          self.bounds.size.width + leftEdge.floatValue + rightEdge.floatValue,
                          self.bounds.size.height + topEdge.floatValue + bottomEdge.floatValue);
    }
    else
    {
        return self.bounds;
    }
}

- (UIView*) hitTest:(CGPoint) point withEvent:(UIEvent*) event
{
    CGRect rect = [self enlargedRect];
    if (CGRectEqualToRect(rect, self.bounds))
    {
        return [super hitTest:point withEvent:event];
    }
    return CGRectContainsPoint(rect, point) ? self : nil;
} 
```
