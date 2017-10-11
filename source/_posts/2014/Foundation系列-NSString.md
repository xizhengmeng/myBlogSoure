---
title: NSString && NSData
date: 2016-11-15 18:08:53
tags:
- iOS
---
### 内存简述，Copy与Strong关键字
#### 内存简述
为了方便观察引用计数，这里在MRC下来进行测试。先重新定义NSLog让其不打印时间戳。再编写一个打印宏，用来打印NSString对象的类、内存地址、值、引用计数。
<!--more-->
```
#define NSLog(FORMAT, ...) fprintf(stderr,"%s\n",[[NSString stringWithFormat:FORMAT, ##__VA_ARGS__] UTF8String]);
#define YLog(_var) ({ NSString *name = @#_var; NSLog(@"%@: %@ -> %p : %@  %tu", name, [_var class], _var, _var, [_var retainCount]); })
```
再新建一个StrObject类，用于测试。该类创建实例时，将打印一个NSString，内容为"Str"。
```
@interface StrObject : NSObject

@end

@implementation StrObject

- (id)init {

  self = [super init];

  if (self) {
    NSLog(@"StrObj Create");
    NSString *strA = @"Str";
    YLog(strA);
}

return self;
}

- (void)dealloc {

  [super dealloc];

  NSLog(@"StrObj Dealloc");
}
```
之后运行以下代码。
（一般来说我们可以用以下几种方法来创建NSString对象，其中stringWithString在iOS 6之后已变为多余的方法（redundant），因其等同于字面量创建法，使用这方法编译器会给出警告，）

```
//创建并释放StrObject对象
StrObject *obj = [StrObject new];
[obj release];

//创建一些字符串
NSString *str1 = @"Str"; YLog(str1);
NSString *str2 = @"Str"; YLog(str2);
NSString *str3 = [NSString stringWithString:@"Str"]; YLog(str3);
NSString *str4 = [NSString stringWithFormat:@"Str"]; YLog(str4);
NSString *str5 = [str1 retain]; YLog(str5);
NSString *str6 = [str1 copy]; YLog(str6);
NSString *str7 = [str1 mutableCopy]; YLog(str7);

//改变NSString变量的字符串内容
str1 = @"StrNew"; YLog(str1);
str2 = str1; YLog(str2);

//创建一个字符串并将其释放
NSString *str8 = @"Str";
[str8 retain];
[str8 release];
[str8 release];
YLog(str8);

//=======输出结果========
//创建并释放StrObject对象
//StrObj Create
//strA: __NSCFConstantString -> 0x10d2ce088 : Str  18446744073709551615
//StrObj Dealloc

//创建一些字符串
//str1: __NSCFConstantString -> 0x10d2ce088 : Str  18446744073709551615
//str2: __NSCFConstantString -> 0x10d2ce088 : Str  18446744073709551615
//str3: __NSCFConstantString -> 0x10d2ce088 : Str  18446744073709551615
//str4: NSTaggedPointerString -> 0xa000000007274533 : Str  18446744073709551615
//str5: __NSCFConstantString -> 0x10d2ce088 : Str  18446744073709551615
//str6: __NSCFConstantString -> 0x10d2ce088 : Str  18446744073709551615
//str7: __NSCFString -> 0x7f951a409c80 : Str  1

//改变NSString变量的字符串内容
//str1: __NSCFConstantString -> 0x10d2ce1e8 : StrNew  18446744073709551615
//str2: __NSCFConstantString -> 0x10d2ce1e8 : StrNew  18446744073709551615

//创建一个字符串并将其释放
//str8: __NSCFConstantString -> 0x10d2ce088 : Str  18446744073709551615
```

可观察到，虽然我们通过不同方法创建了不同的NSString对象，但字符串内容一致，结果显示strA、str1、str2、str3、str5、str6都指向同一个地址。即便@"Str"首先在StrObject对象中出现，赋值给strA，释放了之后，str1和其他对象仍然指向同一个地址。

将一个新的字符串内容@"Str a"赋值给str1，并且将str1赋值给str2后，从结果看到str1、str2的内存地址改变了，且指向同一个地址。

之后创建的str8，对其进行多次release后，内存地址（和strA地址相同）和retaionCount都不曾变化。

##### __NSCFConstantString

