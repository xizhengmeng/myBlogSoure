---
title: Runloop概述
date: 2015-07-21 09:35:22
tags:
- iOS进阶
categories: iOS
---

## 1.基本概念
### 为什么需要runloop
- 使程序一直活着
- 决定程序何时应该处理哪些event
- 调用解耦(message queue)
- 节省CPU时间
- 开启主线程要消耗1M内存，开启一个后台线程需要消耗512k内存，我们应当在线程没有任务的时候休眠，来释放所占用的资源

### 与runloop相关的类
- NSTimer
- UIEvent
- Autorelease
- NSObjec-->NSDelayedPerforming
- NSObjec-->NSThreadPerformAddition
- CADisplayLink
- CATransition
- CAAnimation
- dispatch_get_main_queue()
- NSURLConnection

### runloop与线程

- 每条线程都有唯一的一个与之对应的RunLoop对象
- 主线程的RunLoop已经自动创建好了，子线程的RunLoop需要主动创建
- RunLoop在第一次`获取时创建`，在线程结束时销毁
- 一般情况下我们是没有必要去启用线程的RunLoop的，除非你在一个单独的线程中需要长久的检测某个事件，比如说主线程。
- 线程和RunLoop之间是以键值对的形式一一对应的，其中key是thread，value是runLoop(这点可以从苹果公开的源码中看出来)，其实RunLoop是管理线程的一种机制，这种机制不仅在IOS上有，在Node.js中的EventLoop，Android中的Looper，都有类似的模式

### 获取runloop
>Foundation
```
[NSRunLoop currentRunLoop]; // 获得当前线程的RunLoop对象
[NSRunLoop mainRunLoop]; // 获得主线程的RunLoop对象
```

>Core Foundation
```
CFRunLoopGetCurrent(); // 获得当前线程的RunLoop对象
CFRunLoopGetMain(); // 获得主线程的RunLoop对象
```

