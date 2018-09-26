---
title: UIViewController
date: 2018-09-25 23:51:53
category: iOS
---

# 判断当前 ViewController 是 Push 模式还是 Present 模式的方法

判断当前 ViewController 在它的导航栈里面的位置，如果为 0，则是 push；如果大于 0，则是 present。

```objective-c
NSInteger index = [self.navigationController.viewControllers indexOfObject:self];
if (index == 0) {
    NSLog(@"Present");
}else{
    NSLog(@"Push");
}
```

<!-- more -->