这些对象地址相同，是因为他们都是__NSCFConstantString对象，也就是字符串常量对象，可以看到其isa都是__NSCFConstantString，该对象存储在栈上，创建之后由系统来管理内存释放，相同内容的NSCFConstantString对象地址相同。该对象引用计数很大，为固定值不会变化，表示无限运行的retainCount，对其进行retain或release也不会影响其引用计数。

当创建一个NSCFConstantString对象时，会检测这个字符串内容是否已经存在，如果存在，则直接将地址赋值给变量；不存在的话，则创建新地址，再赋值。

总的来说，对于NSCFConstantString对象，只要字符串内容不变，就不会分配新的内存地址，无论你是赋值、retain、copy。这种优化在大量使用NSString的情况下可以节省内存，提高性能。

##### __NSCFString

在上面的输出结果中，我们看到另外还有两类isa分别是：__NSCFString和NSTaggedPointerString，先来看__NSCFString。

在我的理解，__NSCFString对象是一种NSString子类，存储在堆上，不属于字符串常量对象。该对象创建之后和其他的Obj对象一样引用计数为1，对其执行retain和release将改变其retainCount。
```
NSString *str1 = [NSString stringWithFormat:@"ThisIsAStr"];
YLog(str1);
NSString *str2 = [str1 retain];
YLog(str1);
YLog(str2);

NSString *str3 = [str1 copy];
YLog(str1);
YLog(str2);
YLog(str3);

//输出结果
//str1: __NSCFString -> 0x100206c70 : ThisIsAStr  1

//str1 retain后赋值给str2
//str1: __NSCFString -> 0x100206c70 : ThisIsAStr  2
//str2: __NSCFString -> 0x100206c70 : ThisIsAStr  2

//str3由str1浅复制得来
//str1: __NSCFString -> 0x100206c70 : ThisIsAStr  3
//str2: __NSCFString -> 0x100206c70 : ThisIsAStr  3
//str3: __NSCFString -> 0x100206c70 : ThisIsAStr  3

//对str1进行release
//str1: __NSCFString -> 0x100206c70 : ThisIsAStr  2
//str2: __NSCFString -> 0x100206c70 : ThisIsAStr  2
//str3: __NSCFString -> 0x100206c70 : ThisIsAStr  2
```
##### NSTaggedPointerString

这个对象是标签指针，苹果在 64 位环境下对NSString、NSNumber等对象做了一些优化。简单的说就是把指针指向的内容直接放在了指针变量的内存地址中，在 64 位环境下指针变量的大小达到了 8 位，能容纳长度较小的内容，于是使用了标签指针来优化数据的存储。从其引用计数可以看出，这种对象也是无垠的retainCount，这种对象存储在指针的内容中。

对 NSString对象来说，当非字面量的数字，英文字母字符串的长度小于等于9的时候会自动成为NSTaggedPointerString类型，如果有中文或其他特殊符号（可能是非 ASCII 字符）存在的话则会直接成为__NSCFString类型。

```
NSString *str1 = @"123456789"; YLog(str1);
NSString *str2 = [NSString stringWithFormat:@"123456789"]; YLog(str2);
NSString *str3 = [NSString stringWithFormat:@"1234567890"]; YLog(str3);
NSString *str4 = [NSString stringWithFormat:@"五"]; YLog(str4);

//输出结果
//str1: __NSCFConstantString -> 0x10a0c5108 : 123456789  18446744073709551615
//str2: NSTaggedPointerString -> 0xa1ea1f72bb30ab19 : 123456789  18446744073709551615
//str3: __NSCFString -> 0x7f8183c02630 : 1234567890  1
//str4: __NSCFString -> 0x7f8183c0fae0 : 五  1
```

#### 声明NSSting为属性时用copy还是strong
声明NSString属性一般来说用copy，因为父类指针可以指向子类对象，而NSMutableNSString是NSString的子类，使用strong的话该NSString属性可能指向一个NSMutableNSString可变对象，如果这个可变对象的内容在外部被修改了，那该属性所属的对象可能对此毫不知情。

