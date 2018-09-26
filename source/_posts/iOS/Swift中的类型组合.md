---
title: Swift中的类型组合
date: 2018-09-25 23:51:53
category: iOS
---

    > 作者是[John Sundell](https://twitter.com/johnsundell)，阅读[原文](https://www.swiftbysundell.com/posts/composing-types-in-swift)

# Swift 中的组合类型

组合是个非常有用的技术，可以让我们在多个类型中以一种更加解耦的方式共享代码。它通常是取代子类化的一种替代方法，而不是依赖继承。

下面我们来看一下 Swift 中的`struct`、`class`、`enum`是如何组合的。

<!-- more -->

## Struct 组合

假设我们开发一个社交 app，它有`User`和`Friend`两个模型。`User`代表 app 里面的所有类型用户，`Friend`包含一个用户和成为朋友的日期。

如果你以前学习的编程语言严重依赖继承，比如 Objective-C，那么当你设计模型时，第一个想法可能是让`Friend`继承于`User`，就像这样：

```swift
class User {
    var name: String
    var age: Int
}

class Friend: User {
    // 成为朋友的日期
    var friendshipDate: Date
}
```

上面的代码可以工作，但是有三大缺点：

- 要使用继承，必须是`class`引用类型。但在 Swift 里面，模型通常是不可变的，应该是`struct`
- `Friend`可以用于参数是`User`的函数里。看起来好像无害，但会增加错误使用代码的风险，比如调用函数`saveDataForCurrentUser`时，传了一个`Friend`
- 对`Friend`来说，实际上我们并不想要`User`属性是可变的（你改不了朋友的名字 😅），但因为我们依赖继承，也就继承了所有属性的可变性。

下面就是使用组合的 struct 来组织代码：

```swift
struct User {
    var name: String
    var age: Int
}

struct Friend {
    let user: User
    var friendshipDate: Date
}
```

现在，`Friend`的`user`属性是`let`，意味着无法更改。

# Class 组合

那是不是说我们应该让所有的类型都是不可变的`struct`呢？并不是。`class`类型是非常强大的工具，而且有时你会希望类型具有引用语义而不是值语义。但即使使用`class`，我们也不一定非要用继承，组合有时也很不错。

假设我们需要一个显示朋友列表的视图控制器`FriendListViewController`，它有一个`tableView`。最普通的做法肯定就是在`FriendListViewController`里面实现`tableView`的 datasouce：

```swift
extension FriendListViewController: UITableViewDataSource {
   func tableView(_ tableView: UITableView,
                  numberOfRowsInSection section: Int) -> Int {
        return friends.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        ...
    }
}
```

这并没有什么不对，但绝不建议这么做。我们可以使用组合的方式来让代码更加解耦、可重用。

我们先创建一个专用的 datasource 对象，遵循`UITableViewDataSource`协议，然后我们把一个`freind`数组赋给它。

```
class FriendListTableViewDataSource: UITableViewDataSource {
    var friends = [Friend]()

    func tableView(_ tableView: UITableView,
                   numberOfRowsInSection section: Int) -> Int {
        return friends.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        ...
    }
}
```

然后在`FriendListViewController`里面，我们创建一个`FriendListTableViewDataSource`对象引用，用它作为 table view 的 datasource:

```swift
class FriendListViewController: UITableViewController {
    private let dataSource = FriendListTableViewDataSource()

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.dataSource = dataSource
    }

    func render(_ friends: [Friend]) {
        dataSource.friends = friends
        tableView.reloadData()
    }
}
```

这种方法的优美之处在于，万一我们想要在其他地方显示朋友列表时可以很容易地重用这个功能，比如查找朋友界面。总之，把东西移出视图控制器是一种避免"Massive view controller"症状的好方法。就像我们把 datasource 作为一个独立的组合类型一样，我们也可以对其他的功能做同样的事（数据、图片加载，缓存等）。

另一个在多个视图控制器之间使用组合的方式就是使用 child 视图控制器，详情请看这篇[文章](https://www.swiftbysundell.com/posts/using-child-view-controllers-as-plugins-in-swift)。

# Enum 组合

假设我们有一个`Operation`类型来做一些后台任务，任务状态使用`State`枚举表示，有 loading， failed，finished 三种状态：

```swift
class Operation {
    var state = State.loading
}

extension Operation {
    enum State {
        case loading
        case failed(Error)
        case finished
    }
}
```

上面的代码看起来很直观。现在我们来看一下如何在视图控制器里面使用 operation 来处理一组图片的情况：

```swift
class ImageProcessingViewController: UIViewController {
    func processImages(_ images: [UIImage]) {
        // 创建一个operation，在后台处理图片，要么成功要么抛出异常。
        let operation = Operation {
            let processor = ImageProcessor()
            try images.forEach(processor.process)
        }

        // 传一个状态变化时处理的闭包，根据不同的状态刷新界面
        operation.startWithStateHandler { [weak self] state in
            switch state {
            case .loading:
                self?.showActivityIndicatorIfNeeded()
            case .failed(let error):
                self?.cleanupCache()
                self?.removeActivityIndicator()
                self?.showErrorView(for: error)
            case .finished:
                self?.cleanupCache()
                self?.removeActivityIndicator()
                self?.showFinishedView()
            }
        }
    }
}
```

咋一看，好像并没有什么不对的地方。但我们仔细看处理`failed`和`finished`的情况，这里出现了代码重复。

代码重复并不总是坏的，但是当涉及到处理不同的状态时，最好是尽可能少地复制代码。否则我们就要做更多的测试，或者当我们需要修改代码时就会很容易出现 bug。

另一种方案就是使用组合，我用两个 enum 来表示，一个表示`State`，另一个代表`Outcome`:

```swift
extension Operation {
    enum State {
        case loading
        case finished(Outcome)
    }

    enum Outcome {
        case failed(Error)
        case succeeded
    }
}
```

然后我们可以在状态处理闭包里面这么做：

```swift
operation.startWithStateHandler { [weak self] state in
    switch state {
    case .loading:
        self?.showActivityIndicatorIfNeeded()
    case .finished(let outcome):
        // 成功和失败的通用操作都放在这里。
        self?.cleanupCache()
        self?.removeActivityIndicator()

        switch outcome {
        case .failed(let error):
            self?.showErrorView(for: error)
        case .succeeded:
            self?.showFinishedView()
        }
    }
}
```

这样我们就可以避免代码重复了。
