
---
title: 【学习】Swift 里的三种依赖注入方式
date: 2018-09-25 23:51:53
category: iOS
---

    
    
> 原文[在此](https://www.swiftbysundell.com/posts/different-flavors-of-dependency-injection-in-swift)， 作者是[John Sundell](https://twitter.com/johnsundell) 

# Swift 里的三种依赖注入方式

依赖注入可以很好的帮我们在项目中进行解耦和测试，下面介绍三种方法，每个都有自己的优势和劣势。

# 初始化方法
在初始化函数里，将依赖对象传入进去。这应该是最普遍的做法了，最大好处是它保证我们的对象拥有他们所需要的一切，以便立即完成他们的工作。

假设我们要创建一个 `FileLoader`，它从磁盘读取文件。它有两个依赖，一个是 `FileManager`实例，还有一个是 `Cache`实例。使用初始化方法的进行依赖注入，看起来像下面这样：

```swift
class FileLoader {
    private let fileManager: FileManager
    private let cache: Cache

    init(fileManager: FileManager = .default,
         cache: Cache = .init()) {
        self.fileManager = fileManager
        self.cache = cache
    }
}
```
> 注意这里的默认参数使用，它有个好处是：**避免使用单例和创建新实例**

# 属性方法
当你使用自定义的类时，初始化方法非常适合依赖注入。但是当你继承自系统的类，就有一些困难了。其中一个例子就是，当创建视图控制器时，特别是使用 xib 或者 storyboard 的情况下，初始化方法就不再受你管控了。

这时，基于属性的依赖注入就是个不错的选择，它可以在初始化之后进行赋值。而且还可以减少样板代码，尤其是当你存在不需要注入的默认值的时候。

我们来看个例子，`PhotoEditorViewController`是一个让用户从图库选择照片进行编辑的类。它需要两个依赖，一个是系统的`PHPhotoLibrary`单例，还有一个我们自定义的图片编辑引擎`PhotoEditorEngine`。在不使用自定义初始化方法的情况下进行依赖注入，我们可以向下面一样，创建两个有默认值的可变属性：

```swift
class PhotoEditorViewController: UIViewController {
    var library: PhotoLibrary = PHPhotoLibrary.shared()
    var engine = PhotoEditorEngine()
}
```
请注意如何使用协议来为系统图库类提供一种更加抽象的`PhotoLibrary`接口，这种技术来源于"[3个简单步骤测试使用系统单例的Swift代码](https://www.swiftbysundell.com/posts/testing-swift-code-that-uses-system-singletons-in-3-easy-steps)"。

上面的好处是，我们仍然可以很容易地在测试中注入mock，只需重新赋值给视图控制器的属性。

```swift
class PhotoEditorViewControllerTests: XCTestCase {
    func testApplyingBlackAndWhiteFilter() {
        let viewController = PhotoEditorViewController()

        // Assign a mock photo library to gain complete control over
        // what photos are stored in it
        let library = PhotoLibraryMock()
        library.photos = [TestPhotoFactory.photoWithColor(.red)]
        viewController.library = library

        // Run our testing commands
        viewController.selectPhoto(atIndex: 0)
        viewController.apply(filter: .blackAndWhite)
        viewController.savePhoto()

        // Assert that the outcome is correct
        XCTAssertTrue(photoIsBlackAndWhite(library.photos[0]))
    }
}
```



# 参数方法

这种方法非常适合老项目，使其更易测试，而且不需要修改太多已有的结构。

很多时候我们只需要一次特别的依赖注入，或者在特定条件下模拟它。这时，我们可以创建一个特别的接口，接受一个依赖参数。

举个例子，我们有个`NoteManager`类来管理用户写的所有笔记，然后提供一个基于`query`条件的搜索笔记接口。这个操作可能需要耗费一点时间（当用户的笔记比较多的时候），所以我们放在后台队列来处理：

```swift
class NoteManager {
    func loadNotes(matching query: String,
                   completionHandler: @escaping ([Note]) -> Void) {
        DispatchQueue.global(qos: .userInitiated).async {
            let database = self.loadDatabase()
            let notes = database.filter { note in
                return note.matches(query: query)
            }

            completionHandler(notes)
        }
    }
}
```

上面的方法在我们的产品代码里是个很好的解决方案。在测试中，为了[减少flakiness](https://www.swiftbysundell.com/posts/reducing-flakiness-in-swift-tests)（测试中的不稳定性，有时同样的环境、参数下会得出不一样的测试结果），我们一般尽可能地避免异步代码和并行。虽然可以用初始化或者属性的方法来指定一个特定的队列给`NoteManager`运行，但是它需要对类进行较大的改变，而这并不是我们现在想要的。

基于参数的依赖注入就是由此而来，不用重构整个类，仅仅是注入一个`loadNotes`操作运行的队列即可：

```swift
class NoteManager {
    func loadNotes(matching query: String,
                   on queue: DispatchQueue = .global(qos: .userInitiated),
                   completionHandler: @escaping ([Note]) -> Void) {
        queue.async {
            let database = self.loadDatabase()
            let notes = database.filter { note in
                return note.matches(query: query)
            }

            completionHandler(notes)
        }
    }
}
```

这可以简单地让测试代码跑在自定义的队列里，而且我们可以一直等待它完成。这就有点像在测试里写同步代码一样了，让事情变得更简单以及更可预测。

基于参数的依赖注入的另一个使用场景就是：测试静态API。对于静态API，我们没有一个初始化方法，理想情况下，我们也不应该静态地保持任何状态，所以基于参数的依赖注入成为了一个很好的选择。我们来看一下静态类`MessageSender`，它依赖单例：

```Swift
class MessageSender {
    static func send(_ message: Message, to user: User) throws {
        Database.shared.insert(message)

        let data: Data = try wrap(message)
        let endpoint = Endpoint.sendMessage(to: user)
        NetworkManager.shared.post(data, to: endpoint.url)
    }
}
```

一个理想的长久解决方案可能是重构`MessageSender`，不用静态方法，将其注入到每一个需要用到的地方。但为了简单地测试它（比如，为了重现或者验证bug），我们可以把依赖当做参数，而不是单例：

```Swift
class MessageSender {
    static func send(_ message: Message,
                     to user: User,
                     database: Database = .shared,
                     networkManager: NetworkManager = .shared) throws {
        database.insert(message)

        let data: Data = try wrap(message)
        let endpoint = Endpoint.sendMessage(to: user)
        networkManager.post(data, to: endpoint.url)
    }
}
```

我们再次用到了默认参数，两次都是为了方便，但是更重要的是能够添加测试到我们的代码里，并且是100%的向后兼容。

# 总结

哪一种依赖注入的方法是最好的？我的答案是：看情况😅。软件开发中是[没有银弹](https://zh.wikipedia.org/wiki/%E6%B2%A1%E6%9C%89%E9%93%B6%E5%BC%B9)的，一个问题的解决方案有很多种，不同的业务场景适合用不同的方法。了解每种方法的优势和劣势，对你在写代码时有很大的帮助，能让你做出更明智的决定。