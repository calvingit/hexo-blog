---
title: UIDatePicker
date: 2018-09-25 23:51:53
category: iOS
---

# 修改文本颜色

```objective-c
//设置白色文本
[self.datePicker setValue:[UIColor whiteColor] forKeyPath:@"textColor"];
SEL selector = NSSelectorFromString(@"setHighlightsToday:");
NSInvocation *invocation = [NSInvocation invocationWithMethodSignature :
                            [UIDatePicker
                             instanceMethodSignatureForSelector:selector]];
BOOL no = NO;
[invocation setSelector:selector];
[invocation setArgument:&no atIndex:2];
[invocation invokeWithTarget:self.datePicker];
```

<!-- more -->

# 修改分割线颜色

需要注意分割线的高度是 0.5，不是 1

```objective-c
// 设置为白色
UIView * pickerView = _datePicker.subviews.firstObject;
if (pickerView) {
    for (UIView *subView in pickerView.subviews ){
        if (subView.frame.size.height <= 1) {
            subView.backgroundColor = [UIColor whiteColor];
        }
    }
}
```
