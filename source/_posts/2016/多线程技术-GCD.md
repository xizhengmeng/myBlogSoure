---
title: 多线程技术--GCD
date: 2016-07-19 15:12:21
tags:
- iOS进阶
categories: iOS
---

## 什么是GCD

<!--more-->
## GCD的API
- Dipatch是调度的意思
- serial连续
- concurrent同时发生
### Dispatch Queue
>开发者要做的只是定义想执行的任务并追加到适当的Dispatch Queue
源码表示如下：
```
dispatch_async (queue, ^{
  //想执行的任务
}
)
```
Dispatch Queue是执行处理的等待队列，是先入先出的顺序(FIFO)

另外存在两种Dispatch Queue：
- Serial Dispatch Queue，等待现在执行中处理结束
- Concurrent Dispatch Queue，不需要现在执行中处理结束

如下边的代码，追加多个处理
```
dispatch_async(queue, blk0);
dispatch_async(queue, blk1);
dispatch_async(queue, blk2);
dispatch_async(queue, blk3);
dispatch_async(queue, blk4);
```
Concurrent Dispatch Queue之所以不需要等待执行结束，是因为它是使用了多个线程并发执行的，但是并不是有多少个任务就会开多少个线程，线程个数是由系统决定的。

>小结:想要顺序执行就用serial queue,否则使用另外一种

我们现在知道了有queue的存在，那么怎么得到这个queue呢？
### dispatch_queue_create
```
dispatch_queue_t myQueueC = dispatch_queue_create("com.kugou.gcd.myQueue", DISPATCH_QUEUE_CONCURRENT);

dispatch_queue_t myQueueS = dispatch_queue_create("com.kugou.gcd.myQueue", NULL);

```
>第一个参数为名字，一定要设置这个参数，因为发生崩溃的时候，这个将是一个很重要的指引，这个名字会出现在崩溃信息中，第二个参数如果要创建serial queue就设置为NULL，代表空的c指针，如果是另外一种就设置为DISPATCH_QUEUE_CONCURRENT

我们可以创建多个Serial Queue，但是这个时候如果追加任务这些任务是并发执行的，也即是说会创建于queue等数量的线程，那么如果有过多的queue就会有过多的线程，就会消耗不必要的资源，并且会造成线程竞争，这种情况我们要使用`concurrent dispatch queue`

另外要注意的是，虽然我们已经步入了ARC时代，但是Dispatch Queue必须由开发人员来释放，接着上班的代码
```
dispatch_async(myQueueS, ^{NSLog(@"123")});
dispatch_release(myQueueS);
```
在调用async之后立刻释放queue，这样并不会有问题，因为此时block已经对齐有一个引用了。当block结束之后就会释放这个queue。
凡是由create创建的对象都要记得手动释放。
### Main Dispatch Queue/Global Dispatch Queue
第二种获得queue的方法是使用系统标准提供的Dispatch Queue:
- Main Dispatch Queue，主线程中执行，只有一个所以自然是serial dispatch queue
- Global Dispatch Queue，是concurrent dispatch queue，四个优先级

```
//Main Dispatch Queue
dispatch_queue_t mianQueue = dispatch_get_main_queue();

//Global Dispatch Queue高
dispatch_queue_t highQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);

//其他优先级一样的做法
```
对这两种queue使用retaiin和release方法不会产生任何变化，所以这种做法比concurrent dispatch queue更加轻松。

所以我们可以像下边这样直接使用
```
dispatch_asyn(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
  //code
})
```

### dispatch_set_target_queue
dispatch_queue_create函数生成的Dispatch Queue不管是Serial Dispatch Queue还是Concurrent Dispatch Queue，多使用与默认优先级Global Dispatch Queue相同执行优先级的线程，变更执行优先级使用dispatch_set_target_queue函数
```
dispatch_queue_t mySerialQueue = dispatch_queue_create("com.kugou.mySerQueue", NULL);
dispatch_queue_t myGlobalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND,0);
dispatch_set_target_queue(mySerialQueue, myGlobalQueue);
```
指定第一个参数与第二个参数优先级相同

### dispatch_after
用于延时执行
```
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW,3ULL * NSEC_PER_SEC);
dispatch_after(time, dispatch_get_main_queue(),^{
 NSLog(@"123");
})
```
注意，dispatch_after函数并不是在指定时间后执行，而只是在指定时间追加处理到dispatch_queue，此源码在3秒后用dispatch_async函数追加block到main dispatch queue相同。在严格时间要求下这个函数的使用会出问题。

