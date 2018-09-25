
---
title: Xcode 技巧
date: 2018-09-25 23:51:53
category: Softwares
---

# 禁用系统的无效Log输出

在 `Scheme`—`Arguments`—`Environment Variables`，添加`Name`为`OS_ACTIVITY_MODE`，`Value`设为`Disable`。


# 打印App的加载时长

包括整体加载时长，动态库加载时长等

在 `Scheme`—`Arguments`—`Environment Variables`，添加`Name`为`DYLD_PRINT_STATISTICS`，`Value`设为`YES`。

# 自动获取Git的提交次数当做Build

在`Build Phases`添加脚本:

```bash
REV=`git rev-list HEAD | wc -l | awk '{print $1}'`
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${REV}" "${TARGET_BUILD_DIR}"/${INFOPLIST_PATH}
```

# "selector not recognized" 分析

运行时有可能"selector not recognized"，主要是少了如下几类配置：

| Flags       | 位置                                       | 作用                                                                                                                                                      |
| ----------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -ObjC       | Other Linker Flags                         | 链接静态库中所有的Objective-C代码到App，库里面有分类的时候使用                                                                                            |
| -all_load   | Other Linker Flags                         | 链接静态库中所有的代码到App中                                                                                                                             |
| -force_load | Other Linker Flags                         | 链接指定静态库中所有的代码到App中                                                                                                                         |
| YES         | Perform Single-Object Prelink (静态库使用) | 静态库中所有的对象文件合并成单一文件，只要这个单一文件有代码被使用，所有代码都被链接到App中                                                               |
| 伪符号      | 源文件                                     | 在有Category的方法中添加一些伪符号，如空函数体的函数、可被访问的全局变量等。在需要使用category的方法前引用这些伪符号，这些伪符号所在的代码会被链接到App中 |

> 和 "-ObjC"作用类似的有以上的五种方案。可以看出，从增加APP代码体积来看，伪符号方案增加得最少"Perform Single-Object Prelink"、 "-force_load" 和 "–ObjC" 次之，"-all_load" 增加得最多。在开发iOS SDK时，为了方便使用者手动集成，最好是减少使用者需要配置的信息，所以"伪符号"方案和 "Perform Single-Object Prelink"方案是推荐的。另外，第三方SDK常常是闭源的，对于使用者来说，伪符号是透明的，所以从简便性角度看，推荐"Perform Single-Object Prelink"方案。但是"Perform Single-Object Prelink"不能用于巨量源码文件，比如2000个文件的时候，会导致编译失败。


# 快速定位方法的使用者
除了全局搜索之外还可以用以下方法定位：双击选中方法名， 使用快捷键`Control + 1`，在弹出的菜单中选中`caller`，如图

![](http://owzot3i4n.bkt.clouddn.com/121155.jpg)

# OC项目新建文件时增加类名前缀
点击工程的`Target`，在右边的菜单栏`Project Document`中添加`Class Prefix`，如图：

![](http://owzot3i4n.bkt.clouddn.com/121851.jpg)

# Open Quickly
 使用快捷键`Command + Shift + O`，可以快速打开源码文件、API文档，支持模糊搜索变量、方法。

![](http://owzot3i4n.bkt.clouddn.com/122006.jpg)

# 获取Swift的每个函数的编译时间

在**Build Settings**的**Other Swift Flags**添加`-Xfrontend -debug-time-function-bodies`

![](http://owzot3i4n.bkt.clouddn.com/123129.jpg)

# 检查内存泄漏时开启堆栈日志

有时候仅仅打开`Debug Memory Graph`进行内存泄漏调试时，有很多紫色的感叹号提示，但是信息不明确，只有一个内存地址，这时可以在`Scheme`里面打开 `Melloc Stack`：
![](https://ws1.sinaimg.cn/large/0069RVTdgy1fu2eawj593j30ow0e0q50.jpg)

然后选中其中一个内存泄漏的地址，在右边的`Backtrace`里面有调用栈信息。