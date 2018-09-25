
---
title: UIViewController
date: 2018-09-25 23:51:53
category: iOS
---

    
# 判断当前ViewController是Push 模式还是Present模式的方法

判断当前ViewController在它的导航栈里面的位置，如果为0，则是push；如果大于0，则是present。

```objective-c
NSInteger index = [self.navigationController.viewControllers indexOfObject:self];
if (index == 0) {
    NSLog(@"Present");
}else{
    NSLog(@"Push");
}
```