### Dispatch Group
很多情况下我们都需要做结束操作，这个操作肯定是在最后的，如果是serial queue肯定没有问题，把代码放在最后就好了，可是concurrent queue怎么办呢？这个时候我们可以考虑dispatch_grounp
```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0);

dispatch_group_async(group, queue, ^{NSLog(@"1")});
dispatch_group_async(group, queue, ^{NSLog(@"2")});
dispatch_group_async(group, queue, ^{NSLog(@"3")});

dispatch_group_notify(group dispatch_get_main_queue(),^{NSLog(@"done")});

dispatch_release(group);
```
另外，除了使用`dispatch_group_notify`还可以使用`dispatch_group_wait`函数等待全部处理执行结束。
```
long result = dispatch_group_wait(group, DISPATCH_TIME_FOREVER)];
```
第一个参数表示要等待哪个group，第二个参数是等待的时间，这里也可以设置一个dispatch_time，如果这个函数返回值为0，则表示你全部处理完毕，否则代表还有函数没有返回。
等待的含义是，一旦调用wait函数，调用这个函数的线程就会停止，在经过指定的时间或者是group中的全部执行都结束之后，该函数返回。
如果有结束操作这种需求推荐第一种实现方案。

### dispatch_barrier_async
如果现在我们是10个读取的操作，然后并发执行并没有问题，但是如果有一个写入的操作掺杂在其中，这个时候就有点问题了，读写同时进行，就会出问题。

该函数与Concurrent Dispatch Queue同时使用
```
dispatch_queue_t queueC = dispatch_queue_create("com.kugou.gcd.myQueue", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, blk0_reading);
dispatch_async(queue, blk1_reading);
dispatch_async(queue, blk2_reading);
dispatch_barrier_async(queue, blk3_writing);
dispatch_async(queue, blk4_reading);
dispatch_async(queue, blk5_reading);
dispatch_async(queue, blk6_reading);
```
这样写系统会等0，1，2并发执行完，执行blk3，等blk3执行完再并发执行4，5，6

### dispatch_sync

### dispatch_apply

### dispatch_suspend/dispatch_resume

### Dispatch Sempaphore

### dispatch_noce

### Dispatch I/O


## GCD实现

### Dispatch Queue


### Dispatch Source

## GCD经典面试题
在主线程中的某个函数里调用了异步函数，怎么样 block 当前线程 , 且还能响应当前线程的 timer 事件， touch 事件等

>思路便是：利用wait函数，不断地检测是否完成异步操作，然后同时开启一个类死循环，不过这函数的互相调用要用performafter为了给事件和timer调用留出机会和时间。
```
@interface ViewController ()
{
    dispatch_group_t group;
}
@property (nonatomic, strong) RedView *redView;
@property (nonatomic, assign) NSInteger result;
@property (nonatomic, strong) UIButton *btn;
@property (nonatomic, strong) NSTimer *timer;
@end

@implementation ViewController

- (UIButton *)btn {
    if (!_btn) {
        _btn = [UIButton buttonWithType:UIButtonTypeCustom];
        [_btn setTitle:@"点击我啊" forState:UIControlStateNormal];
        [_btn sizeToFit];
        _btn.frame = CGRectMake(100, 100, _btn.bounds.size.width, _btn.bounds.size.height);
        [_btn setTitleColor:[UIColor redColor] forState:UIControlStateNormal];
        [_btn addTarget:self action:@selector(change) forControlEvents:UIControlEventTouchUpInside];
    }
    return _btn;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self.view addSubview:self.btn];
    
    self.timer = [NSTimer scheduledTimerWithTimeInterval:0.5 target:self selector:@selector(chang1) userInfo:nil repeats:YES];
    
    group = dispatch_group_create();
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0);
    dispatch_group_async(group, queue, ^{
        for (int i = 0; i < 10000; i++) {
            NSLog(@"%d", i);
        }
    });
    
    [self wait];
    
}

- (void)wait {
    
    self.result = dispatch_group_wait(group, 10 * NSEC_PER_SEC);
    if (self.result == 0) {
        NSLog(@"结束");
    }else {
        NSLog(@"rsout--%ld", self.result);
        [self performSelector:@selector(wait1) withObject:nil afterDelay:0];
    }
    
}

- (void)wait1 {
    
    self.result = dispatch_group_wait(group, 10 * NSEC_PER_SEC);
    if (self.result == 0) {
        NSLog(@"结束");
    }else {
        NSLog(@"rsout--%ld", self.result);
        [self performSelector:@selector(wait) withObject:nil afterDelay:0];
    }
    
}

- (void)next {
    NSLog(@"1234567");
}

- (void)change {
    self.view.backgroundColor = [UIColor redColor];
}

- (void)chang1 {
    NSLog(@"哇哈哈哈哈");
}
```