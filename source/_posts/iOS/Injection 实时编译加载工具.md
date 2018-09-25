
---
title: Injection 实时编译加载工具
date: 2018-09-25 23:51:53
category: iOS
---

    
# Injection 实时编译加载工具

[Injection](http://johnholdsworth.com/injection.html) 是一个Xcode的注入插件，独立的Mac App。关键功能是实时编译加载，源文件的修改立即反应到界面上。因为它不需要编译整个工程，只是当前的`ViewController`，所以它有相当于写网页界面一样快的速度。

## 基本使用方法

1. 到[官网](http://johnholdsworth.com/injection.html)下载App，已经支持Xcode 9版本。

2. 运行App。

3. 在工程里添加一个`UIViewController`的分类/扩展，增加一个`injected`方法：

   Objective-C 代码：

   ```objective-c
   #import "UIViewController+Inject.h"

   @implementation UIViewController (Inject)

   - (void)injected{
   #if DEBUG
       for (UIView *subView in self.view.subviews) {
           [subView removeFromSuperview];
       }
       [self viewDidLoad];
   #endif
   }
   ```

4. 编译并运行你的项目

5. 修改部分视图代码，比如添加一个按钮，然后保存文件

6. 使用快捷键`Ctrl` + `=`进行代码注入，模拟器会立即看到效果。

7. 开启`File Watcher`之后，只需要运行模拟器之后，启动一次注入即可，以后每次保存文件时会自动检查修改内容进行编译更新。



## 改进方法

上面的注入方法只能用于源码添加子View的修改，不支持Storyboard、Xib方法添加的UI。

因为Storyboard、Xib里面设置的子View在`viewDidLoad`之前就已经添加了，`injected`方法把子View都移除了，所以也就看不到那些子View了。但我们可以改成：不注入`viewDidLoad`，将添加子view和设置UI分开，只注入设置view。


`UIViewController+Inject.m` 代码：

```objective-c
#import "UIViewController+Inject.h"

@implementation UIViewController (Inject)

- (void)injected{
#if DEBUG
    [self configUI];
#endif
}

- (void)configUI{
    NSAssert(YES, @"子类需要覆盖重写-(void)configUI，在这里配置UI");
}

@end
```

`ViewController.m` 代码：

```objective-c
#import "ViewController.h"

@interface ViewController ()
@property (weak, nonatomic) IBOutlet UILabel *label;
@property (weak, nonatomic) IBOutlet NSLayoutConstraint *leading;

@end

@implementation ViewController
{
    UIButton *button;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    [self addSubviews];
    [self configUI];
}

- (void)addSubviews{
    button = [UIButton buttonWithType:UIButtonTypeCustom];
    [self.view addSubview:button];
}

- (void)configUI{
    self.label.text = @"1234457777";
    self.view.backgroundColor = [UIColor blueColor];
    self.leading.constant = 30;
    button.frame = CGRectMake(10, 100, 100, 100);
    [button setTitle:@"测试按8" forState:UIControlStateNormal];
    [button setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];
    button.backgroundColor = [UIColor greenColor];
}

@end
```


**PS：如果是Storyboard或Xib添加的子view，这里需要注意：不能随意移除子视图。**

假如你调用`removeFromSuperview`移除`label`：

```objective-c
[self.label removeFromSuperview];
```
那么，因为`IBOutlet`一般是`weak`属性，`label`会被释放掉，等你再次添加`label`时：

```objective-c
[self.view addSubview:self.label];
```
这是看不到效果的，因为`label`为`nil`。即使你把`IBOutlet`设置为`strong`属性，能看到`label`再次显示在界面上，但`label`的约束是没有的，它会位于左上角。

## 演示

![injection](https://user-images.githubusercontent.com/2109371/34648167-f503e434-f3cf-11e7-8368-a86ea572133c.gif)


## 缺点

只能用于模拟器，不能用于真机，因为iOS 10之后，iPhone的沙盒机制改了。