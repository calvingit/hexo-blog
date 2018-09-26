---
title: CALayer.md
date: 2018-09-25 23:51:53
category: iOS
---

# CALayer 不支持 Autolayout 的问题

iOS 的 CALayer 到目前为止不支持 AutoLayout 也不支持 autoresizingMask。

除了在每次 layoutSubview 的时候改变 layer 的 frame，还可以用 UIView 包装一下：

```objective-c
@implementation BackgroundView

+(Class)layerClass{
    return [CAGradientLayer class];
}

@end
```

使用的时候可以像这样：

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    self.backgroundview = [[BackgrundView alloc] initWithFrame:self.view.frame];
    self.backgroundview.autoresizingMask = UIViewAutoresizingFlexibleHeight | UIViewAutoresizingFlexibleWidth;
    [self.view addSubview:self.backgroundview];
    [self setupCAGradientLayer:(CAGradientLayer*)self.backgroundview.layer];
}
```
