---
title: UIKit系列---UITextField
date: 2014-12-12 20:05:50
tags:
- iOS基础知识
categories: iOS
---


### 总述
我们监听textfield的事件总结来看有两种途径，第一种是代理，第二种是通知，代理能够监听的更加广泛一些
<!--more-->
```
- (BOOL)textFieldShouldBeginEditing:(UITextField *)textField;        // return NO to disallow editing.
- (void)textFieldDidBeginEditing:(UITextField *)textField;           // became first responder
- (BOOL)textFieldShouldEndEditing:(UITextField *)textField;          // return YES to allow editing to stop and to resign first responder status. NO to disallow the editing session to end
- (void)textFieldDidEndEditing:(UITextField *)textField;             // may be called if forced even if shouldEndEditing returns NO (e.g. view removed from window) or endEditing:YES called

- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string;   // return NO to not change text

- (BOOL)textFieldShouldClear:(UITextField *)textField;               // called when clear button pressed. return NO to ignore (no notifications)
- (BOOL)textFieldShouldReturn:(UITextField *)textField;              // called when 'return' key pressed. return NO to ignore.

@end

UIKIT_EXTERN NSString *const UITextFieldTextDidBeginEditingNotification;
UIKIT_EXTERN NSString *const UITextFieldTextDidEndEditingNotification;
UIKIT_EXTERN NSString *const UITextFieldTextDidChangeNotification;
```

### 监听输入
```
[textField addTarget:self action:@selector(textFieldDidChange:) forControlEvents:UIControlEventEditingChanged];
- (void) textFieldDidChange:(UITextField *) TextField{ }
```

### 输入限制
#### 长度
>注意这里得到的string是你输入的那个string，不一定是显示的那个string，经过判断如果你返回yes，那么这个数字将会显示到textfield里边，否则就不会显示的
>并且这里的string，是单个字母或者汉字

