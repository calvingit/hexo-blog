---
title: Carthage
date: 2018-09-25 23:51:53
category: Softwares
---

[Carthage](https://github.com/Carthage/Carthage) 是个 iOS、Mac 开发中的一个第三方库管理工具，类似于 CocoaPods 的作用，但是又有很大的不同之处，最大的一点就是：Carthage 不是集中式的库管理，对 Xcode 工程没有侵入性。因为 Carthage 编译之后是动态 framework，拖到 Xcode 工程里面就可以用了。相对于 CocoaPods 来说，吸引我的就是减少编译时间，因为很多第三方库我不需要去定制开发，每次 clean 之后 CocoaPods 从源码编译的这部分时间就可以节省下来了。

<!-- more -->

# 添加 Framework 步骤

## 添加 Cartfile 文件

类似 Cocoapods 的 Podfile，指定你需要添加的库，目前支持 git 和 github，栗子如下

```
# 必须最低 2.3.1 版本
github "ReactiveCocoa/ReactiveCocoa" >= 2.3.1

# 必须 1.x 版本
github "ReactiveCocoa/ReactiveCocoa" ~> 1.0# (大于或等于 1.0 ，小于 2.0)

# 必须 0.4.1 版本
github "jReactiveCocoa/ReactiveCocoa" == 0.4.1

# 使用最新的版本
github "ReactiveCocoa/ReactiveCocoa"

# 使用一个私有项目，在 "development" 分支
git "https://enterprise.local/desktop/git-error-translations.git" "development"

# 使用本机的Git项目
git "file:///Users/zhangwen/Desktop/work/sources/XLForm" "develop"
```

### 运行`carthage update`命令

这条命令会自动 clone 代码，编译，生成 framework 文件，存放在`Carthage/build/iOS`目录。
还会创建一个`Cartfile.resolved`文件,这个文件是生成后的依赖关系，不能修改。

这条命令会创建多个平台的 framework，比如 macOS, tvOS, iOS 等。如果只需要 iOS 的 framework，使用下面的命令：

```
carthage update --platform ios
```

### 添加 Framework

进入 Xcode 的 General 选项卡，在最下方的 Linked Frameworks and Libraries 中，将 Carthage/Build/iOS 中的 framework 文件添加到项目中

PS:还有人说如下方法，但试过好像没用，不知道是不是因为用了 swift 的 framework

> 在项目中引入依赖的 Framkework，只需要在对应 Target 中的 Build Setting 中的 Framework Search Path 项加入以下路径，Xcode 便会自动搜索目录下的 Framework：
> `$(SRCROOT)/Carthage/Build/iOS`

### 添加脚本

在 Build Phrases 中，点击左上角的 + 号，添加一个 New Run Script Phrase:

```
/usr/local/bin/carthage copy-frameworks
```

然后在`Input Files`添加 framework 的路径，例如

```
$(SRCROOT)/Carthage/Build/iOS/SwiftyJSON.framework
```

PS: 如果不添加这个 copy-frameworks 脚本，那么项目在运行的时候会因为找不到这个动态库而在启动的时候崩溃。

## 注意事项

- OC 项目中使用到了 Swift 的代码

> 需要在`Build Settings`里面的 `Always Embedded Swift Libraries`选择为 YES，默认是 NO.

- 改了 Cartfile 里面的版本号但是更新不了新版本

> 比如：原先是`github "ReactiveCocoa/ReactiveCocoa"`的`2.3`版本, 想要更新版本`2.4`，update 操作总是原来的`2.3`，那就改成分支模式，指定 master 就可以了`github "ReactiveCocoa/ReactiveCocoa "master"`

- 当第三方库存在依赖时，carthage 处理不了，比如`Charts`里面有个`ChartsRealm`依赖`realm`，如果 Cartfile 本身已经有`realm`了，那么`realm`的版本就不确定是哪个了

- 清空缓存目录

> `Carthage/Checkouts`目录下的并不是 git 管理，只是复制了缓存目录的对应版本文件
> 缓存目录在`~/Library/Caches/org.carthage.CarthageKit/dependencies`，里面的就是`git clone`的文件。

## 为你的 framework 支持 Carthage

- 将 framework target 的 scheme 的`shared`属性勾选
- 用命令`carthage build --no-skip-current`测试是否 OK，如果没有 error，基本上就可以了
