---
title: runtime概述
date: 2015-08-10 11:31:29
tags:
- iOS进阶
categories: iOS
---
<!--more-->
## runtime概述
>**什么是runtime?**
runtime直译就是运行时间,run(跑,运行) time(时间),网上大家都叫它运行时,它是一套比较底层的纯C语言API,属于一个C语言库,包含了很多底层的C语言API,它是OC的幕后工作者,我们平时写的OC代码,在运行过程时,都会转为runtime的C语言代码
### 类与对象
####Object
```
interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
```
#### objc_method
```
typedef struct objc_ method {
    SEL method_name;
    char *method_types;
    IMP method_imp;
};
```

#### objc_class
```
struct objc_class {
    Class isa;

#if !__OBJC2__
    Class super_class;                                        
    const char *name;
    long version;                          //类的版本信息，默认为0
    long info;                             //类信息，供运行期使用的一些位标识
    long instance_size;                    //该类的实例变量大小
    struct objc_ivar_list *ivars;          //成员列表
    struct objc_method_list **methodLists; //方法列表
    struct objc_cache *cache;              //方法缓存
    struct objc_protocol_list *protocols;  //协议链表
#endif

} OBJC2_UNAVAILABLE;
```
#### Method & SEL & IMP
>- Method:`(typedef struct objc_method *Method)`在类定义中表示方法的类型，相当于SEL和IMP的映射
>- SEL(selector):`typedef struct objc_selector *SEL`方法选择器，用于在运行时中表示一个方法的名称。一个方法选择器是一个C字符串，它是在Objective-C运行时被注册的。选择器由编译器生成，并且在类被加载时由运行时自动做映射操作。
>- IMP(implement):`typedef id (*IMP)(id, SEL, …)`方法的实现，这是一个指针类型，指向方法实现函数的开始位置。这个函数使用为当前CPU架构实现的标准C调用规范。每一个参数是指向对象自身的指针(self)，第二个参数是方法选择器。然后是方法的实际参数。

------
>**注意**，SEL是由方法的名字，也就是字符串来生成的(包含参数)，也就是说，在不同的类中的方法，只要名字和参数相同，那么它的SEL也是相同的，SEL本质是一个Int类型的一个地址(用%p来表示)，地址中存放着方法的名字所有的SEL存储在一个NSSet中，所以SEL是唯一的，以hash实现搜索定位，效率高于NSArray，在方法的调用的时候，是通过数字的查找而非字符串的查找来进行方法定位的，当我定义一个类的时候，有声明或者实现，都会有对应的SEL以及Method对象存在，但是没有实现是IMP则为空，`SEL selector = NSSelectorFromString(@"run");`这个代码永远是正确的，但是`IMP ipmSel = [FXViewController methodForSelector:selector];`如果这个类没有这个实现，那么是会返回`(libobjc.A.dylib_objc_msgForward)`。
>所以理论上来讲iOS不存在私有方法，因为我们可以通过运行时来调用任何存在的method，只不过这个方法没有在.h文件进行声明的话，在编译层面会进行一些禁止而已，这样也保障了运行的安全。
```
SEL selector = NSSelectorFromString(@"run");
Class class = NSClassFromString(@"FXViewController");
if ([class respondsToSelector:selector]) {
    [class performSelector:selector withObject:nil];
}
```
当我们在类中进行方法的声明或者实现的时候，底层会默认根据方法名，来注册这个方法，生成对应的SEL和Method，SEL会由一个NSSet来做一个统一的维护，另外Method对象会保存在对应的类的methodlist中，当我们在方法调用的时候，系统会快速的通过注册方法来拿到SEL，然后到对应的类的实现中去根据SEL查找Method，如果Method存在则直接跳转到IMP去调用其指向的函数。如果一个方法只有声明没有实现，那么这个类中是不会有Method对象存在的，可以用`class_copyMethodList`进行验证
![Alt text](./1460617119227.png)
这张图在加载类的时候创建，当发现有方法声明，或者方法实现，或者方法调用的时候，就会把对应的字符串进行一个注册，形成上边那样的一张表，这张表中的SEL是经过运算得到的，而不是查找。
#### 运行时方法总结
`通过名字获取SEL，通过SEL和Class获取IMP`
```
// 添加方法
BOOL class_addMethod ( Class cls, SEL name, IMP imp, const char *types );
// 获取实例方法
Method class_getInstanceMethod ( Class cls, SEL name );
// 获取类方法
Method class_getClassMethod ( Class cls, SEL name );
// 获取所有实例方法的数组，不能获取类方法
Method * class_copyMethodList ( Class cls, unsigned int *outCount );
// 替代方法的实现，如果IMP不存在则默认调用class_addMethod方法
IMP class_replaceMethod ( Class cls, SEL name, IMP imp, const char *types );
// 返回方法的具体实现
IMP class_getMethodImplementation ( Class cls, SEL name );
IMP class_getMethodImplementation_stret ( Class cls, SEL name );
// 类实例是否响应指定的selector
BOOL class_respondsToSelector ( Class cls, SEL sel );
```

