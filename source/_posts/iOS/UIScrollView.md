---
title: UIScrollView
date: 2018-09-25 23:51:53
category: iOS
---

# 修改偏移，但是不想触发

旋转之后需要重新设置 contentOffset，但是设置 scrollView 的 contentOffset 会触发 scrollViewDidScroll 调用。这里使用修改 scrollView 的 bounds 来达到目的，但是可以不触发 scrollViewDidScroll 回调。

```
    CGRect bounds = self.scrollView.bounds;
    bounds.origin.x = self.segmentedControl.selectedSegmentIndex * self.view.frame.size.width;
    self.scrollView.bounds = bounds;
```

<!-- more -->

# 旋转时触发了 scrollViewDidScroll

问题描述：

> 假如有三页内容，每一页全屏显示，向左滑动一下滚动一页。当在第一页、第二页时，旋转横竖屏，没有任何问题。但是在第三页时，旋转之后就会触发 scrollViewDidScroll 回调。如果此时在 scrollViewDidScroll 里面计算了 contentOffset，会得到不是想要的结果，使得第三页内容再次滚动到第二页。

解决办法：**将 scrollView 的 delegate 设为 nil，就可以不调用 scrollViewDidScroll，之后再恢复回来**。

```
- (void)viewWillLayoutSubviews{
    [super viewWillLayoutSubviews];
    self.scrollView.delegate = nil;
}

- (void)viewDidLayoutSubviews{
    [super viewDidLayoutSubviews];
    self.scrollView.delegate = self;
}
```
