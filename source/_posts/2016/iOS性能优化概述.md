---
title: iOS性能优化概述
date: 2016-07-21 10:51:58
tags:
- iOS进阶
categories: iOS
---
## 响应速度与运行速度
响应速度和运行速度之间有着微妙的区别，响应速度是指监听用户输入到反馈用户的速度，而运行速度是指处理任务的速度。
<!--more-->
用户都讨厌等待，所以你会说让App运行的更快非常重要。确实如此，但是运行速度的提升有一个边界，假如数据要通过网络下载，或者要进行复杂的计算和渲染，那么App不可能立即显示这些内容。这种情况下用户实际上还是愿意等待的，但是你要针对他们的操作给出即时的响应，这种响应可以是简单的按钮状态的改变也可以是复杂的动画效果。让App运行更快很重要，让其迅速响应同样重要。
使用现实中真实的按钮和开关时会让人感觉靠谱，当按下按钮或者打开开关时你可以百分百确定你进行了操作。但是在触摸屏上你无法感知，所以视觉响应非常重要。如果一个App不能提供这种即时的响应那体验将变得非常糟糕，更具体点说就是响应时间不要超过三分之一秒。当你点击了某个位置但是没有任何事情发生，你会自然而然的认为点击有可能没有被接受。绝大多数人在这时会再点击一次，这可能造成重复的操作。

关于响应速度的三个原则：
- 迅速回馈用户他的操作已经被接受，然后迅速执行。例如点击按钮时提供一个touch-down状态呈现给用户。
- 允许用户任意时刻中断
- 当耗时操作进行时，反馈用户一个进度

针对上边提出的问题，给予状态提示是UI层面的事情，比较容易做到，但是如果保证无论任何时候用户的操作和触摸事件都能得到立刻的响应呢？

### runloop与用户事件响应
在每次的runloop中，需要处理下边几种事件：
- handlePort：跨线程通信的一些消息
- customSrc: 被标记为UI待处理的事件
- mySelector：本线程方法的调用
- timerFired：定时器

当我们程序启动完之后，基本上线程就没有什么事情要处理了，这个时候能够做出改变这个状态的事物有下边这几种：
- 用户手势事件，点一下或者拖一下
- 系统消息，来电话等

具体的两种如下面所描述：

- 界面刷新： 当UI改变（ Frame变化、 UIView/CALayer 的继承结构变化等）时，或手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后，这个 UIView/CALayer 就被标记为待处理。 苹果注册了一个用来监听BeforeWaiting和Exit的Observer，在它的回调函数里会遍历所有待处理的 UIView/CAlayer 以执行实际的绘制和调整，并更新 UI 界面。

- 手势识别： 如果上一步的 _UIApplicationHandleEventQueue() 识别到是一个guesture手势，会调用Cancel方法将当前的touchesBegin/Move/End 系列回调打断。随后系统将对应的 UIGestureRecognizer 标记为待处理。 苹果注册了一个 Observer 监测 BeforeWaiting (Loop即将进入休眠) 事件，其回调函数为 _UIGestureRecognizerUpdateObserver()，其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行GestureRecognizer的回调。 当有 UIGestureRecognizer 的变化(创建/销毁/状态改变)时，这个回调都会进行相应处理。

>总的来讲，并不是当用户触发一个事件的时候这个事件就会被马上执行，而是这个事件会被标记，加入被执行的事件队列，然后等待runloop从这个事件队列中取出来事件，然后执行。
现在假如说，你有一个点击事件，这个时候这个事件被加入到待执行的事件中，如果前边有一系列的事件要做，那么这个事件就要排队，现象就是，你发现你点完之后没有任何的反应。
`[self performSelector:@selector(perform4) withObject:nil afterDelay:0.005];`利用该方法将大段的方法进行拆分，这样可以保证用户事件的响应。