#### 获取列表
```
 unsigned int count;
    //获取属性列表
    objc_property_t *propertyList = class_copyPropertyList([self class], &count);
    for (unsigned int i=0; i<count; i++) {
        const char *propertyName = property_getName(propertyList[i]);
        NSLog(@"property---->%@", [NSString stringWithUTF8String:propertyName]);
    }

    //获取方法列表
    Method *methodList = class_copyMethodList([self class], &count);
    for (unsigned int i; i<count; i++) {
        Method method = methodList[i];
        NSLog(@"method---->%@", NSStringFromSelector(method_getName(method)));
    }

    //获取成员变量列表
    Ivar *ivarList = class_copyIvarList([self class], &count);
    for (unsigned int i; i<count; i++) {
        Ivar myIvar = ivarList[i];
        const char *ivarName = ivar_getName(myIvar);
        NSLog(@"Ivar---->%@", [NSString stringWithUTF8String:ivarName]);
    }

    //获取协议列表
    __unsafe_unretained Protocol **protocolList = class_copyProtocolList([self class], &count);
    for (unsigned int i; i<count; i++) {
        Protocol *myProtocal = protocolList[i];
        const char *protocolName = protocol_getName(myProtocal);
        NSLog(@"protocol---->%@", [NSString stringWithUTF8String:protocolName]);
    }
```

### 方法的执行
- 调用类的alloc方法，分配内存，初始化成员变量，其中isa指针也会被初始化，让这个对象有访问其类的能力；
- 当我们调用某个方法的时候，编译器会对这个方法调用进行转换

```
//1.利用msgSend函数
	[someone method];
	[someone performSelector:@selector(method)];
	---->objc_msgSend(someone, @selector(method));
//2.利用转发函数
	NSinvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSignature];
[invocation invoke];

```

> `objc_msgSend`的作用:
> - 它首先找到 SEL 对应的方法实现 IMP。因为不同的类对同一方法可能会有不同的实现，所以找到的方法实现依赖于消息接收者的类型
>-  然后将消息接收者对象(指向消息接收者对象的指针)以及方法中指定的参数传递给方法实现 IMP。
>-  最后，将方法实现的返回值作为该函数的返回值返回。


#### 注意：
当我们创建一个对象的时候，首先根据类的info来分配内存，然后初始化成员变量，其中isa指针也会被初始化，让这个对象有访问其类的能力，当消息发送给一个对象时，objc_msgSend通过对象的isa指针获取到类的结构体，然后在方法分发表里面查找方法的selector。如果没有找到selector，则通过objc_msgSend结构体中的指向父类的指针找到其父类，并在父类的分发表里面查找方法的selector。依此，会一直沿着类的继承体系到达NSObject类。一旦定位到selector，函数会就获取到了实现的入口点，并传入相应的参数来执行方法的具体实现。如果最后没有定位到selector，则会走消息转发流程。
动态绑定为我们写代码提供了方便，却带来了性能的损耗，因为函数的调用通过地址可以直接确定，而方法需要通过函数的查找，但是方法的缓存一定程度上解决了这个问题。

### 消息转发机制
消息转发用到的方法(消息拦截用到的方法)

