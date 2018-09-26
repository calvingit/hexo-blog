---
title: ReactiveCocoa 使用要点
date: 2018-09-25 23:51:53
category: iOS
---

现在改名为[ReactiveObjC](https://github.com/ReactiveCocoa/ReactiveObjC)。

# 代替 delegate

使用`rac_signalForSelector: fromProtocol:`代替 delegate 的方法时,切记要将 self 设为代理这句话放在订阅代理信号的后面写,否则会无法执行。

<!-- more -->

```objective-c
  //这里订阅收到的是一个x,当一个页面存在多个tableview时,我们可以对x进行判断看是哪个tableview
  [[self rac_signalForSelector:@selector(tableView:didSelectRowAtIndexPath:) fromProtocol:@protocol(UITableViewDelegate) ] subscribeNext:^(RACTuple * x) {
      NSLog(@"x包含了所有参数");
  }];

  //必须将代理最后设置,否则信号是无法订阅到的
  //在设置代理的时候，系统会缓存这个代理对象实现了哪些代码方法
  //如果将代理放在订阅信号前设置,那么当控制器成为代理时是无法缓存这个代理对象实现了哪些代码方法的
  tableview.delegate = self;
```

如果在`viewDidLoad`之前就已经设置了`delegate`，后面再次设置是无效的，比如在 storyboard 或者 xib 里面绑定了`delegate`，上面的代码:

```objective-c
tableview.delegate = self;
```

这句话是没什么作用的，除非你先通过`tableview.delegate = nil`释放掉 delegate，再次设置。

# 代替 Notification

一般需要使用`takeUntil:self.rac_willDeallocSignal`来对通知进行释放掉。

```objective-c
  //takeUntil会接收一个signal,当signal触发后会把之前的信号释放掉
  [[[[NSNotificationCenter defaultCenter] rac_addObserverForName:UIKeyboardDidShowNotification object:nil] takeUntil:self.rac_willDeallocSignal] subscribeNext:^(id x) {

  }];
```