## 性能优化
### 运行时间
前边我们保证了响应速度，现在我们来看一下，如何加快执行的速度。
基本上我们能够依赖的就是Time Profiler，

### 渲染速度 
不幸的是Time Profiler不能找出所有的性能问题。当你的App的帧率掉到60（帧/秒）以下你的App就感觉运行得不是那么平滑了。低帧率导致滚动视图和动画卡顿。
　　帧率下降通常意味着iPad的渲染速度跟不上。视觉上较为理想的帧率不低于60（帧/秒），意味着每一帧应该在六十分之一秒内渲染完。

![](http://i2.piimg.com/567571/8a0a7cd7094e8ce0.png)
基本上我们使用上边这些选项来查找影响渲染的因素。
#### color blended layers
我们知道GPU是图形硬件，主要的工作是混合纹理并算出像素的RGB值，这是一个非常复杂的计算过程，计算的过程越复杂，所需要消耗的时间就越长，GPU的使用率就越高，这并不是一个好的现像，而我们需要做的是减少GPU的计算量。
我们在这个图层上放置一个完全不透明的图层，那么GPU将会把上面的图层合成到下面的图层当中，由于上面的是一个完全不透明的图层，所以上面的图层会部份遮盖掉下面的图层，而在遮盖掉的矩形区域内，GPU会直接使用上面图层的像素来显示。如果我们最底的图层上放置的是一个有透明度的图层，那么在这个矩形区域里，GPU需要混合上下两个图层来计算出在屏幕上显示出来的像素的RGB值。若在同一个区域内，存在着多个有透明度的图层，那么GPU需要更多的计算才能得出最终像素的RGB值。而我们要做的就是避免像素混合，尽可能地为视图设置背景色，且设置opaque为YES，这会大大减少GPU的计算。
以label为例
![](http://i2.piimg.com/567571/324a6b489d2c6fa0.png)
上边为不透明，下边为透明，那么我们得到的结果是下边混合严重
上边为iOS7的情况，那么iOS8呢
![](http://i1.piimg.com/567571/10dcf06a1af655ab.png)
虽然设置了背景色，但在iOS8上用UILabel显示中文却出现了像素混合的情况，这是为什么呢？我们来看看UILabel在iOS8前后的变化，在iOS8以前，UILabel使用的是CALayer作为底图层，而在iOS8开始，UILabel的底图层变成了_UILabelLayer，绘制文本也有所改变，就像上图所视(在iOS8刚发布的时候，我一度怀疑Apple歧视中文)。

那怎么解决呢？首先我们来观察一下上图，从图中我们可以看到在背景色的四周多了一圈透明的边，而这一圈透明的边明显超出了图层的矩形区域，既然发现这了一点，那么解决方案就很明了了。

设置图层的masksToBounds为YES时，图层将会沿着Bounds进行裁剪，我们来看一下修改后的效果。木有问题了。
maskTobounds需要与cornerRadius结合才会离屏渲染，所以这里并不会导致离屏渲染。
可以的话，要求美工在切图的时候，一定不要切出那些留有透明区域的图片，不然在你显示图片的时候，同样会出现像素混合问题。

## tableView的优化

iOS平台因为UIKit本身的特性，需要将所有的UI操作都放在主线程执行，所以有时候就习惯将一些线程安全性不确定的逻辑，以及它线程结束后的汇总工作等等放到了主线程，所以主线程包含大量计算、IO、绘制都有可能造成卡顿。

- 可以通过监控runLoop监控监控卡顿，调用方法主要就是在kCFRunLoopBeforeSources和kCFRunLoopBeforeWaiting之间,还有kCFRunLoopAfterWaiting之后,也就是如果我们发现这两个时间内耗时太长,那么就可以判定出此时主线程卡顿.
- 使用到CFRunLoopObserverRef,通过它可以实时获得这些状态值的变化
- 监控后另外再开启一个线程,实时计算这两个状态区域之间的耗时是否到达某个阀值,便能揪出这些性能杀手.
- 监控到了卡顿现场,当然下一步便是记录此时的函数调用信息,此处可以使用一个第三方Crash收集组件PLCrashReporter,它不仅可以收集Crash信息也可用于实时获取各线程的调用堆栈

- 当检测到卡顿时,抓取堆栈信息,然后在客户端做一些过滤处理,便可以上报到服务器,通过收集一定量的卡顿数据后经过分析便能准确定位需要优化的逻辑

- 设置正确的 reuseidentifer 以重用 cell

- 尽量将 View 设置为不透明,包括 cell 本身（backgroundcolor默认是透明的），图层混合靠GPU去渲染,如果透明度设置为100%，那么GPU就会忽略下面所有的layer，节约了很多不必要的运算。模拟器上点击“Debug”菜单，然后选择“color Blended Layers”，会把所有区域分成绿色和红色,绿色的好,红色的性能差（经过混合渲染的），当然也有一些图片虽然是不透明的，但是也会显示红色，如果检查代码没错的话，一般就是图片自身的性质问题了，直接联系美工或后台解决就好了。除非必须要用GPU加载的，其他最好要用CPU加载，因为CPU一般不会百分百加载，可以通过CoreGraphics画出圆角

- 有时候美工失误，图片大小给错了，引起不必要的图片缩放（可以找美工去改，当然也可以异步去裁剪图片然后缓存下来），还是使用Instrument的Color Misaligned Images，黄色表示图片需要缩放，紫色表示没有像素对齐。当然一般情况下图片格式不会给错，有些图片格式是GPU不支持的，就还要劳烦CPU去进行格式转换。还有可以通过Color Offscreen-Rendered Yellow来检测离屏渲染（就是把渲染结果临时保存，等到用的时候再取出，这样相对于普通渲染更消耗内存，使用maskToBounds、设置shadow，重写drawRect方法都会导致离屏渲染）
避免渐变，cornerRadius在默认情况下，这个属性只会影响视图的背景颜色和 border，但是不会离屏绘制，不影响性能。不用clipsToBounds（过多调用GPU去离屏渲染），而是让后台加载图片并处理圆角，并将处理过的图片赋值给UIImageView。UIImageView 的圆角通过直接截取图片实现，圆角路径直接用贝塞尔曲线UIBezierPath绘制（人为指定路径之后就不会触发离屏渲染），UIGraphicsBeginImageContextWithOptions。UIView的圆角可以使用CoreGraphics画出圆角矩形，核心是CGContextAddArcToPoint 函数。它中间的四个参数表示曲线的起点和终点坐标，最后一个参数表示半径。调用了四次函数后，就可以画出圆角矩形。最后再从当前的绘图上下文中获取图片并返回，最后把这个图片插入到视图层级的底部。
“Flash updated Regions”用于标记发生重绘的区域

- 如果 row 的高度不相同,那么将其缓存下来
- 如果 cell 显示的内容来自网络,那么确保这些内容是通过异步下载
- 使用 shadowPath 来设置阴影，图层最好不要使用阴影,阴影会导致离屏渲染(在进入屏幕渲染之前,还看不到的时候会再渲染一次,尽量不要产生离屏渲染)
- 减少 subview 的数量，不要去添加或移除view，要就显示，不要就隐藏
- 在 cellForRowAtIndexPath 中尽量做更少的操作,最好是在别的地方算好，这个方法里只做数据的显示，如果需要做一些处理,那么最好做一次之后将结果储存起来.
- 使用适当的数据结构来保存需要的信息,不同的结构会带来不同的操作代价
- 使用,rowHeight , sectionFooterHeight 和 sectionHeaderHeight 来设置一个恒定高度 , 而不是从代理(delegate)中获取
- cell做数据绑定的时候，最好在willDisPlayCell里面进行，其他操作在cellForRowAtIndexPath，因为前者是第一页有多少条就执行多少次，后者是第一次加载有多少个cell就执行多少次，而且调用后者的时候cell还没显示
- 读取文件,写入文件,最好是放到子线程,或先读取好,在让tableView去显示
- tableView滚动的时候,不要去做动画(微信的聊天界面做的就很好,在滚动的时候,动态图就不让他动,滚动停止的时候才动,不然可能会有点影响流畅度)。在滚动的时候加载图片，停止拖拽后在减速过程中不加载图片，减速停止后加载可见范围内图片

## 代码优化checkList
### 2.3 提前调整ImageView中的图片大小（同图片和动画的渲染）

如果要在UIImageView中显示一个图片，你应保证图片的大小和UIImageView的大小相同。
因为在运行中缩放图片是很耗费资源的，特别是UIImageView嵌套在UIScrollView中的情况下。

如果图片是从远端服务加载的你不能控制图片大小，比如在下载前调整到合适大小的话，你可以在下载完成后，最好是用background thread，缩放一次，然后在UIImageView中使用缩放后的图片。

这个类比到图片和动画的渲染中，是通用的。

具体方法参考上面的GCD操作。

### 2.4 正确使用容器的特性

Arrays: 有序的一组值。使用index来查找很快，使用value 查找很慢， 插入/删除很慢。 Dictionaries: 存储键值对。 用键来查找比较快。 Sets: 无序的一组值。用值来查找很快，插入/删除很快。 

### 2.5 大文件传输使用gzip

大量app依赖于远端资源和第三方API，你可能会开发一个需要从远端下载XML, JSON, HTML或者其它格式的app。

问题是我们的目标是移动设备，因此你就不能指望网络状况有多好。一个用户现在还在edge网络，下一分钟可能就切换到了3G。不论什么场景，你肯定不想让你的用户等太长时间。

减小文档的一个方式就是在服务端和你的app中打开gzip。这对于文字这种能有更高压缩率的数据来说会有更显著的效用。

当然，现在苹果已经自动支持了，你只需要告诉你们服务端的同学，传输大文件的时候记得用gzip就完了。

### 2.6 View的重用和懒加载

更多的view意味着更多的渲染，也就是更多的CPU和内存消耗，对于那种嵌套了很多view在UIScrollView里边的app更是如此。

重用就是模仿UITableView和UICollectionView的操作: 不要一次创建所有的subview，而是当需要时才创建，当它们完成了使命，把他们放进一个可重用的队列中。
当需要使用View的时候，去可重用队列里面找一找有没有可以被复用的View。
这里我的一份框架中曾经使用过类似的方法去创建一个图片浏览器，大家可以稍做参考。View的重用

懒加载就是在程序启动时并不进行加载，只有当用到这个对象的时候，才进行加载。
这个不仅在属性中可以进行这样的使用，在View上面也是一样，不过实现稍有不同。
懒加载会消耗更少内存，但是在View的显示上会稍有滞后。

### 2.7 Cache

一个极好的原则就是，缓存所需要的，也就是那些不大可能改变但是需要经常读取的东西。

我们能缓存些什么呢？一些选项是，远端服务器的响应，图片，甚至计算结果，比如UITableView的行高。

NSURLConnection默认会缓存资源在内存或者存储中根据它所加载的HTTP Headers。你甚至可以手动创建一个NSURLRequest然后使它只加载缓存的值。

下面是一个可用的代码段，你可以可以用它去为一个基本不会改变的图片创建一个NSURLRequest并缓存它：


+ (NSMutableURLRequest *)imageRequestWithURL:(NSURL *)url { NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];  request.cachePolicy = NSURLRequestReturnCacheDataElseLoad; // this will make sure the request always returns the cached image request.HTTPShouldHandleCookies = NO; request.HTTPShouldUsePipelining = YES; [request addValue:@"image/*" forHTTPHeaderField:@"Accept"];  return request; }


注意你可以通过 NSURLConnection 获取一个URL request， AFNetworking也一样的。这样你就不必为采用这条tip而改变所有的networking代码了。

如果你需要缓存其它不是HTTP Request的东西，你可以用NSCache。

### 2.8 记得处理内存警告

一旦系统内存过低，iOS会通知所有运行中app。在官方文档中是这样记述:


如果你的app收到了内存警告，它就需要尽可能释放更多的内存。最佳方式是移除对缓存，图片object和其他一些可以重创建的objects的strong references.


幸运的是，UIKit提供了几种收集低内存警告的方法:

在app delegate中使用applicationDidReceiveMemoryWarning: 的方法 在你的自定义UIViewController的子类(subclass)中覆盖didReceiveMemoryWarning 注册并接收 UIApplicationDidReceiveMemoryWarningNotification 的通知

一旦收到这类通知，你就需要释放任何不必要的内存使用。


例如，UIViewController的默认行为是移除一些不可见的view， 它的一些子类则可以补充这个方法，删掉一些额外的数据结构。一个有图片缓存的app可以移除不在屏幕上显示的图片。


这样对内存警报的处理是很必要的，若不重视，你的app就可能被系统杀掉。

然而，当你一定要确认你所选择的object是可以被重现创建的来释放内存。一定要在开发中用模拟器中的内存提醒模拟去测试一下。

### 2.9 重用大的开销对象

这里的大开销是指一些初始化很慢的objects，如：NSDateFormatter和NSCalendar。但是，你又不可避免地需要使用它们，比如从JSON或者XML中解析数据。

想要避免使用这个对象的瓶颈你就需要重用他们，可以通过添加属性到你的class里或者创建静态变量来实现。

注意如果你要选择第二种方法，对象会在你的app运行时一直存在于内存中，和单例(singleton)很相似。

下面的代码说明了使用一个属性来延迟加载一个date formatter. 第一次调用时它会创建一个新的实例，以后的调用则将返回已经创建的实例：


// in your .h or inside a class extension @property (nonatomic, strong) NSDateFormatter *formatter;  // inside the implementation (.m) // When you need, just use self.formatter - (NSDateFormatter *)formatter { if (! _formatter) { _formatter = [[NSDateFormatter alloc] init]; _formatter.dateFormat = @"EEE MMM dd HH:mm:ss Z yyyy"; // twitter date format } return _formatter; }


还需要注意的是，其实设置一个NSDateFormatter的速度差不多是和创建新的一样慢的！所以如果你的app需要经常进行日期格式处理的话，你会从这个方法中得到不小的性能提升。

### 2.10 避免反复的处理数据

许多应用需要从服务器加载功能所需的常为JSON或者XML格式的数据。在服务器端和客户端使用相同的数据结构很重要。在内存中操作数据使它们满足你的数据结构是开销很大的。

比如你需要数据来展示一个table view,最好直接从服务器取array结构的数据以避免额外的中间数据结构改变。

类似的，如果需要从特定key中取数据，那么就使用键值对的dictionary。

### 2.11 正确设定背景图片

在View里放背景图片就像很多其它iOS编程一样有很多方法:

使用UIColor的 colorWithPatternImage来设置背景色； 在view中添加一个UIImageView作为一个子View。

如果你使用全画幅的背景图，你就必须使用UIImageView因为UIColor的colorWithPatternImage是用来创建小的重复的图片作为背景的。这种情形下使用UIImageView可以节约不少的内存：


// You could also achieve the same result in Interface Builder UIImageView *backgroundView = [ [UIImageView alloc] initWithImage:[UIImage imageNamed:@"background"]]; [self.view addSubview:backgroundView];


如果你用小图平铺来创建背景，你就需要用UIColor的colorWithPatternImage来做了，它会更快地渲染也不会花费很多内存：


self.view.backgroundColor = [UIColor colorWithPatternImage:[UIImage imageNamed:@"background"]];


### 2.12 试试苹果最新的WKWebView来处理web

UIWebView很有用，用它来展示网页内容或者创建UIKit很难做到的动画效果是很简单的一件事。

但是你可能有注意到UIWebView并不像驱动Safari的那么快。这是由于以JIT compilation 为特色的Webkit的Nitro Engine的限制。

所以想要更高的性能你就要调整下你的HTML了。第一件要做的事就是尽可能移除不必要的javascript，避免使用过大的框架。能只用原生js就更好了。

另外，尽可能异步加载例如用户行为统计script这种不影响页面表达的javascript。

最后，永远要注意你使用的图片，保证图片的符合你使用的大小。使用Sprite sheet提高加载速度和节约内存。

当然，上面是针对你在使用UIWebView的情况下，需要尽量减少使用web的特性，而苹果最近刚推出的Safari的底层框架WKWebView也许能帮我们规避掉很多这样的性能问题。

### 2.13 优化你的TableView

Table view需要有很好的滚动性能，不然用户会在滚动过程中发现动画的瑕疵。

为了保证table view平滑滚动，确保你采取了以下的措施:

正确使用reuseIdentifier来重用cells 尽量使所有的view opaque，包括cell自身 避免渐变，图片缩放，后台选人 缓存行高 如果cell内现实的内容来自web，使用异步加载，缓存请求结果 使用shadowPath来画阴影 减少subviews的数量 尽量不适用cellForRowAtIndexPath:，如果你需要用到它，只用一次然后缓存结果 使用正确的数据结构来存储数据 使用rowHeight, sectionFooterHeight 和 sectionHeaderHeight来设定固定的高，不要请求delegate

### 2.14 选择正确的数据存储方式

当存储大块数据时你会怎么做？

你有很多选择，比如：

使用NSUerDefaults 使用XML, JSON, 或者 plist 使用NSCoding存档 使用类似SQLite的本地SQL数据库 使用 Core Data

NSUserDefaults的问题是什么？虽然它很nice也很便捷，但是它只适用于小数据，比如一些简单的布尔型的设置选项，再大点你就要考虑其它方式了

XML这种结构化档案呢？总体来说，你需要读取整个文件到内存里去解析，这样是很不经济的。使用SAX又是一个很麻烦的事情。

NSCoding？不幸的是，它也需要读写文件，所以也有以上问题。

在这种应用场景下，使用SQLite 或者 Core Data比较好。使用这些技术你用特定的查询语句就能只加载你需要的对象。

在性能层面来讲，SQLite和Core Data是很相似的。他们的不同在于具体使用方法。Core Data代表一个对象的graph model，但SQLite就是一个DBMS。Apple在一般情况下建议使用Core Data，但是如果你有理由不使用它，那么就去使用更加底层的SQLite吧。

如果你使用SQLite，你可以用FMDB(https://github.com/ccgus/fmdb)这个库来简化SQLite的操作，这样你就不用花很多经历了解SQLite的C API了。

### 2.15 把Xib换成Storyboard吧

当你加载一个XIB的时候所有内容都被放在了内存里，包括任何图片。如果有一个不会即刻用到的view，你这就是在浪费宝贵的内存资源了。

Storyboards就是另一码事儿了，storyboard仅在需要时实例化一个view controller.

当加载XIB时，所有图片都被缓存，如果你在做OS X开发的话，声音文件也是。Apple在相关文档中的记述是：


当你加载一个引用了图片或者声音资源的nib时，nib加载代码会把图片和声音文件写进内存。在OS X中，图片和声音资源被缓存在named cache中以便将来用到时获取。在iOS中，仅图片资源会被存进named caches。取决于你所在的平台，使用NSImage 或UIImage 的`imageNamed:`方法来获取图片资源。


很明显，同样的事情也发生在storyboards中，但我并没有找到任何支持这个结论的文档。

另外，快速打开app是很重要的，特别是用户第一次打开它时，对app来讲，第一印象太太太重要了。

你能做的就是使它尽可能做更多的异步任务，比如加载远端或者数据库数据，解析数据。

还是那句话，避免过于庞大的XIB，因为他们是在主线程上加载的。所以尽量使用没有这个问题的Storyboards吧！


注意，用Xcode debug时watchdog并不运行，一定要把设备从Xcode断开来测试启动速度


### 2.16 学会手动创建Autorelease Pool

NSAutoreleasePool负责释放block中的autoreleased objects。一般情况下它会自动被UIKit调用。但是有些状况下你也需要手动去创建它。

假如你创建很多临时对象，你会发现内存一直在减少直到这些对象被release的时候。这是因为只有当UIKit用光了autorelease pool的时候memory才会被释放。

好消息是你可以在你自己的@autoreleasepool里创建临时的对象来避免这个行为：


NSArray *urls = [@"url1",@"url2"]; for (NSURL *url in urls) { @autoreleasepool { NSError *error; NSString *fileContents = [NSString stringWithContentsOfURL: url encoding: NSUTF8StringEncoding error: &error]; /* Process the string, creating and autoreleasing more objects. */ } }


这段代码在每次遍历后释放所有autorelease对象

### 2.17UIImage初始化

A：imagedNamed初始化

B：imageWithContentsOfFile初始化

二者不同之处在于，imageNamed默认加载图片成功后会内存中缓存图片，这个方法用一个指定的名字在系统缓存中查找并返回一个图片对象，如果缓存中没有找到相应的图片对象，则从指定地方加载图片然后缓存对象，并返回这个图片对象

而imageWithContentsOfFile则仅只加载图片，不缓存

大量使用imageNamed方式会在不需要缓存的地方额外增加开销CPU的时间来做这件事，当应用程序需要加载一张比较大的图片并且使用一次性，那么其实是没有必要去缓存这个图片的，用imageWithContentsOfFile是最为经济的方式，这样不会因为UIImage元素较多情况下，CPU会被逐个分散在不必要缓存上浪费过多时间

使用场景需要编程时，应该根据实际应用场景加以区分，UIimage虽小，但使用元素较多问题会有所凸显


### 2.18列表滚动的时候不要对imageview赋值

```
// 只在NSDefaultRunLoopMode模式下显示图片
    [self.imageView performSelector:@selector(setImage:) withObject:[UIImage imageNamed:@"placeholder"] afterDelay:3.0 inModes:@[NSDefaultRunLoopMode]];
```
比如修改sdwebimage中的方法，如果设置imageview为滚动列表模式，则加入该方法，当列表停止滚动模式的时候才加载图片

### 一些第三方的使用
 当然有时候也会用到一些第三方，比如在使用UICollectionView和UITableView的时候，Facebook有一个框架叫AsyncDisplayKit，这个库就可以很好地提升滚动时流畅性以及图片异步下载功能（不支持sb和autoLayout，需要手动进行约束设置），AsyncDisplayKit用相关node类，替换了UIView和它的子类,而且是线程安全的。它可以异步解码图片，调整图片大小以及对图片和文本进行渲染，把这些操作都放到子线程，滑动的时候就流畅许多。我认为这个库最方便的就是实现图片异步解码。UIImage显示之前必须要先解码完成，而且解码还是同步的。尤其是在UICollectionView/UITableView 中使用 prototype cell显示大图，UIImage的同步解码在滚动的时候会有明显的卡顿。另外一个很吸引人的点是AsyncDisplayKit可以把view层次结构转成layer。因为复杂的view层次结构开销很大，如果不需要view特有的功能（例如点击事件），就可以使用AsyncDisplayKit 的layer backing特性从而获得一些额外的提升。当然这个库还处于开发阶段，还有一些地方地方有待完善，比如不支持缓存，我要使用这个库的时候一般是结合Alamofire和AlamofireImage实现图片的缓存

