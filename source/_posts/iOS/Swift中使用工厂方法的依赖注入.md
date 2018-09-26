---
title: Swift中使用工厂方法的依赖注入
date: 2018-09-25 23:51:53
category: iOS
---

> 阅读[原文](https://www.swiftbysundell.com/posts/dependency-injection-using-factories-in-swift)，作者是[John Sundell](https://twitter.com/johnsundell)

# Swift 中使用工厂方法的依赖注入

依赖注入的好处暂且不表，大家都知道。最普遍的做法是在初始化的时候传入它所需要的所有依赖，但是碰到下面这种情况，你就会感觉很蛋疼了：

```swift
class UserManager {
    init(dataLoader: DataLoader, database: Database, cache: Cache,
         keychain: Keychain, tokenManager: TokenManager) {
        ...
    }
}
```

<!-- more -->

这就导致了一个**Massive**的初始化器，以及复杂的依赖管理问题。

# 传递依赖

在使用依赖注入时，我们经常陷入上面这种情况，原因是为了以后可能要用到它，那我们就必须传递下去。

举个例子，我们有个`MessageListViewController`消息类，它依赖`MessageLoader`，用来加载所有消息：

```swift
class MessageListViewController: UITableViewController {
    private let loader: MessageLoader

    init(loader: MessageLoader) {
        self.loader = loader
        super.init(nibName: nil, bundle: nil)
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        loader.load { [weak self] messages in
            self?.reloadTableView(with: messages)
        }
    }
}
```

目前看来问题不大，因为我们只有一个依赖。然而，假设我们需要在点击列表项时，导航到下一个视图`MessageViewController`，让用户查看消息详情，并且回复消息。为了实现这个功能，我们需要一个`MessageSender`消息发送类，注入到`MessageViewController`，就像这样：

```swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let message = messages[indexPath.row]
    let viewController = MessageViewController(message: message, sender: sender)
    navigationController?.pushViewController(viewController, animated: true)
}
```

问题来了，因为`MessageViewController`需要一个`MessageSender`的实例，所以我们得让`MessageListViewController`知晓有这么一个类。最简单的方法就是，在`MessageListViewController`初始化的时候，也添加一个`sender`：

```swift
class MessageListViewController: UITableViewController {
    init(loader: MessageLoader, sender: MessageSender) {
        ...
    }
}
```

虽然上面的代码可以工作，但是它让我们走入了`Massive`初始化器的道路上，使`MessageListViewController`更难用了（好奇的是，为什么列表一开始就要关心`sender`？）。

另外一个普遍的方法就是使用单例，到处都可以访问，而且很方便：

```swift
let viewController = MessageViewController(
    message: message,
    sender: MessageSender.shared
)
```

但是，我们必须[避免使用单例](https://www.swiftbysundell.com/posts/avoiding-singletons-in-swift)。单例方法伴随着很多重大的缺点，会让我们很难理解不清楚的依赖架构。

# 工厂方法重用

如果我们能够跳过上面的所有步骤，让`MessageListViewController`完全不需要关心`MessageSender`，以及随后的视图控制器需要的所有其他依赖，那就棒极了。

我们可以有某种形式的工厂方法，简单地请求一个`message`参数来创建一个`MessageViewController`，就像这样：

```Swift
let viewController = factory.makeMessageViewController(for: message)
```

就像我们在[使用工厂模式避免共享状态](https://www.swiftbysundell.com/posts/using-the-factory-pattern-to-avoid-shared-state-in-swift?rq=factories)中说过的一样，工厂方法可以让你完全解耦一个对象的**使用**和**创建**。当你想要重构或者改变一些东西时，这个方法能使很多对象与它们的依赖保持一种宽松的耦合关系。

## 如何做到呢？

我们先定义一个协议，用来创建 App 内需要的视图控制器，它并不需要知道关于依赖和初始化器的任何东西：

```Swift
protocol ViewControllerFactory {
    func makeMessageListViewController() -> MessageListViewController
    func makeMessageViewController(for message: Message) -> MessageViewController
}
```

接着，我们创建一个工厂协议，用来创建依赖`MessageLoader`：

```swift
protocol MessageLoaderFactory {
    func makeMessageLoader() -> MessageLoader
}
```

## 单一依赖

当工厂协议准备好之后，我们回到`MessageListViewController`，用一个 factory 来替代依赖实例：

```Swift
class MessageListViewController: UITableViewController {
    // Here we use protocol composition to create a Factory type that includes
    // all the factory protocols that this view controller needs.
    typealias Factory = MessageLoaderFactory & ViewControllerFactory

    private let factory: Factory
    // We can now lazily create our MessageLoader using the injected factory.
    private lazy var loader = factory.makeMessageLoader()

    init(factory: Factory) {
        self.factory = factory
        super.init(nibName: nil, bundle: nil)
    }
}
```

目前为止，我们完成了两件事：

1. 将依赖列表缩减到单个工厂
2. `MessageListViewController`现在不需要关心`MessageViewController`的依赖了

## 依赖容器

现在到了实现工厂协议的时候了。在此之前，我们先定义一个`DependencyContainer`，它包含了 App 内的所有需要注入的依赖，包括`MessageSender`。可能还有更底层逻辑的类，比如`NetworkManager`。

```Swift
class DependencyContainer {
    private lazy var messageSender = MessageSender(networkManager: networkManager)
    private lazy var networkManager = NetworkManager(urlSession: .shared)
}
```

这里我们使用`lazy`属性来定义依赖实例，依赖在初始化对象时引用同类的其他属性，而且它还能帮助我们避免[循环依赖](https://en.wikipedia.org/wiki/Circular_dependency)。

最后，我们让依赖容器遵循工厂协议：

```Swift
extension DependencyContainer: ViewControllerFactory {
    func makeMessageListViewController() -> MessageListViewController {
        return MessageListViewController(factory: self)
    }

    func makeMessageViewController(for message: Message) -> MessageViewController {
        return MessageViewController(message: message, sender: messageSender)
    }
}

extension DependencyContainer: MessageLoaderFactory {
    func makeMessageLoader() -> MessageLoader {
        return MessageLoader(networkManager: networkManager)
    }
}
```

## 分散的所有权

接下来我们得思考几个终极问题：

- 在哪里存储依赖容器？
- 谁拥有依赖容器？
- 在哪里配置依赖容器？

这是个很有意思的事情：因为我们把依赖容器注入到对象所需的工厂的实现中，并且由于这些对象对其工厂是强引用，所以我们不需要将容器存储在任何地方 👏。

比方说，`MessageListViewController`是 App 的根视图控制器，我们只需要创建一个`DependencyContainer`实例，然后传过去即可：

```Swift
let container = DependencyContainer()
let listViewController = container.makeMessageListViewController()

window.rootViewController = UINavigationController(
    rootViewController: listViewController
)
```

完全没有必要保存一个全局变量，或者在 app delegate 中使用可选属性。

# 结论

使用工厂协议和容器的方法进行依赖注入，可以避免传递多个依赖和创建复杂的初始化器。虽然这并不是银弹，但是能够让依赖注入更简单，这将使您对对象的实际依赖关系有一个更清晰的了解，而且简化测试。

因为我们将所有工厂定义为协议，所以我们可以在测试中通过实现任何给定的工厂协议的特定版本来模拟它们。
