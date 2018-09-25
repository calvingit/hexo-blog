---
title: 【学习】Child视图控制器的模块化妙用
date: 2018-09-25 23:51:53
category: iOS
---

    > 作者是[John Sundell](https://twitter.com/johnsundell)，阅读[原文](https://www.swiftbysundell.com/posts/using-child-view-controllers-as-plugins-in-swift)

当有很多通用功能的时候，比如 loading 界面，报错弹框等，大部分人习惯性会创建一个基类`BaseViewController`，然后所有视图控制器都继承它：

```swift
class BaseViewController: UIViewController {
    func showActivityIndicator() {
        ...
    }

    func hideActivityIndicator() {
        ...
    }

    func handle(_ error: Error) {
        ...
    }
}
```

初看这一切都很方便，但是时间一长，你就会把大量的东西都堆在`BaseViewController`里面，就会出现**Massive View Controller**的问题，导致很难维护。

> 虽然表面上看起来`BaseViewController`的子类并不是**Massive**，通用功能代码都在父类，但由于继承的特性，子类也依赖了一些不该有的东西。并且假如需要单独测试某个 View Controller，那就必须依赖`BaseViewController`，而`BaseViewController`又包含了很多其他的依赖，很不方便。

# Child 视图控制器

把一个视图控制器根据不同业务功能分拆成几个 child 视图控制器，有以下几个优点：

- 可以接收`viewDidLoad`、`viewWillAppear`等事件，就像它的 Parent 视图控制器
- 内部可以自己管理视图布局（旋转）和业务逻辑
- 可以当做插件使用，很方便的添加和移除

举个例子，我们来实现一个 loading 功能的 child 视图控制器：

```swift
class LoadingViewController: UIViewController {
    private lazy var activityIndicator = UIActivityIndicatorView(activityIndicatorStyle: .gray)

    override func viewDidLoad() {
        super.viewDidLoad()

        activityIndicator.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(activityIndicator)

        NSLayoutConstraint.activate([
            activityIndicator.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            activityIndicator.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        // 延时0.5秒，避免数据加载太快
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) { [weak self] in
            self?.activityIndicator.startAnimating()
        }
    }
}
```

比起使用`BaseViewController`，我们可以在任何一个视图控制器需要显示 loading 标识时，简单的把`LoadingViewController`添加到 child 视图控制器里面。这样做也可以让我们把所有显示 loading 标识逻辑放在单一的地方，避免和一些不想关的东西混在一起。

# 添加和移除 child 控制器

我们可以把这两个功能作为`UIViewController`的 extension：

```
extension UIViewController {
    func add(_ child: UIViewController) {
        addChildViewController(child)
        view.addSubview(child.view)
        child.didMove(toParentViewController: self)
    }

    func remove() {
        guard parent != nil else {
            return
        }

        willMove(toParentViewController: nil)
        removeFromParentViewController()
        view.removeFromSuperview()
    }
}
```

# 使用 child 视图控制器

比起使用 sub view，使用 child 视图控制器的一个好处是：UIKit 管理者 parent 和 child 视图控制器的 layout，会把标准的`UIViewController`事件发送给所有的 child 视图控制器。

下面是在一个列表视图控制器`ListViewController`显示 loading 的代码：

```swift
class ListViewController: UITableViewController {
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        loadItems()
    }

    private func loadItems() {
        let loadingViewController = LoadingViewController()
        add(loadingViewController)

        dataLoader.loadItems { [weak self] result in
            loadingViewController.remove()
            self?.handle(result)
        }
    }
}
```

# 错误处理

创建`ErrorViewController`，当网络出现错误时，提示错误信息。它还包含一个重试按钮，所以我们再加一个`reloadHandler`闭包，当按钮被点击时执行：

```
class ErrorViewController: UIViewController {
    var reloadHandler: () -> Void = {}
}
```

就像我们显示 loading 一样，我们可以这样显示错误信息：

```swift
private extension ListViewController {
    func handle(_ error: Error) {
        let errorViewController = ErrorViewController()

        errorViewController.reloadHandler = { [weak self] in
            self?.loadItems()
        }

        add(errorViewController)
    }
}
```

# 参考

- [A Better MVC, Part 3: Fixing Massive View Controller](https://davedelong.com/blog/2017/11/06/a-better-mvc-part-3-fixing-massive-view-controller/)
