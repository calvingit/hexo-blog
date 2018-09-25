
---
title: UIBarButtonItem
date: 2018-09-25 23:51:53
category: iOS
---

    
# 自定义CustomView时大小约束添加

```objective-c
JCNavigationItemView *itemView = [[JCNavigationItemView alloc] init];
NSLayoutConstraint * heightConstraint = [NSLayoutConstraint constraintWithItem:itemView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1 constant:24];
NSLayoutConstraint * widthConstraint = [NSLayoutConstraint constraintWithItem:itemView attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1 constant:58];
[itemView addConstraints:@[heightConstraint, widthConstraint]];

UIBarButtonItem *item = [[UIBarButtonItem alloc] initWithCustomView:itemView];
self.navigationItem.rightBarButtonItem = item;
```