```
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string;
{ //string就是此时输入的那个字符textField就是此时正在输入的那个输入框返回YES就是可以改变输入框的值NO相反

if ([string isEqualToString:@"\n"]) //按会车可以改变
   {
       return YES;
   }

   NSString * toBeString = [textField.text stringByReplacingCharactersInRange:range withString:string]; //得到输入框的内容，注意，这里如果直接取textField.text那么你得到的是少一个字母的，也就是你这个取到的string

   if (self.myTextField == textField) //判断是否时我们想要限定的那个输入框
   {
       if ([toBeString length] > 20) { //如果输入框内容大于20则弹出警告
            textField.text = [toBeString substringToIndex:20];
           UIAlertView *alert = [[[UIAlertView alloc] initWithTitle:nil message:@"超过最大字数不能输入了" delegate:nil cancelButtonTitle:@"Ok" otherButtonTitles:nil, nil] autorelease];
           [alert show];
           return NO;
       }
   }
   return YES;
}
```
上边的方法有个问题就是不能有效的统计中文，因为输入中文的时候它首先是输入英文，然后当你选了某个中文文字，英文字符串才会被替换掉，那么我们需要一种更为精准的方法
```
- (void)textFieldLimitWithTextField:(UITextField*)textField length:(NSInteger)length{
    NSString *toBeString = textField.text;
    NSString *lang = [[UITextInputMode currentInputMode] primaryLanguage]; // 键盘输入模式
    if ([lang isEqualToString:@"zh-Hans"]) { // 简体中文输入，包括简体拼音，健体五笔，简体手写
        UITextRange *selectedRange = [textField markedTextRange];
        //获取高亮部分
        UITextPosition *position = [textField positionFromPosition:selectedRange.start offset:0];
        // 没有高亮选择的字，则对已输入的文字进行字数统计和限制
        if (!position) {
            if (toBeString.length > length) {
                textField.text = [toBeString substringToIndex:length];
            }
        }
        // 有高亮选择的字符串，则暂不对文字进行统计和限制
        else{

        }
    }
    // 中文输入法以外的直接对其统计限制即可，不考虑其他语种情况
    else{
        if (toBeString.length > length) {
            textField.text = [toBeString substringToIndex:length];
        }
    }
}
```
直接将textfield和要限制的字数传入就好，不过我们需要不断地传入这个textfield
配合使用
```
- (void)checkInputValid{
//    _userClearBtn.hidden = (0 == _userTextField.text.length );

    [self textFieldLimitWithTextField:_userTextField length:20];

    BOOL isUserValid = (_userTextField.text.length >= 1 && _userTextField.text.length <=20);
    BOOL isPwdValid = (_pwdTextField.text.length >= 1) ;

    [self setLoginBtnEnabled:isUserValid && isPwdValid];
//    [self setClearBtnShow:isUserValid];
}
```
```
[tf addTarget:self action:@selector(textFieldChanged:) forControlEvents:UIControlEventEditingChanged];
```
#### 数值不能超过xx
#### 限制输入的类型
```
- (BOOL)textField:(UITextField*)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString*)string{

   NSString*match =nil;;
   if(textField ==_userTextField){
        match =@"^([0-9]){0,}$";
    }else{
        match =@"^([0-9a-zA-Z]|[\\s|-]){0,}$";
    }

   if(textField ==self.userTextField) {
       if(range.location<_lastLength) {
           self.pwdTextField.text=@"";
            [selfcheckInputValid];
        }
    }

   DLog(@"%@--%@", NSStringFromRange(range) , [NSString stringWithFormat:@"%ld", (long)self.lastLength]);

   if(textField ==self.userTextField) {
        self.lastLength= range.location;
    }

   NSPredicate*predicate = [NSPredicatepredicateWithFormat:@"SELF MATCHES %@", match];
   return[predicateevaluateWithObject:string];
}
```
### 样式设置
```
searchTextField=[[UITextField alloc]initWithFrame:CGRectMake(1.0,0.0,searchBackGroundImageView.frame.size.width, searchBackGroundImageView.frame.size.height)];//创建一个UITextField对象，及设置其位置及大小
//输入框中是否有个叉号，在什么时候显示，用于一次性删除输入框中的内容
searchTextField.clearButtonMode = UITextFieldViewModeAlways;
text.clearsOnBeginEditing = YES;//再次编辑就清空
searchTextField.contentVerticalAlignment=UIControlContentVerticalAlignmentCenter;//设置其输入内容竖直居中

UIImageView* imgV=[[UIImageViewalloc]initWithImage:[UIImageimageNamed:@"search_ico"]];

searchTextField.leftView=imgV;//设置输入框内左边的图标

[self.tf11setClearButtonMode:UITextFieldViewModeWhileEditing];//右侧删除按钮

searchTextField.leftViewMode=UITextFieldViewModeAlways;

searchTextField.placeholder=@"请输入关键字";//默认显示的字

searchTextField.secureTextEntry=YES;//设置成密码格式

searchTextField.keyboardType=UIKeyboardTypeDefault;//设置键盘类型为默认的

searchTextField.returnKeyType=UIReturnKeyDefault;//返回键的类型
/*
typedef enum {

    UIReturnKeyDefault, 默认 灰色按钮，标有Return

    UIReturnKeyGo,      标有Go的蓝色按钮

    UIReturnKeyGoogle,标有Google的蓝色按钮，用语搜索

    UIReturnKeyJoin,标有Join的蓝色按钮

    UIReturnKeyNext,标有Next的蓝色按钮

    UIReturnKeyRoute,标有Route的蓝色按钮

    UIReturnKeySearch,标有Search的蓝色按钮

    UIReturnKeySend,标有Send的蓝色按钮

    UIReturnKeyYahoo,标有Yahoo的蓝色按钮

    UIReturnKeyYahoo,标有Yahoo的蓝色按钮

    UIReturnKeyEmergencyCall, 紧急呼叫按钮

} UIReturnKeyType;
*/

searchTextField.delegate=self;//设置委托

textFied.adjustsFontSizeToFitWidth = YES;//设置为YES时文本会自动缩小以适应文本窗口大小.默认是保持原来大小,而让长文本滚动，与下边的配合使用

text.minimumFontSize = 20;//设置自动缩小显示的最小字体大小

text.autocapitalizationType = UITextAutocapitalizationTypeNone;//首字母是否大写
text.enablesReturnKeyAutomatically = YES;//输入框没有字的时候return变成灰色

```
#### 光标显示位置调整
设置leftView，同是设置以下两个方法：
```
searchTextField.leftView=imgV;//设置输入框内左边的图标

searchTextField.leftViewMode=UITextFieldViewModeAlways;

```
#### 设置textfiled的边框的颜色和样式
尽量使用layer的属性来设置

