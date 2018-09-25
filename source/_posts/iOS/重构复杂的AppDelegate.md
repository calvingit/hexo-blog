---
title: 【学习】重构复杂的AppDelegate
date: 2018-09-25 23:51:53
category: iOS
---

# 命令模式

[命令模式](https://zh.wikipedia.org/wiki/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F) 以对象来代表单一动作和事件，并把对象叫做命令。命令封装了所有参数，命令的调用者无需了解命令做了什么。

我们给每一个 app delegate 责任定义一个命令，命令的名称表明了目的。

```swift
protocol Command {
    func execute()
}

struct InitializeThirdPartiesCommand: Command {
    func execute() {
        // 第三库初始化
    }
}

struct InitailizeViewControllerCommand: Command {
    let keyWindow: UIWindow

    func execute() {
        keyWindow.rootViewController = UIViewController()
    }
}

struct InitailizeAppearanceCommand: Command {
    func execute() {
        // 设置 UIAppearance
    }
}

struct RegisterToRemoteNotificationCommand: Command {
    func execute() {
        // 注册远程通知
    }
}
```

接下来，我们定义一个`StartupCommandsBuilder`来封装命令的创建细节。`AppDelegate` 调用 builder 来初始化命令，然后触发命令。

```swift
// MARK: - Builder

final class StartupCommandsBuilder {
    private var window: UIWindow!
    func setKeyWindow(_ window: UIWindow) -> StartupCommandsBuilder {
        self.window = window
        return self
    }
    func build() -> [Command] {
        return [
            InitializeThirdPartiesCommand(),
            InitialViewControllerCommand(keyWindow: window),
            InitializeAppearanceCommand(),
            RegisterToRemoteNotificationsCommand()
        ]
    }
}

// MARK: - App Delegate

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        StartupCommandsBuilder()
            .setKeyWindow(window!)
            .build()
            .forEach { $0.execute() }

        return true
    }
}
```

新的命令可以直接添加到 builder，而不需要修改`AppDelegate`。

# 组合模式

[组合模式](http://www.runoob.com/design-pattern/composite-pattern.html)把一组相似的对象当做一个单一对象处理，依据树形结构来组合对象。这种模式创建了一个包含自己对象组的类，提供了修改相同对象组的方式。

iOS 里面最显著的用法就是`UIView`的 `subViews`。

```swift
typealias AppDelegateType = UIResponder & UIApplicationDelegate

class CompositeAppDelegate: AppDelegate {
    private let appDelegates: [AppDelegateType]

    init(appDelegates: [AppDelegateType]) {
        self.appDelegates = appDelegates
    }

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        appDelegates.forEach { _ = $0.application?(application, didFinishLaunchingWithOptions: launchOptions) }
        return true
    }

    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        appDelegates.forEach { _ = $0.application?(application, didRegisterForRemoteNotificationsWithDeviceToken: deviceToken) }
    }
}
```

然后，实现`AppDelegate`：

```swift
class PushNotificationsAppDelegate: AppDelegateType {
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        // Registered successfully
    }
}

class StartupConfiguratorAppDelegate: AppDelegateType {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Perform startup configurations, e.g. build UI stack, setup UIApperance
        return true
    }
}

class ThirdPartiesConfiguratorAppDelegate: AppDelegateType {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Setup third parties
        return true
    }
}
```

我们定义一个工厂类`AppDelegateFactory`实现创建 App Delegate 的逻辑。我们的主`AppDelegate`创建了子`AppDelegate`组合，然后传递方法给它们。

```swift
enum AppDelegateFactory {
    static func makeDefault() -> AppDelegateType {
        return CompositeAppDelegate(appDelegates: [PushNotificationsAppDelegate(), StartupConfiguratorAppDelegate(), ThirdPartiesConfiguratorAppDelegate()])
    }
}

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    let appDelegate = AppDelegateFactory.makeDefault()

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        _ = appDelegate.application?(application, didFinishLaunchingWithOptions: launchOptions)
        return true
    }

    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        appDelegate.application?(application, didRegisterForRemoteNotificationsWithDeviceToken: deviceToken)
    }
}
```

# 中介者模式

[中介者模式](http://www.runoob.com/design-pattern/mediator-pattern.html) 使用一个中介对象来封装一系列的对象交互。

我们定义一个中介者`AppLifecycleMediator`，用它来传递`UIApplication`的生命周期事件给隐藏的监听者。这些监听者必须实现协议`AppLifecycleListener`。

```swift
protocol AppLifecycleListener {
    func onAppWillEnterForground()
    func onAppDidEnterBackground()
    func onAppDidFinishLaunching()
}

class AppLifecycleMediator: NSObject {
    private let listeners: [AppLifecycleListener]

    init(listeners: [AppLifecycleLister]) {
        self.listeners = listeners
        super.init()
        subscribe()
    }

        deinit {
        NotificationCenter.default.removeObserver(self)
    }

    private func subscribe() {
        NotificationCenter.default.addObserver(self, selector: #selector(onAppWillEnterForeground), name: .UIApplicationWillEnterForeground, object: nil)
        NotificationCenter.default.addObserver(self, selector: #selector(onAppDidEnterBackground), name: .UIApplicationDidEnterBackground, object: nil)
        NotificationCenter.default.addObserver(self, selector: #selector(onAppDidFinishLaunching), name: .UIApplicationDidFinishLaunching, object: nil)
    }

    @objc private func onAppWillEnterForeground() {
        listeners.forEach { $0.onAppWillEnterForeground() }
    }

    @objc private func onAppDidEnterBackground() {
        listeners.forEach { $0.onAppDidEnterBackground() }
    }

    @objc private func onAppDidFinishLaunching() {
        listeners.forEach { $0.onAppDidFinishLaunching() }
    }
}

extension AppLifecycleMediator {
    static func makeDefaultMediator() -> AppLifecycleMediator {
        let listener1 = ...
        let listener2 = ...
        return AppLifecycleMediator(listeners: [listener1, listener2])
    }
}
```

现在，在`AppDelegate`里面只需要一行代码就行了：

```swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?

    let mediator = AppLifecycleMediator.makeDefaultMediator()

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        return true
    }
}
```
