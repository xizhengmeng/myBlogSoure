---
title: iOS工程小知识
date: 2017-02-26 23:02:06
tags:
---
### 1:出现（ linker command failed with exit code 1）
如果具体的错误是这个与第三方.a库重复，那么要更改的是，类名和全局变量的名字
<!--more-->
### 2.SEGV_ACCERR错误
说明对象被过度释放，查看是否有在=nil之后又使用了该对象，比如说这段代码
case MessageComposeResultSent:
        {
            //信息传送成功
//            [JRMsgShow showMsg:@"发送成功"];
            if (self.closeBlock) {
                self.closeBlock();
            }
            if (self.messageSuccessBlock) {
                self.messageSuccessBlock();
            }
            [[UIApplication sharedApplication].keyWindow.rootViewController dismissViewControllerAnimated:YES completion:nil];
        }

很明显前边我们使用了closeblock，这个block之行之后，很快self会死掉，而这个之后我们又使用了self.messageSuccessBlock这样再次使用self，必然导致崩溃，不过这是在iOS8上面，在iOS9和10上就没有这个问题，说明苹果做了系统级的优化

### 3.参数传递尽量要用model，哪怕是回调，因为可能后边会增加需求，那么参数传递将会变得很恶心
