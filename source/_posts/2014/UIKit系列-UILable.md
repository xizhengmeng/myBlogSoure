---
title: UIKit系列--UILable
date: 2014-11-16 10:17:32
tags:
- iOS
---

### 多行UILabel的高度计算1
```
UILabel *titleLabel = [UILabel new];
titleLabel.font = [UIFont systemFontOfSize:14];
NSString *titleContent = @"亲，欢迎您通过以下方式与我们的营销顾问取得联系，交流您再营销推广工作中遇到的问题，营销顾问将免费为您提供咨询服务。";
titleLabel.text = titleContent;
titleLabel.numberOfLines = 0;//多行显示，计算高度
titleLabel.textColor = [UIColor lightGrayColor];
CGSize titleSize = [titleContent boundingRectWithSize:CGSizeMake(kScreen_Width, MAXFLOAT) options:NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName:[UIFont systemFontOfSize:14]} context:nil].size;
titleLabel.size = titleSize;
titleLabel.x = 0;
titleLabel.y = 0;

[self.view addSubview:titleLabel];
```

### 多行UILabel的高度计算2

```
- (void)setLineSpace:(CGFloat)lineSpace andMaxWidth:(CGFloat)maxWidth andMaxHeight:(CGFloat)maxHeight andFont:(UIFont *)font textColor:(UIColor *)color withText:(NSString *)text {
    
    [self setLineSpace:lineSpace andMaxWidth:maxWidth andMaxHeight:maxHeight andFont:font textColor:color withText:text andLeftSpace:0];
}

- (void)setLineSpace:(CGFloat)lineSpace andMaxWidth:(CGFloat)maxWidth andMaxHeight:(CGFloat)maxHeight andFont:(UIFont *)font textColor:(UIColor *)color withText:(NSString *)text andLeftSpace:(CGFloat)width {
    if (!text) {
        text = @"";
    }
    self.numberOfLines = 0;
    NSMutableParagraphStyle* parag = [[NSMutableParagraphStyle alloc] init];
    
    [parag setLineSpacing:lineSpace];
    parag.lineBreakMode = NSLineBreakByCharWrapping;
    
    NSDictionary *attributes = @{NSFontAttributeName:font,NSParagraphStyleAttributeName:parag,NSForegroundColorAttributeName:color};
    if (width != 0) {
        [parag setFirstLineHeadIndent:width];//首行缩进
    }
    self.attributedText = [[NSAttributedString alloc] initWithString:text attributes:attributes];
    
    NSMutableParagraphStyle *paraStyle = [[NSMutableParagraphStyle alloc] init];
    paraStyle.lineBreakMode = NSLineBreakByCharWrapping;
    paraStyle.lineSpacing = lineSpace;
    if (width != 0) {
        [paraStyle setFirstLineHeadIndent:width];//首行缩进
    }
    NSDictionary *dic = @{NSFontAttributeName:font, NSParagraphStyleAttributeName:paraStyle, NSKernAttributeName:@1.1f
                          };
    CGFloat maxH = MAXFLOAT;
    if (maxHeight != 0) {
        maxH = maxHeight;
    }
    
    CGSize sizeH = [text boundingRectWithSize:CGSizeMake(maxWidth, maxH) options:NSStringDrawingUsesLineFragmentOrigin attributes:dic context:nil].size;
    
    CGRect frame = self.frame;
    CGPoint position = frame.origin;
    self.frame = CGRectMake(position.x, position.y, maxWidth, sizeH.height);
    self.lineBreakMode = NSLineBreakByTruncatingTail;
}
```
上边的代码基本上做到了，设置好文字的宽度和高度，然后给一段文字可以自动的折行，自动的点点点，这样还有一个问题就是，如果文字只有一行，那么高度可能会有不准，所以在末尾可以进行一下修正
```
self.desLabel.textAlignment = NSTextAlignmentCenter;

[self.desLabel sizeToFit];

if (self.desLabel.height_jr > 30) {
    self.desLabel.height_jr = 30;
}
```
sizeToFit方法基本上是会不改变文字的宽度，然后会让折行消失，然后再到下边设置一下这个高度，高度也是比较容易找到的，只要打印一下想要的行数就可以得到。