#### 设置光标颜色
```
//设置光标颜色1
self.tintColor = [UIColor redColor];
```
#### 设置占位文字颜色

```
// 设置光标的颜色2

UIColor *color = [UIColor whiteColor];  
    _userName.attributedPlaceholder = [[NSAttributedString alloc] initWithString:@"用户名" attributes:@{NSForegroundColorAttributeName: color}];  
```
### 各种小技巧

#### 监听删除
```
- (BOOL)textField:(UITextField*)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString*)string{

   NSString*match =nil;;
   if(textField ==_userTextField){
        match =@"^([0-9]){0,}$";
    }else{
        match =@"^([0-9a-zA-Z]|[\\s|-]){0,}$";
    }

   if(textField ==self.userTextField) {
       if(range.length==1) {
           self.pwdTextField.text=@"";
            [selfcheckInputValid];
        }
    }
}

在这个方法里边拿到range，如果length为0说明在正向输入，如果为1说明在删除字符
```
#### 密文变明文光标错位
```
思路：删除内容，然后重新赋值

- (void)pwdShowBtnClick:(UIButton*)btn{
    btn.selected= !(btn.selected);
   _pwdTextField.secureTextEntry= !(btn.selected);
   NSString*pwd =_pwdTextField.text;
   _pwdTextField.text=@"";
//    [_pwdTextField resignFirstResponder];
   _pwdTextField.text= pwd;
    [_pwdTextFieldbecomeFirstResponder];
}
```
#### 各种收起键盘
- 点击背景view的任何地方
- 点击return

```
[self.textField resignFirstResponder];

[self.view endEditing:YES];//最彻底的办法

-(BOOL)textFieldShouldReturn:(UITextField *)textField
{
    [textField resignFirstResponder];
    returnYES;
}
```
#### 虚拟键盘挡住textfield
- 设置inputAccessoryView，在这个上边添加另外一个textfield
- 监听键盘弹出事件，此时将firstresponder设置为该textfield

#### 给数字键盘添加取消或者其他按钮

