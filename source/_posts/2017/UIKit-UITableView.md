---
title: UIKit-UITableView
date: 2016-11-30 17:30:22
tags:
- iOS
---

### 更改cell点击的高亮色
在具体的Cell中重写函数：
<!--more-->
```
- (void)setHighlighted:(BOOL)highlighted animated:(BOOL)animated{//重写高亮函数
if (highlighted) {
self.textLabel.backgroundColor = [UIColor yellowColor];
self.textLabel.textColor = [UIColor redColor];
self.textLabel.text = [NSString stringWithFormat:@"这是第%ld行  点中我啦哈哈哈",(long)self.tag];
}else{
self.textLabel.backgroundColor = [UIColor clearColor];
self.textLabel.textColor = [UIColor blackColor];
self.textLabel.text = [NSString stringWithFormat:@"这是第%ld行  哈哈哈",(long)self.tag];
}
}
```

### 如何来写分割线
你在第一次cell创建的时候去布局，会发现取到的self.height永远是44，是因为后边系统才会根据你设置的高度来确定这个高度，也就是说它会再次赋值，调用layoutsubviews会再次，所以你的布局方法应该写到这个方法里边，才能够保证你的布局和取到的size是正确的，否则一切都是白费
最佳实践是把布局的方法写到reloadmodel方法里边去，分割线的位置要写到layoutsubviews中去，这样拿到self.height的高度才是准确的，其他的控件是从上到下布局的，所以无所谓，但是从下到上布局的控件就要放倒这个方法中去

### cell的预加载
一旦调用cellfor，紧接着就会调用willdispaly，这个时候这个cell已经出现在了visiblecells里边了，其实这个时候这个cell已经应该出现在屏幕上了，所以预加载的说法在tableview上是不存在的





