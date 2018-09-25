---
title: iOS 应用版本号比较
date: 2018-09-25 23:51:53
category: iOS
---

```
NSDictionary *infoDict = [[NSBundle mainBundle] infoDictionary];
NSString *vison = infoDict[@"CFBundleVersion"];
//    NSString *vison = @"1.2.5";
NSString *vison2 = @"1.13.4";
if ([vison compare:vison2 options:NSNumericSearch] == NSOrderedDescending)
{
    NSLog(@"%@ is bigger",vison);
} else {
    NSLog(@"%@ is bigger",vison2);
}
```
