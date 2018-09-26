---
title: iOS 应用版本号比较
date: 2018-09-25 23:51:53
category: iOS
---

```objective-c
NSDictionary *infoDict = [[NSBundle mainBundle] infoDictionary];
// App 版本
NSString *version = infoDict[@"CFBundleVersion"];
NSString *certainVersion = @"1.13.4";
if ([version compare:certainVersion options:NSNumericSearch] == NSOrderedDescending)
{
    NSLog(@"%@ is bigger",version);
} else {
    NSLog(@"%@ is bigger",certainVersion);
}
```
