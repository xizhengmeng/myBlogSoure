---
title: iOS中的定时和延时
date: 2015-01-23 17:04:55
tags:
- iOS基础知识
categories: iOS
---
我们使用延时和定时可以使用以下方法：
- NSTimer


## NSTimer
### 循环引用
我们使用一个nstimer的时候，使用一个类方法来创建一个timer，返回一个timer指针，有时需要把tiemr加到runloop，有时系统自己来帮你做，你会发现，即使我并没有去强引用这个指针，timer只要在runloop中，timer就可以自然的回调，这是因为timer被runloop所引用，我们用控制器去引用这个timer，也只是增加了一个引用而已。

```
+(id)scheduledTimerWithTimeInterval:(NSTimeInterval)inTimeInterval block:(void (^)())inBlock repeats:(BOOL)inRepeats
{
    void (^block)() = [inBlock copy];
    id ret = [self scheduledTimerWithTimeInterval:inTimeInterval target:self selector:@selector(jdExecuteSimpleBlock:) userInfo:block repeats:inRepeats];
    [block release];
    return ret;
}

+(id)timerWithTimeInterval:(NSTimeInterval)inTimeInterval block:(void (^)())inBlock repeats:(BOOL)inRepeats
{
    void (^block)() = [inBlock copy];
    id ret = [self timerWithTimeInterval:inTimeInterval target:self selector:@selector(jdExecuteSimpleBlock:) userInfo:block repeats:inRepeats];
    [block release];
    return ret;
}

+(void)jdExecuteSimpleBlock:(NSTimer *)inTimer;
{
    if([inTimer userInfo])
    {
        void (^block)() = (void (^)())[inTimer userInfo];
        block();
    }
}
```
这也是Block实现的tiemr不会被释放的原因。

