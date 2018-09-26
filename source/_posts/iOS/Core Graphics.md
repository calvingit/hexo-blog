---
title: Core Graphics
date: 2018-09-25 23:51:53
category: iOS
---

# 画虚线

使用到函数：`CGContextSetLineDash`

此函数需要四个参数：

- context – 这个不用多说
- phase - 表示在第一个虚线绘制的时候跳过多少个点
- lengths – 指明虚线是如何交替绘制，具体看例子
- count – lengths 数组的长度

  <!-- more -->

例如：

```
//表示先绘制10个点，再跳过10个点，如此反复
CGFloat lengths[] = {10,10};
CGContextSetLineDash(context, 0, lengths,2);
```
