---
title: 常用 Xcode 扩展
date: 2018-09-25 23:51:53
category: iOS
---

# 常用 Xcode 扩展

收集一些好用的 Xcode 扩展，提高开发效率。

## Xcode 8+ 启用扩展

将下载的 app 拖到`Applications`之后，打开运行一遍，然后在设置——扩展——Xcode Source Editor 里面勾选。如果 Xcode Source Editor 不出现，来回点击**全部**/**操作**等，相当于刷新一下。

![](https://i.loli.net/2018/01/06/5a50c2fd5aa7d.jpg)

随便打开一个 Xcode 工程，在`Editor`菜单下就可以看到你启用的扩展了。PS：只是打开 Xcode 没有用，因为没有用到编辑器。

![](https://i.loli.net/2018/01/07/5a51b180493c0.jpg)

## 格式化工具

- [Snowonder](https://github.com/Karetski/Snowonder/releases)

格式化头文件的导入代码，默认将 framework 的头文件放在最上面，其他按字母顺序排序。

![](https://raw.githubusercontent.com/Karetski/Snowonder/master/Resources/ReadmeHeader.png)

- [XAlign](https://github.com/qfish/XAlign)

`#define`、属性、变量等代码对齐，可以自定义对齐方式。

![](https://camo.githubusercontent.com/14aada28ed13ccf89beb8772de6dc1ef55c914ad/687474703a2f2f7166692e73682f58416c69676e2f696d616765732f70726f70657274792e676966)

![](https://camo.githubusercontent.com/f61bfc31e144ad6a9d7ca26fa19547a3af5da8c6/687474703a2f2f7166692e73682f58416c69676e2f696d616765732f646566696e652e676966)

![](https://camo.githubusercontent.com/7973c0e352b1f91e3efe5b3550cff5df97f4589a/687474703a2f2f7166692e73682f58416c69676e2f696d616765732f657175616c2e676966)

- [Swimat](https://github.com/Jintin/Swimat)

Swift 代码格式化，安装`brew cask install swimat`

![](https://github.com/Jintin/Swimat/raw/master/README/preview.gif)

- [Clean Closure](https://github.com/BalestraPatrick/CleanClosureXcode)

将 Swift 闭包里面的括号去掉，需要自己编译打包。

![](https://github.com/BalestraPatrick/CleanClosureXcode/raw/master/result.gif)

## 代码生成工具

- [quicktype](https://github.com/quicktype/quicktype-xcode/)

复制 json 数据，粘贴时生成支持 Codable 的 struct，支持 Swift，C++，Obj-C++，Java 等。还有自己的[网站](https://app.quicktype.io/#l=swift)。

- [SwiftInitializerGenerator](https://github.com/Bouke/SwiftInitializerGenerator)

根据选择的代码生成初始化方法。
![](https://github.com/Bouke/SwiftInitializerGenerator/raw/master/docs/demo.gif)

- [SwitchCaseGenerator](https://github.com/timaktimak/SwitchCaseGenerator)

自动生成 switch 的每个 case。
![](https://github.com/timaktimak/SwitchCaseGenerator/raw/master/Assets/Example.gif)

- [CodeGenerator](https://github.com/WANGjieJacques/CodeGenerator/)

  代码生成工具，有很多功能：

  1. 根据`protocol`生成 mock 数据
  2. 生成`Equatable`代码
  3. 生成`Hashable`代码
  4. 生成`Init`代码
  5. 生成`NSCoding`代码

## 其他工具

- [XcodeWay](https://github.com/onmyway133/XcodeWay)

  项目相关的目录导航，比如打开工程目录，在 iTerm 打开工程目录，打开 DerivedData 目录，Provisioning Profiles 目录等

- [Linex](https://github.com/kaunteya/Linex)

  有三个扩展：`Line`，`Selection`，`Convert`，分别是行操作，选择操作，转换操作。

  - Line：复制当前行，删除当前行，在当前行下面/上面另起一行，合并行，等
  - Selection：选择单词，选择行，对齐
  - Convert：数字增减 1，bool 变量取反