先来看正常的情况，NSString属性指向一个不可变对象。
```
//Person对象
@interface Person : NSObject
@property (nonatomic, strong) NSString *name;
@end

@implementation Person
@end

//测试代码
NSString *strA = @"Susan_Miller";
Person *personA = [Person new];
personA.name = strA;
YLog(strA);
YLog(personA.name);

strA = @"Susan_Test";
YLog(strA);
YLog(personA.name);

//输出结果
//strA: __NSCFConstantString -> 0x102f20108 : Susan_Miller  18446744073709551615
//personA.name: __NSCFConstantString -> 0x102f20108 : Susan_Miller  18446744073709551615
//strA: __NSCFConstantString -> 0x102f20148 : Susan_Test  18446744073709551615
//personA.name: __NSCFConstantString -> 0x102f20108 : Susan_Miller  18446744073709551615
```

从结果可观察到此时的name属性指向一个不可变的字符串常量，就算strA因为内容变化而生成了新地址，对name属性不会有影响，无论声明name属性时关键字用copy还是strong（不可变对象的copy是浅复制，strong也是指针引用，所以strA内容改变后地址同时改变，不会影响name属性）。

再来看name属性指向一个可变对象的情况，属性关键字用strong。
```
//属性关键字为strong
@property (nonatomic, strong) NSString *name;

//测试代码
NSMutableString *strB = [@"Susan_Miller" mutableCopy];
Person *personB = [Person new];
personB.name = strB;
YLog(strB);
YLog(personB.name);

[strB setString:@"Susan_Test"];
YLog(strB);
YLog(personB.name);

//输出结果
//strB: __NSCFString -> 0x7ff9b2c1a090 : Susan_Miller  2
//personB.name: __NSCFString -> 0x7ff9b2c1a090 : Susan_Miller  2
//strB: __NSCFString -> 0x7ff9b2c1a090 : Susan_Test  2
//personB.name: __NSCFString -> 0x7ff9b2c1a090 : Susan_Test  2
```
从结果可看出，name属性指向了一个可变对象strB，当strB的内容改变时，name属性也跟着改变，而person对此不知情，可能会产生错误。

继续来看name属性关键字使用copy时的情况。
```
//属性关键字为copy
@property (nonatomic, copy) NSString *name;

//测试代码
NSMutableString *strB = [@"Susan_Miller" mutableCopy];
Person *personB = [Person new];
personB.name = strB;
YLog(strB);
YLog(personB.name);

[strB setString:@"Susan_Test"];
YLog(strB);
YLog(personB.name);

//输出结果
//strB: __NSCFString -> 0x7fb15af31360 : Susan_Miller  1**
//personB.name: __NSCFString -> 0x7fb15af26620 : Susan_Miller  1**
//strB: __NSCFString -> 0x7fb15af31360 : Susan_Test  1**
//personB.name: __NSCFString -> 0x7fb15af26620 : Susan_Miller  1**
```
此时，因为对NSMutableString进行copy是深复制（即内容拷贝），所以name属性与strB指向不同的地址，strB的内容更改不会影响到name属性。

所以，声明NSString为属性时，如果希望保护属性封装性不受外界影响，则应该使用copy关键字，让所属对象持有的是一份“不可变”（immutable）副本，不用担心字符串内容无意间变动。

### 去除字符串中的中文
```
- (NSString *)removeChinese:(NSString *)string {

    NSString *chi = [self getChineseStringWithString:string];

    return [string stringByReplacingOccurrencesOfString:chi withString:@""];
}

- (NSString *)getChineseStringWithString:(NSString *)string
{
    //(unicode中文编码范围是0x4e00~0x9fa5)
    for (int i = 0; i < string.length; i++) {
        int utfCode = 0;
        void *buffer = &utfCode;
        NSRange range = NSMakeRange(i, 1);

        BOOL b = [string getBytes:buffer maxLength:2 usedLength:NULL encoding:NSUTF16LittleEndianStringEncoding options:NSStringEncodingConversionExternalRepresentation range:range remainingRange:NULL];

        if (b && (utfCode >= 0x4e00 && utfCode <= 0x9fa5)) {
            return [string substringFromIndex:i];
        }
    }
    return nil;
}
```

### - (BOOL)containsString:(NSString *)str NS_AVAILABLE(10_10, 8_0);//7以下不可用

### NSString的各种转换
#### NSData 与 NSString
```
　　NSData --> NSString
　　NSString *aString = [[NSString alloc] initWithData:adata encoding:NSUTF8StringEncoding];
　　NSString --> NSData
　　NSString *aString = @"1234";
　　NSData *aData = [aString dataUsingEncoding: NSUTF8StringEncoding];
```

