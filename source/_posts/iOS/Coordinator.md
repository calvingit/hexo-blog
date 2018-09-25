
---
title: Coordinator
date: 2018-09-25 23:51:53
category: iOS
---

    
# 背景
早在2015年初，[Soroush Khanlou](http://khanlou.com)提出来这个概念，[原文在此](http://khanlou.com/2015/01/the-coordinator/)。同年，他参加了***NSSpain*** 演讲，主题就是讲`Coordinator`。然后他在自己的博客中重新梳理了一遍，写下了这篇流传很广的[文章](http://khanlou.com/2015/10/coordinators-redux/)。


那为什么会提出这种模式呢？


苹果提倡 MVC 模式来组织代码，但每一个 iOS 开发人员应该都已经体会过了 **Massive View Controller** 的痛楚。所以，现在开发一些稍微大点的App，都会增加一层 **View Model**或者 **Presenter** 来分担 **View Controller**的压力。

但是，即使用了 **MVVM/MVP** 模式，转移了大部分的业务逻辑在 **View Model** 处理， App 依然存在下面三个问题：

### 1.超载的App Delegate
**App Delegate**是App的入口，它的主要责任应该是从操作系统传递消息给App的子系统。但是，正因为


### 2.减负之后依然责任重大的 View Controller

### 3.耦合的导航

`Coordinator`是一种代码组织模式，没有一个标准的库或框架来做基础。