### runloop基本结构
`NSRunLoop继承自NSObject`
![](http://i1.piimg.com/567571/d478dc563d0af5b8.png)
- runloop与Thread一一对应，但是并不是一个thread只能有一runloop，我们可以给runloop嵌套runloop

#### CFRunLoopMode
- CFRunLoopModeRef代表RunLoop的运行模式
    - 一个 RunLoop 包含若干个 Mode，每个Mode又包含若干个Source/Timer/Observer
    - 每次RunLoop启动时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode
    - 如果需要切换Mode，只能退出Loop，再重新指定一个Mode进入
    - 这样做主要是为了分隔开不同组的Source/Timer/Observer，让其互不影响

![](http://i1.piimg.com/567571/83261feb726f1627.png)

>系统默认注册了5个Mode:(前两个跟最后一个常用)
- kCFRunLoopDefaultMode：App的默认Mode，通常主线程是在这个Mode下运行
- UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
- UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
- GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
- kCFRunLoopCommonModes: 这是一个占位用的Mode，不是一种真正的Mod
![](http://i1.piimg.com/567571/8d1aff60c4248ca3.png)
#### CFRunLoopSource
- CFRunLoopSourceRef是事件源（输入源）
- 按照官方文档的分类
    - Port-Based Sources (基于端口,跟其他线程交互,通过内核发布的消息)
    - Custom Input Sources (自定义)
    - Cocoa Perform Selector Sources (performSelector…方法)
- 按照函数调用栈的分类
    - Source0：非基于Port的
    - Source1：基于Port的

Source0: event事件，只含有回调，需要先调用CFRunLoopSourceSignal(source)，将这个 Source 标记为待处理，然后手动调用 CFRunLoopWakeUp(runloop) 来唤醒 RunLoop。
Source1: 包含了一个 mach_port 和一个回调，被用于通过内核和其他线程相互发送消息,能主动唤醒 RunLoop 的线程。

#### CFRunLoopTimer
本质就是一个timer
```
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay;//该方法定义在RunLoop中，不过是NSObject的分类；
+ (CADisplayLink *)displayLinkWithTarget:(id)target selector:(SEL)sel;//这个类方法定义在DisPlayLink中
- (void)addToRunLoop:(NSRunLoop *)runloop forMode:(NSString *)mode;//这也是DisPlayLink中的方法
```
- CFRunLoopTimerRef是基于时间的触发器
- 基本上说的就是NSTimer(CADisplayLink也是加到RunLoop),它受RunLoop的Mode影响
- GCD的定时器不受RunLoop的Mode影响
#### CFRunLoopObserver
- CFRunLoopObserverRef是观察者，能够监听RunLoop的状态改变
- 可以监听的时间点有以下几个

![](http://img8.a.pcs.baidu.com/rest/2.0/pcs/thumbnail?method=generate&path=%2Fimages%2FSnip20160721_13.png&app_id=246327&width=1600&height=1600)



## 应用
- CADisplayLink
- NSTimer
- ImageView显示
- PerformSelector
- 常驻线程
- 自动释放池
- NSURLConnection的执行过程
- AFNetWorking中是如何使用RunLoop的?

### 深入理解Perform Selector
```
- (void)tryPerformSelectorOnMianThread{

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        [self performSelector:@selector(perform1) withObject:nil];
    });
}

- (void)mainThreadMethod{

NSLog(@"execute %s",__func__);

// print: execute -[ViewController mainThreadMethod]
}
```

```
- (void)tryPerformSelectorOnBackGroundThread{

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

[self performSelector:@selector(backGroundThread) onThread:[NSThread currentThread] withObject:nil waitUntilDone:NO];

});
}
- (void)backGroundThread{

NSLog(@"%u",[NSThread isMainThread]);

NSLog(@"execute %s",__FUNCTION__);

}
```
这样我们在ViewDidLoad中调用tryPerformSelectorOnMianThread,就会立即执行，并且输出:print: execute -[ViewController mainThreadMethod];
上边的performselector在子线程中依然可以执行，因为它不依赖于timer，不需要启动runloop，但是下边的做法就不行。

和上面的例子一样，我们使用GCD,让这个方法在后台线程中执行

```
- (void)tryPerformSelectorOnBackGroundThread{

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

[self performSelector:@selector(backGroundThread) onThread:[NSThread currentThread] withObject:nil waitUntilDone:NO];

});
}
- (void)backGroundThread{

NSLog(@"%u",[NSThread isMainThread]);

NSLog(@"execute %s",__FUNCTION__);

}
```
同样的，我们调用tryPerformSelectorOnBackGroundThread这个方法，我们会发现，下面的backGroundThread不会被调用，这是什么原因呢？
这是因为，在调用performSelector:onThread: withObject: waitUntilDone的时候，系统会给我们创建一个Timer的source，加到对应的RunLoop上去，然而这个时候我们没有RunLoop,如果我们加上RunLoop:
```
- (void)tryPerformSelectorOnBackGroundThread{

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

[self performSelector:@selector(backGroundThread) onThread:[NSThread currentThread] withObject:nil waitUntilDone:NO];

NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
[runLoop run];

});
}
```
这时就会发现我们的方法正常被调用了。那么为什么主线程中的perfom selector却能够正常调用呢？通过上面的例子相信你已经猜到了，主线程的RunLoop是一直存在的，所以我们在主线程中执行的时候，无需再添加RunLoop。
小结:当perform selector在后台线程中执行的时候，这个线程必须有一个开启的runLoop

### Perform Selector after
当某个事件触发了某段代码执行的时候，是要在一个runloop中执行完毕的，如果这个线程有runloop的话，这样如果在一个主线程有一大段代码要执行，那么就会阻碍这个线程，或者更准确来讲是让这个runloop不能去处理用户事件和刷新屏幕，那该怎么办呢？
答案就是这个方法`- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay`我们可以把一个大段的方法执行切分成几份，然后用该方法连接，每次执行这个方法，就会进入到下一个runloop才会执行，而每个runloop的执行，用户事件的处理和屏幕刷新是在代码执行前边的，所以这样就有机会中断代码的执行，就像是下边这样
```
@property (nonatomic, strong) UIButton *btn1;
@property (nonatomic, strong) UIButton *btn2;
@property (nonatomic, assign) BOOL next;

@end

int b = 0;

@implementation ViewController

- (UIButton *)btn1 {
    if (!_btn1) {
        _btn1 = [UIButton buttonWithType:UIButtonTypeCustom];
        [_btn1 setTitle:@"点我1" forState:UIControlStateNormal];
        [_btn1 sizeToFit];
        [_btn1 addTarget:self action:@selector(click1) forControlEvents:UIControlEventTouchUpInside];
        [_btn1 setTitleColor:[UIColor redColor] forState:UIControlStateNormal];
    }
    return _btn1;
}

- (UIButton *)btn2 {
    if (!_btn2) {
        _btn2 = [UIButton buttonWithType:UIButtonTypeCustom];
        [_btn2 setTitle:@"点我2" forState:UIControlStateNormal];
        [_btn2 sizeToFit];
        [_btn2 addTarget:self action:@selector(click2) forControlEvents:UIControlEventTouchUpInside];
        [_btn2 setTitleColor:[UIColor redColor] forState:UIControlStateNormal];
    }
    return _btn2;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self.view addSubview:self.btn1];
    [self.view addSubview:self.btn2];
    
    self.btn1.frame = CGRectMake(10, 30, 50, 20);
    self.btn2.frame = CGRectMake(100, 30, 50, 20);
}

- (void)click1 {
    
    for (int i = 0; i < 3000; i++) {
        NSLog(@"%d", i);
    }
[self performSelector:@selector(perform1) withObject:nil afterDelay:0.005];

}

- (void)perform1 {
    for (int i = 3000; i < 6000; i++) {
        NSLog(@"%d", i);
    }
    [self performSelector:@selector(perform4) withObject:nil afterDelay:0.005];

}

- (void)perform4 {
    if (!_next) return;
    for (int i = 6000; i < 9000; i++) {
        NSLog(@"%d", i);
    }
    [self performSelector:@selector(perform5) withObject:nil afterDelay:0.005];
}

- (void)perform5 {
    if (!_next) return;
    for (int i = 600; i < 900; i++) {
        NSLog(@"%d", i);
    }
}

- (void)click2 {
    _next = NO;
}

```
实验证明，只有after方法有这样的魔力。
>总结:当我们创建了一个timer事件的时候，这个时候这个事件就会被放到下一个runloop中执行，如果这个线程中根本就没有起runloop，那么也就不存在下一个runloop，这个事件也就永远不会被执行了。

### CADisplayLink
用法和timer一样，放在子线程的话记得自己创建runloop
```
 dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            // Track FPS using display link
            _displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(displayLinkTick)];
    //        [_displayLink setPaused:YES];
            [_displayLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
            [[NSRunLoop currentRunLoop] run];
        });
```

### Timer
```
@interface ViewController ()
@property (nonatomic, strong) UIScrollView *scrollView;
@property (nonatomic, assign) NSInteger count;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self.view addSubview:self.scrollView];
 
    NSTimer *timer = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(printfString) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];// 标记为common modes的模式：UITrackingRunLoopMode和NSDefaultRunLoopMode兼容,timer就都可以跑了
}

- (UIScrollView *)scrollView {
    if (!_scrollView) {
        _scrollView = [[UIScrollView alloc] initWithFrame:CGRectMake(0, 100, kScreenW, 200)];
        _scrollView.backgroundColor = [UIColor redColor];
        _scrollView.contentSize = CGSizeMake(kScreenW * 2, 0);
    }
    return _scrollView;
}

- (void)printfString {
    self.count ++;
    NSLog(@"%ld", self.count);
}
```
这个时候代码按照我们预定的结果运行，如果我们把这个Tiemr放到后台线程中呢?
```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

    NSTimer *myTimer = [NSTimer scheduledTimerWithTimeInterval:0.5 target:self selector:@selector(timerAction) userInfo:nil repeats:YES];

    [myTimer fire];

});
```
这个时候我们会发现，这个timer只执行了一次，就停止了。这是为什么呢？通过上面的讲解，想必你已经知道了，NSTimer,只有注册到RunLoop之后才会生效，这个注册是由系统自动给我们完成的,既然需要注册到RunLoop,那么我们就需要有一个RunLoop,我们在后台线程中加入如下的代码:
```
NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop run];
```
这样我们就会发现程序正常运行了。在Timer注册到RunLoop之后，RunLoop会为其重复的时间点注册好事件，比如1：10，1：20，1：30这几个时间点。有时候我们会在这个线程中执行一个耗时操作，这个时候RunLoop为了节省资源，并不会在非常准确的时间点回调这个Timer，这就造成了误差(Timer有个冗余度属性叫做tolerance,它标明了当前点到后，容许有多少最大误差)，可以在执行一段循环之后调用一个耗时操作，很容易看到timer会有很大的误差，这说明在线程很闲的时候使用NSTiemr是比较傲你准确的，当线程很忙碌时候会有较大的误差。系统还有一个CADisplayLink,也可以实现定时效果，它是一个和屏幕的刷新率一样的定时器。如果在两次屏幕刷新之间执行一个耗时的任务，那其中就会有一个帧被跳过去，造成界面卡顿。另外GCD也可以实现定时器的效果，由于其和RunLoop没有关联，所以有时候使用它会更加的准确，这在最后会给予说明。
### ImageView

需求:当用户在拖拽时(UI交互时)不显示图片,拖拽完成时显示图片

方法1 监听UIScrollerView滚动 (通过UIScrollViewDelegate监听,此处不再举例)

方法2 RunLoop 设置运行模式

```
// 只在NSDefaultRunLoopMode模式下显示图片
    [self.imageView performSelector:@selector(setImage:) withObject:[UIImage imageNamed:@"placeholder"] afterDelay:3.0 inModes:@[NSDefaultRunLoopMode]];
```
比如修改sdwebimage中的方法，如果设置imageview为滚动列表模式，则加入该方法，当列表停止滚动模式的时候才加载图片

### 一直活着的后台线程
现在有这样一个需求，每点击一下屏幕，让子线程做一个任务,然后大家一般会想到这样的方式:
```
@interface ViewController ()

@property(nonatomic,strong) NSThread *myThread;

@end

@implementation ViewController

- (void)alwaysLiveBackGoundThread{

NSThread *thread = [[NSThread alloc]initWithTarget:self selector:@selector(myThreadRun) object:@"etund"];
self.myThread = thread;
[self.myThread start];

}
- (void)myThreadRun{

NSLog(@"my thread run");

}
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{

    NSLog(@"%@",self.myThread);
    [self performSelector:@selector(doBackGroundThreadWork) onThread:self.myThread withObject:nil waitUntilDone:NO];
}
- (void)doBackGroundThreadWork{

    NSLog(@"do some work %s",__FUNCTION__);

}
@end
```
这个方法中，我们利用一个强引用来获取了后天线程中的thread,然后在点击屏幕的时候，在这个线程上执行doBackGroundThreadWork这个方法，此时我们可以看到，在touchesBegin方法中，self.myThread是存在的，但是这是为是什么呢？这就要从线程的五大状态来说明了:新建状态、就绪状态、运行状态、阻塞状态、死亡状态，这个时候尽管内存中还有线程，但是这个线程在执行完任务之后已经死亡了，经过上面的论述，我们应该怎样处理呢？我们可以给这个线程的RunLoop添加一个source，那么这个线程就会检测这个source等待执行，而不至于死亡(有工作的强烈愿望而不死亡):
```
- (void)myThreadRun{

[[NSRunLoop currentRunLoop] addPort:[[NSPort alloc] init] forMode:NSDefaultRunLoopMode];
[[NSRunLoop currentRunLoop] run]

  NSLog(@"my thread run");

}
```
这个时候再次点击屏幕，我们就会发现，后台线程中执行的任务可以正常进行了。
小结:正常情况下，后台线程执行完任务之后就处于死亡状态，我们要避免这种情况的发生可以利用RunLoop，并且给它一个Source这样来保证线程依旧还在

### NSURLConnection的执行过程
在使用NSURLConnection时，我们会传入一个Delegate,当我们调用了[connection start]之后，这个Delegate会不停的收到事件的回调。实际上，start这个函数的内部会获取CurrentRunloop，然后在其中的DefaultMode中添加4个source。如下图所示，CFMultiplexerSource是负责各种Delegate回调的，CFHTTPCookieStorage是处理各种Cookie的。如下图所示:
![](006.png)
从中可以看出，当开始网络传输是，我们可以看到NSURLConnection创建了两个新的线程:com.apple.NSURLConnectionLoader和com.apple.CFSocket.private。其中CFSocket是处理底层socket链接的。NSURLConnectionLoader这个线程内部会使用RunLoop来接收底层socket的事件，并通过之前添加的source，来通知(唤醒)上层的Delegate。这样我们就可以理解我们平时封装网络请求时候常见的下面逻辑了:
```
while (!_isEndRequest)
{
    NSLog(@"entered run loop");
    [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
}

NSLog(@"main finished，task be removed");

- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{

  _isEndRequest = YES;

}
```
这里我们就可以解决下面这些疑问了:<br/>

- 为什么这个While循环不停的执行，还需要使用一个RunLoop? 程序执行一个while循环是不会耗费很大性能的，我们这里的目的是想让子线程在有任务的时候处理任务，没有任务的时候休眠，来节约CPU的开支。
- 如果没有为RunLoop添加item,那么它就会立即退出，这里的item呢? 其实系统已经给我们默认添加了4个source了。
- 既然[[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];让线程在这里停下来，那么为什么这个循环会持续的执行呢？因为这个一直在处理任务，并且接受系统对这个Delegate的回调，也就是这个回调唤醒了这个线程，让它在这里循环。


### AFNetWorking中是如何使用RunLoop的?
在AFN中AFURLConnectionOperation是基于NSURLConnection构建的，其希望能够在后台线程来接收Delegate的回调。
为此AFN创建了一个线程,然后在里面开启了一个RunLoop，然后添加item
```
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
@autoreleasepool {
    [[NSThread currentThread] setName:@"AFNetworking"];
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
    [runLoop run];
}

}

+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}
```
这里这个NSMachPort的作用和上文中的一样，就是让线程不至于在很快死亡，然后RunLoop不至于退出(如果要使用这个MachPort的话，调用者需要持有这个NSMachPort，然后在外部线程通过这个port发送信息到这个loop内部,它这里没有这么做)。然后和上面的做法相似，在需要后台执行这个任务的时候，会通过调用:[NSObject performSelector:onThread:..]来将这个任务扔给后台线程的RunLoop中来执行。
```
- (void)start {
[self.lock lock];
if ([self isCancelled]) {
    [self performSelector:@selector(cancelConnection) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
} else if ([self isReady]) {
    self.state = AFOperationExecutingState;
    [self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
}
[self.lock unlock];
}
```
### GCD定时器的实现
```
- (void)gcdTimer{

// get the queue
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);

// creat timer
self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
// config the timer (starting time，interval)
// set begining time
dispatch_time_t start = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC));
// set the interval
uint64_t interver = (uint64_t)(1.0 * NSEC_PER_SEC);

dispatch_source_set_timer(self.timer, start, interver, 0.0);

dispatch_source_set_event_handler(self.timer, ^{

    // the tarsk needed to be processed async
    dispatch_async(dispatch_get_global_queue(0, 0), ^{

        for (int i = 0; i < 100000; i++) {
            NSLog(@"gcdTimer");
        }
    });
});
dispatch_resume(self.timer);

}
```






## 谈谈你对Run Loop的理解

- RunLoop是多线程的一个很重要的机制，就是一个线程一次只能执行一个任务，执行完任务后就会退出线程。主线程会通过do-while死循环让程序持续等待下一个任务不退出。通过mach_msg()让runloop没事时进入trap状态，节省CPU资源。非主线程通常来说就是为了执行某个任务而创建的，执行完就会归还资源，因此默认不开启RunLoop
- 实质上，对于子线程的runloop是默认不存在的，因为苹果采用了懒加载的方式，如果没有手动调用[NSRunLoop currentRunLoop]的话，就不会去查询当前线程的RunLoop，也不会创建、加载
- 当然如果子线程处理完某个任务后不退出，需要继续等待接受事件，需要启动的时候也可以手动启动，比如说添加定时器的时候就要手动开始RunLoop
如何处理事件

- 界面刷新： 当UI改变（ Frame变化、 UIView/CALayer 的继承结构变化等）时，或手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后，这个 UIView/CALayer 就被标记为待处理。 苹果注册了一个用来监听BeforeWaiting和Exit的Observer，在它的回调函数里会遍历所有待处理的 UIView/CAlayer 以执行实际的绘制和调整，并更新 UI 界面。

- 手势识别： 如果上一步的 _UIApplicationHandleEventQueue() 识别到是一个guesture手势，会调用Cancel方法将当前的touchesBegin/Move/End 系列回调打断。随后系统将对应的 UIGestureRecognizer 标记为待处理。 苹果注册了一个 Observer 监测 BeforeWaiting (Loop即将进入休眠) 事件，其回调函数为 _UIGestureRecognizerUpdateObserver()，其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行GestureRecognizer的回调。 当有 UIGestureRecognizer 的变化(创建/销毁/状态改变)时，这个回调都会进行相应处理。

- 网络请求：最底层是CFSocket层，然后是CFNetwork将其封装，然后是NSURLConnection对CFNetwork进行面向对象的封装，NSURLConnection是iOS7中新增的接口。当网络开始传输时，NSURLConnection创建了两个新线程：com.apple.NSURLConnectionLoader和com.apple.CFSocket.private。其中CFSocket线程是处理底层socket连接的。NSURLConnectionLoader这个线程内部会使用RunLoop来接受底层socket的事件，并添加到上层的Delegate

应用

- 滑动与图片刷新：当tableView的cell上有需要从网络获取的图片的时候，滚动tableView，异步线程回去加载图片，加载完成后主线程会设置cell的图片，但是会造成卡顿。可以设置图片的任务在CFRunloopDefaultMode下进行，当滚动tableView的时候，Runloop切换到UITrackingRunLoopMode，不去设置图片，而是而是当停止的时候，再去设置图片。（在viewDidLoad中调用self.imageView performSelector@selector(setImage) withObject:...afterDelay:...inModes@[NSDefayltRunLoopMode]）

- 常驻子线程，保持子线程一直处理事件 为了保证线程长期运转，可以在子线程中加入RunLoop，并且给Runloop设置item，防止Runloop自动退出


## RunLoop处理的几种事件
>static void __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__();
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__();
static void __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__();
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__();
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__();
static void __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__();

他对这六类事件有如下解释

1.Observer事件，runloop中状态变化时进行通知。（微信卡顿监控就是利用这个事件通知来记录下最近一次main runloop活动时间，在另一个check线程中用定时器检测当前时间距离最后一次活动时间过久来判断在主线程中的处理逻辑耗时和卡主线程）。这里还需要特别注意，CAAnimation是由RunloopObserver触发回调来重绘，接下来会讲到。

2.Block事件，非延迟的NSObject PerformSelector立即调用，dispatch_after立即调用，block回调。

3.Main_Dispatch_Queue事件：GCD中dispatch到main queue的block会被dispatch到main loop执行。

4.Timer事件：延迟的NSObject PerformSelector，延迟的dispatch_after，timer事件。CADisplayLink也是这里触发

5.Source0事件：处理如UIEvent，CFSocket这类事件。需要手动触发。触摸事件其实是Source1接收系统事件后在回调 __IOHIDEventSystemClientQueueCallback() 内触发的 Source0，Source0 再触发的 _UIApplicationHandleEventQueue()。source0一定是要唤醒runloop及时响应并执行的，如果runloop此时在休眠等待系统的 mach_msg事件，那么就会通过source1来唤醒runloop执行。
[self performSelector:@selector(perform1) withObject:nil];是走source0的

6.Source1事件：处理系统内核的mach_msg事件。

验证方法是，打断点，lldb上边的调用栈

### 总结
如果我们想要打断一个方法的执行，一个办法就是用`[self performSelector:@selector(perform4) withObject:nil afterDelay:0.005];`来连接方法的执行，这里延迟时间设置为0有时是无效的，我们可以给一个很小的时间，