#### NSString转化为UNICODE String：
```
(NSString*)fname ＝ @“Test”;
char fnameStr[10];
memcpy(fnameStr, [fname cStringUsingEncoding:NSUnicodeStringEncoding], 2*([fname length]));
与strcpy相比，memcpy并不是遇到'\0'就结束，而是一定会拷贝完n个字节
```

#### NSString 转化为 char *
```
NSString * str＝ @“Test”;
const char * a =[str UTF8String];
```

#### char * 转化为 NSString
```
NSString *str=[NSString stringWithCString encoding:NSUTF8StringEncoding];
```

#### char * 转化 NSData
```
方法一： char * a = (char*)malloc(sizeof(byte)*16); NSData *data = [NSData dataWithBytes: a length:strlen(a)]; 
方法二： 转换为NSString： - (id)initWithUTF8String:(const char *)bytes 然后用NSString的 - (NSData *)dataUsingEncoding:(NSStringEncoding)encoding
```

#### NSData 转化 char *
```
NSData data ； 
char* a=[data bytes];
```

#### NSString 转化 NSURL
```
//NSURL *url = [NSURL URLWithString:[str  stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding ]];

NSString *urlString=[@"http://www.google.com/search?client=safari&rls=en&q=搜索&ie=UTF-8&oe=UTF-8" stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
```

#### NSURL 转化 NSString
```
NSURL *url=[NSURL URLWithString:urlString];

NSString *s=[[url absoluteString] stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
```

#### NSData 与 Byte
```
　　NSData --> Byte
　　NSString *testString = @"1234567890";
　　NSData *testData = [testString dataUsingEncoding: NSUTF8StringEncoding];
　　Byte *testByte = (Byte *)[testData bytes];
　　Byte --> NSData
　　Byte byte[] = {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23};
　　NSData *adata = [[NSData alloc] initWithBytes:byte length:24];
```
#### NSData 与 UIImage
```
　　NSData --> UIImage
　　UIImage *aimage = [UIImage imageWithData: imageData];
　　//例：从本地文件沙盒中取图片并转换为NSData
　　NSString *path = [[NSBundle mainBundle] bundlePath];
　　NSString *name = [NSString stringWithFormat:@"ceshi.png"];
　　NSString *finalPath = [path stringByAppendingPathComponent:name];
　　NSData *imageData = [NSData dataWithContentsOfFile: finalPath];
　　UIImage *aimage = [UIImage imageWithData: imageData];
　　UIImage－> NSData
　　NSData *imageData = UIImagePNGRepresentation(aimae);
```
#### NSData 与 NSMutableData
```
　　NSData --> MSMutableData
　　NSData *data=[[NSData alloc]init];
　　NSMutableData *mdata=[[NSMutableData alloc]init];
　　mdata=[NSData dataWithData:data];
```
#### NSData合并为一个NSMutableData
```
- (NSString *)filePathWithName:(NSString *)filename
{
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *documentsDirectory = [paths objectAtIndex:0];
return [documentsDirectory stringByAppendingPathComponent:filename];
}

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
//音频文件路径
NSString *mp3Path1 = [[NSBundle mainBundle] pathForResource:@"1" ofType:@"mp3"];
NSString *mp3Path2 = [[NSBundle mainBundle] pathForResource:@"2" ofType:@"mp3"];
//音频数据
NSData *sound1Data = [[NSData alloc] initWithContentsOfFile: mp3Path1];
NSData *sound2Data = [[NSData alloc] initWithContentsOfFile: mp3Path2];
//合并音频
NSMutableData *sounds = [NSMutableData alloc];
[sounds appendData:sound1Data];
[sounds appendData:sound2Data];
//保存音频

NSLog(@"data length:%d", [sounds length]);

[sounds writeToFile:[self filePathWithName:@"tmp.mp3"] atomically:YES];

[window makeKeyAndVisible];

return YES;
}
```