```
+ (BOOL)resolveClassMethod:(SEL)sel;
+ (BOOL)resolveInstanceMethod:(SEL)sel;
//后两个方法需要转发到其他的类处理
- (id)forwardingTargetForSelector:(SEL)aSelector;
- (void)forwardInvocation:(NSInvocation *)anInvocation;
```
![](http://7xrn7f.com1.z0.glb.clouddn.com/16-7-17/16603936.jpg)


我们调用一个方法有两种形式
```
[object message];

[object performSelector:@selector(method) withObject:nil];
```
第一种我们可以确定对象是否能够处理该方法，如果没有声明编译器会直接报错，不过也存在没有方法实现的风险，第二种方式就会直接编译通过，运行崩溃。
规避的方法有两种
- 第一就是在调用方法之前判断是否该方法实现
```
if ([self respondsToSelector:@selector(method)]) {
    [self performSelector:@selector(method)];
}
```
- 消息转发机制：通过这一机制，我们可以告诉对象如何处理未知的消息。
>默认情况下，对象接收到未知的消息，会导致程序崩溃，通过控制台，我们可以看到以下异常信息：

```
-[SUTRuntimeMethod method]: unrecognized selector sent to instance 0x100111940

*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[SUTRuntimeMethod method]: unrecognized selector sent to instance 0x100111940'
```

>这段异常信息实际上是由NSObject的”doesNotRecognizeSelector”方法抛出的。不过，我们可以采取一些措施，让我们的程序执行特定的逻辑，而避免程序的崩溃。
>消息转发机制基本上分为三个步骤：
- 动态方法解析
- 备用接收者
- 完整转发


#### 动态方法解析
对象在接收到未知的消息时，首先会调用所属类的类方法+resolveInstanceMethod:(实例方法)或者+resolveClassMethod:(类方法)。在这个方法中，我们有机会为该未知消息新增一个”处理方法”“。不过使用该方法的前提是我们已经实现了该”处理方法”，只需要在运行时通过class_addMethod函数动态添加到类里面就可以了。

```
void functionForMethod1(id self, SEL _cmd) {

   NSLog(@"%@, %p", self, _cmd);

}
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSString *selectorString = NSStringFromSelector(sel);
    if ([selectorString isEqualToString:@"method1"]) {
        class_addMethod(self.class, @selector(method1), (IMP)functionForMethod1, "@:");
    }
    return [super resolveInstanceMethod:sel];
}
```
不过这种方案更多的是为了实现@dynamic属性。
#### 备用接收者

```
- (id)forwardingTargetForSelector:(SEL)aSelector
```

如果一个对象实现了这个方法，并返回一个非nil的结果，则这个对象会作为消息的新接收者，且消息会被分发到这个对象。当然这个对象不能是self自身，否则就是出现无限循环。当然，如果我们没有指定相应的对象来处理aSelector，则应该调用父类的实现来返回结果。

```
interface SUTRuntimeMethodHelper : NSObject
- (void)method2;
end
implementation SUTRuntimeMethodHelper
- (void)method2 {
    NSLog(@"%@, %p", self, _cmd);
}
@(02-Foundation)end
#pragma mark -
@interface SUTRuntimeMethod () {
    SUTRuntimeMethodHelper *_helper;
}
@end
@implementation SUTRuntimeMethod
+ (instancetype)object {
    return [[self alloc] init];
}
- (instancetype)init {
    self = [super init];
    if (self != nil) {
        _helper = [[SUTRuntimeMethodHelper alloc] init];
    }
    return self;
}
- (void)test {
    [self performSelector:@selector(method2)];
}
- (id)forwardingTargetForSelector:(SEL)aSelector {
    NSLog(@"forwardingTargetForSelector");
    NSString *selectorString = NSStringFromSelector(aSelector);
    // 将消息转发给_helper来处理
    if ([selectorString isEqualToString:@"method2"]) {
        return _helper;
    }
    return [super forwardingTargetForSelector:aSelector];
}
@end
```
#### 完整转发
如果在上一步还不能处理未知消息，则唯一能做的就是启用完整的消息转发机制了。此时会调用以下方法
```
- (void)forwardInvocation:(NSInvocation *)anInvocation
```
>对象会创建一个表示消息的NSInvocation对象，把与尚未处理的消息有关的全部细节都封装在anInvocation中，包括selector，目标(target)和参数。我们可以在forwardInvocation方法中选择将消息转发给其它对象
```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
    if (!signature) {
        if ([SUTRuntimeMethodHelper instancesRespondToSelector:aSelector]) {
            signature = [SUTRuntimeMethodHelper instanceMethodSignatureForSelector:aSelector];
        }
    }
    return signature;
}
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([SUTRuntimeMethodHelper instancesRespondToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:_helper];
    }
}
```
>NSObject的forwardInvocation:方法实现只是简单调用了doesNotRecognizeSelector:方法，它不会转发任何消息。这样，如果不在以上所述的三个步骤中处理未知消息，则会引发一个异常。

### 动态方法添加
首先调用一个不存在的方法
```
//隐式调用方法
[target performSelector:@selector(resolveAdd:) withObject:@"test"];
```
然后，在target对象内部重写拦截调用的方法，动态添加方法。
```
void runAddMethod(id self, SEL _cmd, NSString *string){
    NSLog(@"add C IMP ", string);
}
+ (BOOL)resolveInstanceMethod:(SEL)sel{

    //给本类动态添加一个方法
    if ([NSStringFromSelector(sel) isEqualToString:@"resolveAdd:"]) {
        class_addMethod(self, sel, (IMP)runAddMethod, "v@:*");
    }
    return YES;
}
```
>其中class_addMethod的四个参数分别是：
- Class cls 给哪个类添加方法，本例中是self
- SEL name 添加的方法，本例中是重写的拦截调用传进来的selector。
- IMP imp 方法的实现，C方法的方法实现可以直接获得。如果是OC方法，可以用- + (IMP)instanceMethodForSelector:(SEL)aSelector;获得方法的实现。
- "v@:*"方法的签名，代表有一个参数的方法

###Method Swizzling
为了swizzle一个方法，我们可以在分发表中将一个方法的现有的选择器映射到不同的实现，而将该选择器对应的原始实现关联到一个新的选择器中。
```
#import <objc/runtime.h>
@implementation UIViewController (Tracking)
+ (void)load {
        static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];         
        // When swizzling a class method, use the following:
                    // Class class = object_getClass((id)self);
        SEL originalSelector = @selector(viewWillAppear:);
                    SEL swizzledSelector = @selector(xxx_viewWillAppear:);
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
                    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        BOOL didAddMethod =
                        class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));
        if (didAddMethod) {
                        class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}
#pragma mark - Method Swizzling
- (void)xxx_viewWillAppear:(BOOL)animated {
        [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}
@end
```
#### 注意
- Swizzling应该总是在+load中执行
>在Objective-C中，运行时会自动调用每个类的两个方法。+load会在类初始加载时调用，+initialize会在第一次调用类的类方法或实例方法之前被调用。这两个方法是可选的，且只有在实现了它们时才会被调用。由于method swizzling会影响到类的全局状态，因此要尽量避免在并发处理中出现竞争的情况。+load能保证在类的初始化过程中被加载，并保证这种改变应用级别的行为的一致性。相比之下，+initialize在其执行时不提供这种保证—事实上，如果在应用中没为给这个类发送消息，则它可能永远不会被调用。

- Swizzling应该总是在dispatch_once中执行
>与上面相同，因为swizzling会改变全局状态，所以我们需要在运行时采取一些预防措施。原子性就是这样一种措施，它确保代码只被执行一次，不管有多少个线程。GCD的dispatch_once可以确保这种行为，我们应该将其作为method swizzling的最佳实践

- 小心无限循环
>咋看上去是会导致无限循环的。但令人惊奇的是，并没有出现这种情况。在swizzling的过程中，方法中的[self xxx_viewWillAppear:animated]已经被重新指定到UIViewController类的-viewWillAppear:中。在这种情况下，不会产生无限循环。不过如果我们调用的是[self viewWillAppear:animated]，则会产生无限循环，因为这个方法的实现在运行时已经被重新指定为xxx_viewWillAppear:了。

### 对象关联
现在你准备用一个系统的类，但是系统的类并不能满足你的需求，你需要额外添加一个属性。
这种情况的一般解决办法就是继承。
但是，只增加一个属性，就去继承一个类，总是觉得太麻烦类。
这个时候，runtime的关联属性就发挥它的作用了。
```
//首先定义一个全局变量，用它的地址作为关联对象的key
static char associatedObjectKey;
//设置关联对象
objc_setAssociatedObject(target, &associatedObjectKey, @"添加的字符串属性", OBJC_ASSOCIATION_RETAIN_NONATOMIC); //获取关联对象
NSString *string = objc_getAssociatedObject(target, &associatedObjectKey);
NSLog(@"AssociatedObject = %@", string);
```
>objc_setAssociatedObject的四个参数：
- id object给谁设置关联对象。
- const void *key关联对象唯一的key，获取时会用到。
- id value关联对象。
- objc_AssociationPolicy关联策略，有以下几种策略：
>enum {
    OBJC_ASSOCIATION_ASSIGN = 0,
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, 
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,
    OBJC_ASSOCIATION_RETAIN = 01401,
    OBJC_ASSOCIATION_COPY = 01403 
};
如果你熟悉OC，看名字应该知道这几种策略的意思了吧。


`objc_getAssociatedObject`的两个参数。
- id object获取谁的关联对象。
- const void *key根据这个唯一的key获取关联对象。
其实，你还可以把添加和获取关联对象的方法写在你需要用到这个功能的类的类别中，方便使用。
```
//添加关联对象
- (void)addAssociatedObject:(id)object{
    objc_setAssociatedObject(self, @selector(getAssociatedObject), object, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
//获取关联对象
- (id)getAssociatedObject{
    return objc_getAssociatedObject(self, _cmd);
}
```
注意：这里面我们把getAssociatedObject方法的地址作为唯一的key，_cmd代表当前调用方法的地址。

### 总结
- 方法调用的本质就是通过instance/class+SEL来找到IMP，最终调用IMP的函数来执行调用的
- 方法调用可以使用invoke的方法
- 增加方法的是在某个类中增加Method，也就是增加SLE和IMP
- 替换方法的是更改SLE与IMP的映射关系

---------------------
参考：
[南峰子-runtime](http://southpeak.github.io/blog/2014/10/25/objective-c-runtime-yun-xing-shi-zhi-lei-yu-dui-xiang/)

[iOS-runtime理解-简书](http://www.jianshu.com/p/927c8384855a)

