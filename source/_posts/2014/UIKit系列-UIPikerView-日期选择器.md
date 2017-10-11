---
title: UIKit系列-UIPikerView-日期选择器
date: 2014-12-15 11:52:49
tags:
- IOS
---

## 日期选择器
<!--more-->
```
//
//  ViewController.m
//  ceshi
//
//  Created by Hanson on 16/11/15.
//  Copyright © 2016年 wxg. All rights reserved.
//

#import "ViewController.h"

@interface ViewController ()<UIPickerViewDataSource,UIPickerViewDelegate>
@property (nonatomic, strong) NSMutableArray *yearArr;
@property (nonatomic, strong) NSMutableArray *monthArr;
@property (nonatomic, strong) NSMutableArray *dayArr;

@property (nonatomic, strong) UIPickerView *pickerView;

@property (nonatomic, strong) UIButton *confirmBtn;

@property (nonatomic, assign) NSInteger month;
@property (nonatomic, assign) NSInteger day;

@property (nonatomic, strong) NSMutableArray *singleMonth;
@property (nonatomic, strong) NSMutableArray *doubleMonth;
@end

@implementation ViewController

- (void)viewDidLoad {
[super viewDidLoad];

UIPickerView *pickerView = [[UIPickerView alloc] initWithFrame:CGRectMake(0, 100, 320, 216)];
self.pickerView = pickerView;
// 显示选中框
pickerView.showsSelectionIndicator=YES;
pickerView.dataSource = self;
pickerView.delegate = self;
[self.view addSubview:pickerView];

[self.view addSubview:self.confirmBtn];
}

#pragma Mark -- UIPickerViewDataSource
// pickerView 列数
- (NSInteger)numberOfComponentsInPickerView:(UIPickerView *)pickerView {
return 3;
}

// pickerView 每列个数
- (NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component {
switch (component) {
case 0:
{
return self.yearArr.count;
}
break;
case 1:
{
return self.monthArr.count;
}
break;
case 2:
{
return self.dayArr.count;
}
break;
default:
{
return 0;
}
break;
}
}

// 每列宽度
- (CGFloat)pickerView:(UIPickerView *)pickerView widthForComponent:(NSInteger)component {

    return 110;
}
// 返回选中的行
- (void)pickerView:(UIPickerView *)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component
{
    if (component == 1) {
    self.month = row;
    NSString *month = [self.monthArr objectAtIndex:row];
    NSString *day = [self.dayArr objectAtIndex:self.day];
    [self handleUnselecteDay:day andMonth:month];
    } else if(component == 2){
    self.day = row;
    NSString *day = [self.dayArr objectAtIndex:row];
    NSString *month = [self.monthArr objectAtIndex:self.month];
    [self handleUnselecteDay:day andMonth:month];
    }
}

- (void)handleUnselecteDay:(NSString *)day andMonth:(NSString *)month {

    if ([self.doubleMonth containsObject:month]) {
    if ([day isEqualToString:@"31日"]) {
    [self.pickerView selectRow:30 inComponent:2 animated:YES];
    }
    }else if ([month isEqualToString:@"2月"]) {
    if ([day isEqualToString:@"31日"] || [day isEqualToString:@"30日"]) {
    [self.pickerView selectRow:29 inComponent:2 animated:YES];
    }
    }
}

- (UIView *)pickerView:(UIPickerView *)pickerView viewForRow:(NSInteger)row forComponent:(NSInteger)component reusingView:(UIView *)view {

    UILabel *titleLabel;
    if (view) {
    titleLabel = (UILabel *)view;
    }else {
    titleLabel = [[UILabel alloc] init];
    }

    NSString *title;
    switch (component) {
    case 0:
    {
    title = [self.yearArr objectAtIndex:row];
    }
    break;
    case 1:
    {
    title = [self.monthArr objectAtIndex:row];
    }
    break;
    case 2:
    {
    title = [self.dayArr objectAtIndex:row];
    }
    break;

    default:
    break;
}

titleLabel.text = title;
[titleLabel sizeToFit];

titleLabel.textColor = [UIColor blueColor];

return titleLabel;
}

//返回当前行的内容,此处是将数组中数值添加到滚动的那个显示栏上
-(NSString*)pickerView:(UIPickerView *)pickerView titleForRow:(NSInteger)row forComponent:(NSInteger)component
{
    switch (component) {
    case 0:
    {
        return [self.yearArr objectAtIndex:row];
    }
    break;
    case 1:
    {
        return [self.monthArr objectAtIndex:row];
    }
    break;
    case 2:
    {
        return [self.dayArr objectAtIndex:row];
    }
    break;

    default:
    {
        return nil;
    }
    break;
    }
}

- (void)confirmClick {
    NSInteger row0 = [self.pickerView selectedRowInComponent:0];
    NSInteger row1 = [self.pickerView selectedRowInComponent:1];
    NSInteger row2 = [self.pickerView selectedRowInComponent:2];

    NSString *year = self.yearArr[row0];
    NSString *month = self.monthArr[row1];
    NSString *day = self.dayArr[row2];

    NSLog(@"%@%@%@", year, month, day);
}

#pragma setter and getter
- (NSMutableArray *)yearArr {
    if (!_yearArr) {
        _yearArr = [NSMutableArray array];
        for (int i = 1990; i < 2017; i++) {
            [_yearArr addObject:[NSString stringWithFormat:@"%d年", i]];
        }
    }
return _yearArr;
}

- (NSMutableArray *)monthArr {
    if (!_monthArr) {
        _monthArr = [NSMutableArray array];
        for (int i  = 1; i < 13; i++) {
        [_monthArr addObject:[NSString stringWithFormat:@"%d月", i]];
    }
    }
    return _monthArr;
}

- (NSMutableArray *)dayArr {
    if (!_dayArr) {
        _dayArr = [NSMutableArray array];
        for (int i = 0; i < 32; i++) {
        [_dayArr addObject:[NSString stringWithFormat:@"%d日", i]];
    }
    }
    return _dayArr;
}

- (UIButton *)confirmBtn {
if (!_confirmBtn) {
    _confirmBtn = [UIButton buttonWithType:UIButtonTypeCustom];
        [_confirmBtn setTitle:@"确定" forState:UIControlStateNormal];
        [_confirmBtn sizeToFit];
        [_confirmBtn setTitleColor:[UIColor redColor] forState:UIControlStateNormal];
        [_confirmBtn addTarget:self action:@selector(confirmClick) forControlEvents:UIControlEventTouchUpInside];
    }
    return _confirmBtn;
}

- (NSMutableArray *)singleMonth {//31天
if (!_singleMonth) {
        _singleMonth = [NSMutableArray array];
        [_singleMonth addObjectsFromArray:@[@"1月",@"3月",@"5月",@"7月",@"8月",@"10月",@"12月"]];
   }
   return _singleMonth;
}

- (NSMutableArray *)doubleMonth {
if (!_doubleMonth) {
    _doubleMonth = [NSMutableArray array];
    [_doubleMonth addObjectsFromArray:@[@"4月",@"6月",@"9月",@"11月"]];
  }
  return _doubleMonth;
}
@end

```