### 转译字符
> 转义字符 意义 ASCII码值(十进制)
\a 响铃(BEL) 007
\b 退格(BS) 008
\f 换页(FF) 012
\n 换行(LF) 010
\r 回车(CR) 013
\t 水平制表(HT) 009
\v 垂直制表(VT) 011
\\ 反斜杠 092
\? 问号字符 063
\' 单引号字符 039
\" 双引号字符 034
\0 空字符(NULL) 000
\ddd 任意字符 三位八进制
\xhh 任意字符 二位十六进制
\a:蜂鸣，响铃
\b:回退：向后退一格
\f:换页
\n:换行，光标到下行行首
\r:回车，光标到本行行首
\t:水平制表
\v:垂直制表
\\:反斜杠
\':单引号
\":双引号
\?:问号
\ddd:三位八进制
\xhh:二位十六进制
\0:空字符(NULL),什么都不做
注：
1，\v垂直制表和\f换页符对屏幕没有任何影响，但会影响打印机执行响应操作。
2，\n其实应该叫回车换行。换行只是换一行，不改变光标的横坐标；回车只是回到行首，不改变光标的纵坐标。
3，\t 光标向前移动四格或八格，可以在编译器里设置
4，\' 在字符里（即单引号里）使用。在字符串里(即双引号里)不需要，只要用 ' 即可。
5，\? 其实不必要。只要用 ? 就可以了（在windows VC6 和tc2 中验证）。


### 常用的富文本
#### 单纯改变一句话中的某些字的颜色（一种颜色）
```
/**
*  单纯改变一句话中的某些字的颜色（一种颜色）
*
*  @param color    需要改变成的颜色
*  @param totalStr 总的字符串
*  @param subArray 需要改变颜色的文字数组(要是有相同的 只取第一个)
*
*  @return 生成的富文本
*/
+ (NSMutableAttributedString *)ls_changeCorlorWithColor:(UIColor *)color TotalString:(NSString *)totalStr SubStringArray:(NSArray *)subArray {

NSMutableAttributedString *attributedStr = [[NSMutableAttributedString alloc] initWithString:totalStr];
for (NSString *rangeStr in subArray) {

NSRange range = [totalStr rangeOfString:rangeStr options:NSBackwardsSearch];
[attributedStr addAttribute:NSForegroundColorAttributeName value:color range:range];
}

return attributedStr;
}
```

#### 改变每行的字间距
```
/**
*  单纯改变句子的字间距（需要 <CoreText/CoreText.h>）
*
*  @param totalString 需要更改的字符串
*  @param space       字间距
*
*  @return 生成的富文本
*/
+ (NSMutableAttributedString *)ls_changeSpaceWithTotalString:(NSString *)totalString Space:(CGFloat)space {

NSMutableAttributedString *attributedStr = [[NSMutableAttributedString alloc] initWithString:totalString];
long number = space;
CFNumberRef num = CFNumberCreate(kCFAllocatorDefault,kCFNumberSInt8Type,&number);
[attributedStr addAttribute:(id)kCTKernAttributeName value:(__bridge id)num range:NSMakeRange(0,[attributedStr length])];
CFRelease(num);

return attributedStr;
}
```

#### 改变行间距
```
/**
*  单纯改变段落的行间距
*
*  @param totalString 需要更改的字符串
*  @param lineSpace   行间距
*
*  @return 生成的富文本
*/
+ (NSMutableAttributedString *)ls_changeLineSpaceWithTotalString:(NSString *)totalString LineSpace:(CGFloat)lineSpace {

NSMutableAttributedString *attributedStr = [[NSMutableAttributedString alloc] initWithString:totalString];

NSMutableParagraphStyle * paragraphStyle = [[NSMutableParagraphStyle alloc] init];
[paragraphStyle setLineSpacing:lineSpace];

[attributedStr addAttribute:NSParagraphStyleAttributeName value:paragraphStyle range:NSMakeRange(0, [totalString length])];

return attributedStr;
}
```

#### 同时更改行间距和字间距
```
/**
*  同时更改行间距和字间距
*
*  @param totalString 需要改变的字符串
*  @param lineSpace   行间距
*  @param textSpace   字间距
*
*  @return 生成的富文本
*/
+ (NSMutableAttributedString *)ls_changeLineAndTextSpaceWithTotalString:(NSString *)totalString LineSpace:(CGFloat)lineSpace textSpace:(CGFloat)textSpace {

NSMutableAttributedString *attributedStr = [[NSMutableAttributedString alloc] initWithString:totalString];

NSMutableParagraphStyle * paragraphStyle = [[NSMutableParagraphStyle alloc] init];
[paragraphStyle setLineSpacing:lineSpace];

[attributedStr addAttribute:NSParagraphStyleAttributeName value:paragraphStyle range:NSMakeRange(0, [totalString length])];

long number = textSpace;
CFNumberRef num = CFNumberCreate(kCFAllocatorDefault,kCFNumberSInt8Type,&number);
[attributedStr addAttribute:(id)kCTKernAttributeName value:(__bridge id)num range:NSMakeRange(0,[attributedStr length])];
CFRelease(num);

return attributedStr;
}
```