```
@implementation ViewController
- (UITextField *)textField {
    if (!_textField) {
        _textField = [[UITextField alloc] init];
        _textField.frame = CGRectMake(100, 100, 200, 50);
        _textField.keyboardType = UIKeyboardTypeNumberPad;
        _textField.backgroundColor = [UIColor blueColor];
    }
    return _textField;
}
- (void)viewDidLoad {
    [super viewDidLoad];
   [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillShowOnDelay:) name:UIKeyboardWillShowNotification object:nil];
 self.littlecancleButton.backgroundColor = [UIColor clearColor];//这句主要是为了提前加载button，如果在键盘的通知中添加button
 ，就会自动加上动画
    [self.view addSubview:self.textField];
}

- (void)windowDidShow:(NSNotification *)noti {
    NSLog(@"------");
}

- (void)keyboardWillShowOnDelay:(NSNotification *)noti {

    UIWindow *keyboardWindow = nil;
    
    for (UIWindow *testWindow in [[UIApplication sharedApplication] windows]) {
        if ([NSStringFromClass([testWindow class]) isEqualToString:@"UIRemoteKeyboardWindow"]) {
            keyboardWindow = testWindow;
            break;
        }
    }
    
    if (!keyboardWindow) return;
    
    [keyboardWindow addSubview:self.littlecancleButton];
    [UIView animateWithDuration:0.1 animations:^{
        self.littlecancleButton.frame = CGRectMake(0, [UIScreen mainScreen].bounds.size.height - 53, [UIScreen mainScreen].bounds.size.width * 0.3333, 53);
    }];
   
}

- (UIButton *)littlecancleButton {
    if (!_littlecancleButton) {
        _littlecancleButton = [UIButton buttonWithType:UIButtonTypeCustom];
        [_littlecancleButton setTitle:@"取消" forState:UIControlStateNormal];
        _littlecancleButton.titleLabel.font = [UIFont systemFontOfSize:20];
        [_littlecancleButton sizeToFit];
        [_littlecancleButton setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
        _littlecancleButton.frame = CGRectMake(0, [UIScreen mainScreen].bounds.size.height + 216 - 53, [UIScreen mainScreen].bounds.size.width * 0.3333, 53);
        [_littlecancleButton addTarget:self action:@selector(cancleBtnClick) forControlEvents:UIControlEventTouchUpInside];

    }
    return _littlecancleButton;
}

- (void)cancleBtnClick {
    NSLog(@"1111111");
}

@end
```

#### 虚拟键盘遮挡提示弹框
解决思路同添加取消按钮，就是将提示框添加到键盘的window上

#### textfield随着键盘移动
```
- (void)viewDidLoad {
    [super viewDidLoad];
   
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(keyboardWillShow:) name:UIKeyboardWillChangeFrameNotification object:nil];
    
    [self.view addSubview:self.textField];
    
    UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tap)];
    [self.view addGestureRecognizer:tap];
}

- (void)tap {
    [self.view endEditing:YES];
}

- (void)keyboardWillShow:(NSNotification *)noti {
    double duration = [noti.userInfo[UIKeyboardAnimationDurationUserInfoKey] doubleValue];
    
    CGFloat keyboardY = [noti.userInfo[UIKeyboardFrameEndUserInfoKey] CGRectValue].origin.y;
    CGFloat ty = keyboardY - kScreenH;
    
    [UIView animateWithDuration:duration animations:^{
        self.textField.transform = CGAffineTransformMakeTranslation(0, ty);
    }];
}
```

### 配合使用正则表达式
- 校验是否为有效手机号
```
//是否为有效手机号的判断
- (NSString *)valiMobile:(NSString *)mobile{
if (mobile.length < 11)
{
return @"请填写有效的手机号";
}else{
/**
* 移动号段正则表达式
*/
NSString *CM_NUM = @"^((13[4-9])|(147)|(15[0-2,7-9])|(178)|(18[2-4,7-8]))\\d{8}|(1705)\\d{7}$";
/**
* 联通号段正则表达式
*/
NSString *CU_NUM = @"^((13[0-2])|(145)|(15[5-6])|(176)|(18[5,6]))\\d{8}|(1709)\\d{7}$";
/**
* 电信号段正则表达式
*/
NSString *CT_NUM = @"^((133)|(153)|(177)|(18[0,1,9]))\\d{8}$";
NSPredicate *pred1 = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CM_NUM];
BOOL isMatch1 = [pred1 evaluateWithObject:mobile];
NSPredicate *pred2 = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CU_NUM];
BOOL isMatch2 = [pred2 evaluateWithObject:mobile];
NSPredicate *pred3 = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CT_NUM];
BOOL isMatch3 = [pred3 evaluateWithObject:mobile];

if (isMatch1 || isMatch2 || isMatch3) {
return nil;
}else{
return @"请填写有效的手机号";
}
}
return nil;
}
```


