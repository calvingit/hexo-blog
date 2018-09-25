
---
title: 【学习】Coordinator 架构模式
date: 2018-09-25 23:51:53
category: iOS
---

    
    > 作者是[Soroush Khanlou](http://khanlou.com)，阅读[原文](http://khanlou.com/2015/10/coordinators-redux/)。

# Coordinator 架构模式

2015年1月，[Soroush Khanlou](http://khanlou.com)提出来[Coordinator](http://khanlou.com/2015/01/the-coordinator/)这个概念。同年10月份，他参加了NSSpain演讲，主题就是讲`Coordinator`架构，可以在线观看演讲的[PPT](https://www.slideshare.net/secret/3jJlEE1weo0RRl)和[视频](https://vimeo.com/144116310)。

## 三个问题

### 超载的App Delegate
在指导我们把代码放在正确的地方这方面，Apple几乎没做什么事，完全由开发者自己去决定如何组织app。很显然地，首选的地方就是app的delegate。

**App Delegate**是App的入口，它的主要责任应该是从操作系统传递消息给App的子系统。不幸的是，正因为它处在所有东西的中心位置，所以就极其容易把很多东西放在这里。受这种策略影响的情况之一就是**Root**视图控制器的设置。如果这个**Root**视图控制器又是一个 `UITabBarController`的话，那就得设置`UITabBarController`的所有子视图控制器，而，app delegate就是一个好地方。

我做的第一个app里，我把**Root**视图控制器的设置都放在app delegate中（我多刚开始做iOS开发的人都和我一样）。但这些代码真的不属于那里，这么做只是为了方便。

做完第一个app之后，我就意识到了问题所在，我变聪明了。我耍了个花招：

```objective-c
@interface SKTabBarController : UITabBarController  
```
我创建了一个子类，继承于`UITabBarController`，然后把我的初始化代码放在这里。对这个问题而言，它是一个临时补丁，但基本上，它也不是这个代码的正确位置。我建议，我们从责任的角度上来看待**Root**视图控制器。管理子视图控制器是它的责任，但是分配和设置就不是了。我们创建一个东西的子类，只是为了把那些无法安放的代码隐藏起来，但它原本就不需要被子类化。

对这个App的设置逻辑来说，**Coordinator** 是更好的地方。

### 责任太多

这是另一个令人混淆的问题。如同把大量的责任归到app delegate身上一样，每个视图控制器同样会受到这样的影响。

目前视图控制器要负责以下东西：

- Model-View 绑定
- 子类创建
- 获取数据
- 布局
- 数据转换
- 导航流
- 用户输入
- 模型变化
- 等等其他

在"[8种模式帮你消灭Massive View Controller](http://khanlou.com/2014/09/8-patterns-to-help-you-destroy-massive-view-controller/)"这篇文章里，我提出一些方法来把这些责任放在每个视图控制器的子模块中。决定不能把所有的责任放在一个地方，否则我们就得处理3000行代码的视图控制器。

但究竟哪些东西要放在这个类？其他的东西该放在什么地方？视图控制器的职责是什么？这些问题还没有一个清晰的答案。

我很喜欢Graham Lee说过的话：

> 当你太过于看重MVC时，你会看着你创建的每一个类，然后问一个问题，“这是一个模型，一个视图，还是一个控制器？” 因为这个问题没有任何意义，答案也没有意义：任何不明显的数据或者明显的图形都被放入无定形的“控制器”集合中，最终将你的整个代码库吸入内部，就像一个黑洞在自身的重量下崩溃一样。

上面这些责任，哪些是在控制器层的？Graham 说得对，这个问题没有什么意义。我们陷进了“控制”这个可怕的单词里面，所以我们必须非常小心让视图控制器所做的事。否则，那就是一个黑洞。

**Coordinator**能帮我们解决这个问题。

### 平滑的导航流

我想讨论的最后一个问题是导航流。

```objective-c
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {  
	id object = [self.dataSource objectAtIndexPath:indexPath];  
	SKDetailViewController *detailViewController = [[SKDetailViewController alloc] initWithDetailObject:object];  
	[self.navigationController pushViewController:detailViewController animated:YES];  
}  
```

这是一段非常普通的代码，我相信它也在苹果的模板代码里面。不幸的是，这些是垃圾代码。我们一行行地来看：

```objective-c
id object = [self.dataSource objectAtIndexPath:indexPath];
```

第1行代码没什么问题。`dataSource`在逻辑上属于视图控制器，我们向它请求一个对象。

```objective-c
SKDetailViewController *detailViewController = [[SKDetailViewController alloc] initWithDetailObject:object]; 
```

这就是事情开始变得有点棘手的地方。视图控制器正在实例化并配置一个新视图控制器，它处于导航链的下一个位置。它“知道”接下来会发生什么，也知道要配置什么。这个视图控制器知道太多app里面的细节了。

```objective-c
[self.navigationController pushViewController:detailViewController animated:YES]; 
```

第三行代码完全脱离轨道了。这个视图控制器现在正“抓”着父节点，发出明确的消息给它，告诉它要干什么。它这是对父节点发号施令。现实生活中，孩子绝不应该使唤他们的父母。在编程世界里，我认为孩子们不应该知道他们的父母是谁！

在"[8种模式帮你消灭Massive View Controller](http://khanlou.com/2014/09/8-patterns-to-help-you-destroy-massive-view-controller/)"文章里，我建议使用`Navigator`类型，它能注入到包含着导航逻辑的视图控制器里。如果你只需要在一个地方使用，`Navigator`是个很好的解决方案，但是我们很快就会遇到一个`Navigator`无法帮我解决的问题。

这三行代码负责了很多东西，但并不是只有这个视图控制器才这样做。

假设你有一个照片编辑App，导航流如下：

> 选择照片`PhotoSelectionViewController` >>拉伸 `StraighteningViewController` >> 滤镜`FilteringViewController` >> 字幕`CaptioningViewController`

现在你的导航流在三个不同的对象之间传播。更进一步，`PhotoSelectionViewController`是被present出来的，但是dimiss操作必须在`CaptioningViewController`处理。

传递`Navigator`的方式只会让这些视图控制器都耦合在一个链条里，并没有解决每个视图控制器都知道链条里的下一个对象的问题。

`Coordinator`可以帮我们解决这个问题。

## 库 VS 框架

我认为，Apple想要我们都用这些方式来写代码。Apple希望我们让视图控制器成为世界的中心，因为所有的app都以相同的风格编写，让它们在SDK变化中发挥最大的影响。不幸的是，对开发者而言，这并不总是最好的。我们得负责app以后的维护，所以可靠的设计和可延展性的代码对我们来说具有更高的优先级。

有人说库和框架的区别就是：你调用库，框架调用你。我希望将第三方的依赖关系尽可能地看作是库。

当使用`UIKit`，你并没有掌控它。你调用`-pushViewController:animated:`，它做了很多工作，在未来的某个不确定时间里，它调用了下一个视图控制器的`-viewDidLoad:`，这里你可以做更多的事情。与其让`UIKit`来决定你的代码什么时候运行，不如尽可能早地脱离`UIKit`的领地，这样你就可以完全控制你的代码是如何流动的。

我曾经认为视图控制器是app里面最高级别的东西，知道如何运行整个演出。但我开始想，把它翻转一下会是什么样子。视图对它的视图控制器来说是透明的，受视图控制器的指挥。如果用同样的方式，我们让视图控制器变成另一件透明的东西呢?

## Coordinators

### 什么是 coordinator ？

coordinator是一个管理一个或多个视图控制器的对象。从视图控制器中拿走所有的驱动逻辑，然后把这些东西移动到上一层，这将会让你的生活变得更美好。

所有的东西开始于**app coordinator**，它解决了app delegate的超载问题。app delegate保存**app coordinator**，然后启动它。接着，**app coordinator**会设置app的 root 视图控制器。你可以在书中找到这种模式，比如[《企业应用架构模式》](https://book.douban.com/subject/4826290/)。他们把它叫做[应用控制器](http://martinfowler.com/eaaCatalog/applicationController.html)，**app coordinator**是一个专门为iOS而做的特殊版本。**app coordinator** 可以创建并配置视图控制器，也可以生成新的子coordinator来执行子任务。

coordinator从视图控制器接管了那些责任呢？主要来说有，导航和模型变化。这里说的模型变化指的是，将用户的修改保存到数据库，或者向API发起一个`PUT`或者`POST`请求，以及任何可以破坏用户数据的内容。

当我们从一个视图控制器中取出这些任务时，我们会得到一个惰性的视图控制器。它可以被present，它可以获取数据，转换之后并显示它，但关键是不能改变数据。我们现在知道，任何时候我们present一个视图控制器，它不会把东西拽在自己手里。每当它需要让我们知道某个事件或用户输入时，它就会使用delegate方法。让我们看一看代码示例。

### 代码示例

让我们从app delegate开始。

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {  
	self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];  
	self.rootViewController = [[UINavigationController alloc] init];
	
	self.appCoordinator = [[SKAppCoordinator alloc] initWithNavigationController:self.rootViewController];  
	[self.appCoordinator start];
	
	[self.window makeKeyAndVisible];  
}
```

app delegate初始化window和root视图控制器，然后启动app coordinator。coordinator的初始化与启动工作分离，这可以让我们随意创建它（懒加载等），然后在需要的时候启动它。

coordinator就是个简单的`NSObject`：

```objective-c
@interface SKAppCoordinator : NSObject 
```

这就很好了，没啥黑科技在里面。`UIViewController`类有几千行代码，我们调用它的方法时并不知道实际会发生什么，因为它是闭源。让运行我们app的对象成为简单的NSObject类型，会让一切变得更简单。

app coordinator初始化它需要的数据，其中包括root视图控制器。

```objective-c
- (instancetype)initWithNavigationController:(UINavigationController *)navigationController {  
	self = [super init];  
	if (!self) return nil;
	
	_navigationController = navigationController;
	
	return self;  
}  
```

一旦我们调用`start`方法，coordinator就开始工作：

```objective-c
- (void)start {  
	if ([self isLoggedIn]) {  
		[self showContent];  
	} else {  
		[self showAuthentication];  
	}  
} 
```

从一开始，coordinator 就在做决定。以前，像这样的逻辑没有一个很好的地方。你可以把它放在视图控制器或app delegate中，但两者都有缺陷：如果在视图控制器中，你会让它做出一些超出其目的的决定；如果在app delegate中，你就用一些它不关心的东西污染了它。

我们来看一下`showAuthentication`方法，这里，我们的基础coordinator会生成很多子coordinator来工作和执行子任务。

```objective-c
- (void)showAuthentication {  
	SKAuthenticationCoordinator *authCoordinator = [[SKKAuthenticationCoordinator alloc] initWithNavigationViewController:self.navigationController];  
	authCoordinator.delegate = self;  
	[authCoordinator start];  
	[self.childCoordinators addObject:authCoordinator];  
}
```

我们使用`childCoordinators`数组来保存子coordinator，防止它被释放掉。

视图控制器存在于树形结构中，每个视图控制器有一个视图。视图也组成一棵树，每个视图有个layer。layer也存在于树中。因为`childCoordinators`数组的存在，你得到了一颗coordinator的树。

子coordinator会创建一些视图控制器，等待他们工作，当工作完了就会发信号给我们。当一个coordinator发信号显示它已经完成时，它会自动清除它，pop它push过的任何视图控制器，然后使用delegate将消息返回给它的父coordinator。

一旦我们授权认证通过，我们会得到delegate消息，释放子coordinator，然后回到定期规划好的地方。

```objective-c
- (void)coordinatorDidAuthenticate:(SKAuthenticationCoordinator *)coordinator {  
	[self.childCoordinators removeObject:coordinator];  
	[self showContent];  
}  
```

在授权认证coordinator中，它会创建任何需要的视图控制器，然后推送到导航控制器里。我们来看一下到底是怎样的：

```objective-c
@implementation AuthCoordinator

- (instancetype)initWithNavigationController:(UINavigationController *)navigationController {  
	self = [super init];  
	if (!self) return nil;
	
	_navigationController = navigationController;
	
	return self;  
}  
```

初始化和app coordinator相似。

```objective-c
- (void)start {  
	SKFirstRunViewController *firstRunViewcontroller = [SKFirstRunViewController new];  
	firstRunViewcontroller.delegate = self;  
	[self.navigationController pushViewController:firstRunViewcontroller animated:NO];  
}
```

`SKFirstRunViewController`有个注册、登录按钮，可能还有一些滑动显示的介绍。我们把coordinator设为它的delegate，然后push到导航控制器。

当用户点击“注册”按钮时，我们会通过delegate了解到。`SKFirstRunViewController`不需要知道哪个注册视图控制器要创建和展现，coordinator会处理这些。

```objective-c
- (void)firstRunViewControllerDidTapSignup:(SKFirstRunViewController *)firstRunViewController {  
	SKSignUpViewController *signUpViewController = [[SKSignUpViewController alloc] init];  
	signupViewController.delegate = self;  
	[self.navigationController pushViewController:signupViewController animated:YES];  
} 
```

我们变成注册视图控制器的delegate，那样当它的按钮被按下时，它就会通知我们。

```objective-c
- (void)signUpViewController:(SKSignUpViewController *)signupViewController didTapSignupWithEmail:(NSString *)email password:(NSString *)password {  
	//...  
}  
```

在这里，我们实际执行了注册 API请求的工作，并保存了授权认证token，然后我们通知了我们的父coordinator。

当视图控制器(如用户输入)发生任何事情时，它将告诉它的delegate(在本例中是coordinator)，coordinator将执行用户想要的实际任务。让coordinator完成工作是很重要的，这样视图控制器仍然是惰性的。

### coordinator 的优势

#### 1. 每个视图控制器都是孤立的

视图控制器不知道任何东西，除了呈现数据。有任何事情发生时，视图控制器会告诉它的delegate，当然它并不知道它的delegate是谁。

以前，当遇到分岔路的时候，视图控制器需要问“我在iPad上还是iPhone上？”，“用户在做A/B测试吗？”。现在，视图控制器再也不需要问这样的问题了。我们是否只是把这个问题推到coordinator上? 在某种程度上来说是这样的，但是我们可以用更好的方式解决它。

当我们需要同时有两个flow用于A/B测试或多个 size class 时，，您只需用整个coordinator 对象，替换在视图控制器中粘贴一堆条件语句。

如果你想理解flow的工作方式，现在非常简单了，因为所有代码都在一个地方。

#### 2. 可重用的视图控制器

视图控制器不会对将对被present的上下文进行任何假设，也不会假设他们的按钮将被用于什么。它们可以很好地被使用和重用，不需要放置任何逻辑在里面。

假如你正在开发一个iPad版本的app，唯一要做的事情就是替换你的coordinator，你可以重用所有的视图控制器。

#### 3. 每一个任务以及子任务都有一个专注的封装方式

即使跨多个视图控制器的任务，它也被封装起来了。如果你的iPad版本只是重用其中的某些子任务，那也非常容易。

#### 4. Coordinator 把显示绑定和副作用分开

当弹出一个视图控制器时，你再也不需要担心它会弄乱你的数据。它只能读取和显示，绝不会写入或破坏数据。这个概念和"[命令与查询分离](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)"相似。

#### 5. Coordinator 完全在你的掌控之中

你再也不用坐等`viewDidLoad`运行后才开始工作，你自己完全掌握着整个流程。再也不需要去理解`UIViewController`里面的不可见代码。不是被访问了，而是你开始主动访问。

翻转这个模型让你更容易理解会发生什么事情。对你来说，app的动作完全是透明的，`UIKit`现在仅仅是一个库，当你需要的时候才访问它。

[Backchannel SDK](https://github.com/backchannel/BackchannelSDK-iOS) 使用了这种模式来管理所有的视图控制器（译注：本文作者所在公司的开源的SDK）。本文中的app coordinator和 auth coordinator 例子都来自于此。

最后，coordinator只是一种组织模式。没有库给你使用，因为它很简单。也没有`pod`给你`install`，不需要继承什么。甚至都不需要遵循一个协议。这不是一个弱点，而是使用像coordinator这样的模式的优势：它只是你的代码，没有依赖项。

它们能让你的app和你的代码更易管理。视图控制器将更易重用，让你的app变成比以前更简单。