#### 更改某些文字的颜色并修改其字体，突出重点强调
```
/**
*  改变某些文字的颜色 并单独设置其字体
*
*  @param font        设置的字体
*  @param color       颜色
*  @param totalString 总的字符串
*  @param subArray    想要变色的字符数组
*
*  @return 生成的富文本
*/
+ (NSMutableAttributedString *)ls_changeFontAndColor:(UIFont *)font Color:(UIColor *)color TotalString:(NSString *)totalString SubStringArray:(NSArray *)subArray {

NSMutableAttributedString *attributedStr = [[NSMutableAttributedString alloc] initWithString:totalString];

for (NSString *rangeStr in subArray) {

NSRange range = [totalString rangeOfString:rangeStr options:NSBackwardsSearch];

[attributedStr addAttribute:NSForegroundColorAttributeName value:color range:range];
[attributedStr addAttribute:NSFontAttributeName value:font range:range];
}

return attributedStr;
```

### 富文本相关属性汇总
>- NSFontAttributeName ：字体字号
    - value值：UIFont类型
- NSParagraphStyleAttributeName ： 段落样式
    - value值：NSParagraphStyle类型（其属性如下）
    - lineSpacing 行间距(具体用法可查看上面的设置行间距API)
    - paragraphSpacing 段落间距
    - alignment 对齐方式
    - firstLineHeadIndent 指定段落开始的缩进像素
    - headIndent 调整全部文字的缩进像素
- NSForegroundColorAttributeName 字体颜色
    - value值：UIColor类型
- NSBackgroundColorAttributeName 背景颜色
    - value值：UIColor类型
- NSObliquenessAttributeName 字体粗倾斜
    - value值：NSNumber类型
- NSExpansionAttributeName 字体加粗
    - value值：NSNumber类型(比例) 0就是不变 1增加一倍
- NSKernAttributeName 字间距
    - value值：CGFloat类型
- NSUnderlineStyleAttributeName 下划线
    - value值：1或0
- NSUnderlineColorAttributeName 下划线颜色
    - value值：UIColor类型
- NSStrikethroughStyleAttributeName 删除线
    - value值：1或0
- NSStrikethroughColorAttributeName 删除线颜色
    - value值：UIColor类型
- NSStrokeColorAttributeName 字体颜色
    - value值：UIColor类型
- NSStrokeWidthAttributeName 字体描边
    - value值：CGFloat
- NSLigatureAttributeName 连笔字
    - value值：1或0
- NSShadowAttributeName 阴影
    - value值：NSShawdow类型（下面是其属性）
    - shadowOffset 影子与字符串的偏移量
    - shadowBlurRadius 影子的模糊程度
    - shadowColor 影子的颜色
- NSTextEffectAttributeName 设置文本特殊效果,目前只有图版印刷效果可用
    - value值：NSString类型
- NSAttachmentAttributeName 设置文本附件
    - value值：NSTextAttachment类型（没研究过，可自行百度研究）
- NSLinkAttributeName 链接
    - value值：NSURL (preferred) or NSString类型
- NSBaselineOffsetAttributeName 基准线偏移
    - value值：NSNumber类型
- NSWritingDirectionAttributeName 文字方向 分别代表不同的文字出现方向
    - value值：@[@(1),@(2)]
- NSVerticalGlyphFormAttributeName 水平或者竖直文本 在iOS没卵用，不支持竖版
    - value值：1竖直 0水平

### 获取汉字的拼音 
```
+ (NSString *)transform:(NSString *)chinese
{
//将NSString装换成NSMutableString
NSMutableString *pinyin = [chinese mutableCopy];
//将汉字转换为拼音(带音标)
CFStringTransform((__bridge CFMutableStringRef)pinyin, NULL, kCFStringTransformMandarinLatin, NO);
NSLog(@"%@", pinyin);
//去掉拼音的音标
CFStringTransform((__bridge CFMutableStringRef)pinyin, NULL, kCFStringTransformStripCombiningMarks, NO);
NSLog(@"%@", pinyin);
//返回最近结果
return pinyin;
}
```
