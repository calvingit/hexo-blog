---
title: Coordinator and FlowController
date: 2018-09-25 23:51:53
category: iOS
---

Every new architecture that comes out, either [iOS](https://github.com/onmyway133/fantastic-ios-architecture) or [Android](https://github.com/onmyway133/fantastic-android-architecture), makes me very excited. I'm always looking for ways to structure apps in a better way. But after some times, I see that we're too creative in creating architecture, aka constraint, that is too far away from the platform that we're building. I often think "If we're going too far from the system, then it's very hard to go back"

<!-- more -->

I like things that embrace the system. One of them is [Coordinator](http://khanlou.com/2015/10/coordinators-redux/) which helps in encapsulation and navigation. Thanks to my friend [Vadym](https://github.com/vadymmarkov/) for showing me `Coordinator` in action.

The below screenshot from [@khanlou](https://github.com/khanlou) 's [talk](https://www.youtube.com/watch?v=a1g3k3NObkE) at CocoaHeads Stockholm clearly says many things about `Coordinator`

[![img](https://user-images.githubusercontent.com/2284279/32828592-d03ad9a0-c9ef-11e7-8759-cb0292d668df.png)](https://user-images.githubusercontent.com/2284279/32828592-d03ad9a0-c9ef-11e7-8759-cb0292d668df.png)

But after reading [A Better MVC](https://davedelong.com/blog/2017/11/06/a-better-mvc-part-3-fixing-massive-view-controller/), I think we can leverage view controller containment to do navigation using `UIViewController` only.

Since I tend to call view controllers as `LoginController, ProfileController, ...` and the term `flow` to group those related screens, what should we call a `Coordinator` that inherits from `UIViewController` 🤔 Let's call it `FlowController` 😎 .

The name is not that important, but the concept is simple. `FlowController` was also inspired by this [Flow Controllers on iOS for a Better Navigation Control](http://albertodebortoli.com/blog/2014/09/03/flow-controllers-on-ios-for-a-better-navigation-control/) back in 2014. The idea is from awesome iOS people, this is just a sum up from my experience 😇

So `FlowController` can just a `UIViewController` friendly version of `Coordinator`. Let see how `FlowController` fits better into `MVC`

> «UIViewController is the center of the universe.»
> — [@onmyway133](https://twitter.com/onmyway133?ref_src=twsrc%5Etfw)
>
> — Elvis Nuñez (
>
> @3lvis
>
> )
>
> \6. oktober 2017

## 1. FlowController and AppDelegate

Your application mostly starts from `AppDelegate`, in that you setup `UIWindow`. So we should follow the same "top down" approach for `FlowController`, starting with `AppFlowController`. You can construct all dependencies that your app need for `AppFlowController`, so that it can pass to other child `FlowController`

Here is how to declare `AppFlowController` in `AppDelegate`

```
struct DependencyContainer: AuthServiceContainer, PhoneServiceContainer, NetworkingServiceContainer,
  LocationServiceContainer, MapServiceContainer, HealthServiceContainer {

  let authService: AuthServiceProtocol
  let phoneService: PhoneService
  let networkingService: NetworkingService
  let locationService: LocationService
  let mapService: MapService
  let healthService: HealthService

  static func make() -> DependencyContainer {
    // Configure and make DependencyContainer here
  }
}

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

  var window: UIWindow?
  var appFlowController: AppFlowController!

  func application(_ application: UIApplication,
                   didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    appFlowController = AppFlowController(
      dependencyContainer: DependencyContainer.make()
    )

    window = UIWindow(frame: UIScreen.main.bounds)
    window?.rootViewController = appFlowController
    window?.makeKeyAndVisible()

    appFlowController.start()

    return true
  }
}
```

Here are some hypothetical `FlowController` that you may encounter

- AppFlowController: manages UIWindow and check whether to show onboarding, login or main depending on authentication state

  - OnboardingFlowController: manages `UIPageViewController` and maybe ask for some permissions

  - LoginFlowController: manages `UINavigationController` to show login, sms verification, forget password, and optionally start `SignUpFlowController`

  - MainFlowController: manages

    ```
    UITabBarController
    ```

    with each tab serving main features

    - FeedFlowController: show feed with list of items
    - ProfileFlowController: show profile
    - SettingsFlowController: show settings, and maybe call logout, this will delegates up the `FlowController` chain.

The cool thing about `FlowController` is it makes your code very self contained, and grouped by features. So it's easy to move all related things to its own package if you like.

## 2. FlowController as container view controller

> In general, a view controller should manage either sequence or UI, but not both.

Basically, `FlowController` is just a container view controller to solve the `sequence`, based on a simple concept called `composition`. It manages many child view controllers in its flow. Let' say we have a `ProductFlowController` that groups together flow related to displaying products, `ProductListController`, `ProductDetailController`, `ProductAuthorController`, `ProductMapController`, ... Each can delegate to the `ProductFlowController` to express its intent, like `ProductListController` can delegate to say "product did tap", so that `ProductFlowController` can construct and present the next screen in the flow, based on the embedded `UINavigationController` inside it.

Normally, a `FlowController` just displays 1 child `FlowController` at a time, so normally we can just update its frame

```
final class AppFlowController: UIViewController {
  override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()

    childViewControllers.first?.view.frame = view.bounds
  }
}
```

## 3. FlowController as dependency container

Each view controller inside the flow can have different dependencies, so it's not fair if the first view controller needs to carry all the stuff just to be able to pass down to the next view controllers. Here are some dependencies

- ProductListController: ProductNetworkingService
- ProductDetailController: ProductNetworkingService, ImageDowloaderService, ProductEditService
- ProductAuthorController: AuthorNetworkingService, ImageDowloaderService
- ProductMapController: LocationService, MapService

Instead the `FlowController` can carry all the dependencies needed for that whole flow, so it can pass down to the view controller if needed.

```
struct ProductDependencyContainer {
  let productNetworkingService: ProductNetworkingService
  let imageDownloaderService: ImageDownloaderService
  let productEditService: ProductEditService
  let authorNetworkingService: AuthorNetworkingService
  let locationService: LocationService
  let mapService: MapService
}

class ProductFlowController {
  let dependencyContainer: ProductDependencyContainer

  init(dependencyContainer: ProductDependencyContainer) {
    self.dependencyContainer = dependencyContainer
  }
}

extension ProductFlowController: ProductListControllerDelegate {
  func productListController(_ controller: ProductListController, didSelect product: Product) {
    let productDetailController = ProductDetailController(
      productNetworkingService: dependencyContainer.productNetworkingService,
      productEditService: dependencyContainer.productEditService,
      imageDownloaderService: dependencyContainer.imageDownloaderService
    )

    productDetailController.delegate = self
    embeddedNavigationController.pushViewController(productDetailController, animated: true)
  }
}
```

Here are some ways that you can use to pass dependencies into `FlowController`

- [Using protocol compositon for dependency injection](http://merowing.info/2017/04/using-protocol-compositon-for-dependency-injection/)
- [Dependency injection using factories in Swift](https://www.swiftbysundell.com/posts/dependency-injection-using-factories-in-swift)

## 4. Adding or removing child FlowController

**Coordinator**

With `Coordinator`, you need to keep an array of child `Coordinators`, and maybe use address (`===` operator) to identify them

```
class Coordinator {
  private var children: [Coordinator] = []

  func add(child: Coordinator) {
    guard !children.contains(where: { $0 === child }) else {
      return
    }

    children.append(child)
  }

  func remove(child: Coordinator) {
    guard let index = children.index(where: { $0 === child }) else {
      return
    }

    children.remove(at: index)
  }

  func removeAll() {
    children.removeAll()
  }
}
```

**FlowController**

With `FlowController`, since it is `UIViewController` subclass, it has `viewControllers` to hold all those child `FlowController`. Just add these extensions to simplify your adding or removing of child `UIViewController`

```
extension UIViewController {
  func add(childController: UIViewController) {
    addChildViewController(childController)
    view.addSubview(childController.view)
    childController.didMove(toParentViewController: self)
  }

  func remove(childController: UIViewController) {
    childController.willMove(toParentViewController: nil)
    childController.view.removeFromSuperview()
    childController.removeFromParentViewController()
  }
}
```

And see in action how `AppFlowController` work with adding

```
final class AppFlowController: UIViewController {
  func start() {
    if authService.isAuthenticated {
      startMain()
    } else {
      startLogin()
    }
  }

  private func startLogin() {
    let loginFlowController = LoginFlowController(
    loginFlowController.delegate = self
    add(childController: loginFlowController)
    loginFlowController.start()
  }

  fileprivate func startMain() {
    let mainFlowController = MainFlowController()
    mainFlowController.delegate = self
    add(childController: mainFlowController)
    mainFlowController.start()
  }
}
```

and with removing when the child `FlowController` finishes

```swift
extension AppFlowController: LoginFlowControllerDelegate {
  func loginFlowControllerDidFinish(_ flowController: LoginFlowController) {
    remove(childController: flowController)
    startMain()
  }
}
```

## 5. AppFlowController does not need to know about UIWindow

**Coordinator**

Usually you have an `AppCoordinator`, which is held by `AppDelegate`, as the root of your `Coordinator` chain. Based on login status, it will determine which `LoginController` or `MainController` will be set as the `rootViewController`, in order to do that, it needs to be injected a `UIWindow`

```swift
window = UIWindow(frame: UIScreen.main.bounds)
appCoordinator = AppCoordinator(window: window!)
appCoordinator.start()
window?.makeKeyAndVisible()
```

You can guess that in the `start` method of `AppCoordinator`, it must set `rootViewController` before `window?.makeKeyAndVisible()` is called.

```swift
final class AppCoordinator: Coordinator {
  private let window: UIWindow

  init(window: UIWindow) {
    self.window = window
  }

  func start() {
    if dependencyContainer.authService.isAuthenticated {
      startMain()
    } else {
      startLogin()
    }
  }
}
```

**FlowController**

But with `AppFlowController`, you can treat it like a normal `UIViewController`, so just setting it as the `rootViewController`

```swift
appFlowController = AppFlowController(
window = UIWindow(frame: UIScreen.main.bounds)
window?.rootViewController = appFlowController
window?.makeKeyAndVisible()

appFlowController.start()
```

## 6. LoginFlowController can manage its own flow

Supposed we have login flow based on `UINavigationController` that can display `LoginController`, `ForgetPasswordController`, `SignUpController`

**Coordinator**

What should we do in the `start` method of `LoginCoordinator`? Construct the initial controller `LoginController` and set it as the `rootViewController` of the `UINavigationController`? `LoginCoordinator` can create this embedded `UINavigationController` internally, but then it is not attached to the `rootViewController` of `UIWindow`, because `UIWindow` is kept privately inside the parent `AppCoordinator`.

We can pass `UIWindow` to `LoginCoordinator` but then it knows too much. One way is to construct `UINavigationController` from `AppCoordinator` and pass that to `LoginCoordinator`

```swift
final class AppCoordinator: Coordinator {
  private let window: UIWindow

  private func startLogin() {
    let navigationController = UINavigationController()

    let loginCoordinator = LoginCoordinator(navigationController: navigationController)

    loginCoordinator.delegate = self
    add(child: loginCoordinator)
    window.rootViewController = navigationController
    loginCoordinator.start()
  }
}

final class LoginCoordinator: Coordinator {
  private let navigationController: UINavigationController

  init(navigationController: UINavigationController) {
    self.navigationController = navigationController
  }

  func start() {
    let loginController = LoginController(dependencyContainer: dependencyContainer)
    loginController.delegate = self

    navigationController.viewControllers = [loginController]
  }
}
```

**FlowController**

`LoginFlowController` leverages `container view controller` so it fits nicely with the way `UIKit` works. Here `AppFlowController` can just add `LoginFlowController` and `LoginFlowController` can just create its own `embeddedNavigationController`.

```swift
final class AppFlowController: UIViewController {
  private func startLogin() {
    let loginFlowController = LoginFlowController(
      dependencyContainer: dependencyContainer
    )

    loginFlowController.delegate = self
    add(childController: loginFlowController)
    loginFlowController.start()
  }
}

final class LoginFlowController: UIViewController {
  private let dependencyContainer: DependencyContainer
  private var embeddedNavigationController: UINavigationController!
  weak var delegate: LoginFlowControllerDelegate?

  init(dependencyContainer: DependencyContainer) {
    self.dependencyContainer = dependencyContainer
    super.init(nibName: nil, bundle: nil)

    embeddedNavigationController = UINavigationController()
    add(childController: embeddedNavigationController)
  }

  func start() {
    let loginController = LoginController(dependencyContainer: dependencyContainer)
    loginController.delegate = self

    embeddedNavigationController.viewControllers = [loginController]
  }
}
```

## 7. FlowController and responder chain

**Coordinator**

Sometimes we want a quick way to bubble up message to parent `Coordinator`, one way to do that is to replicate `UIResponder` chain using `associated object` and protocol extensions, like [Inter-connect with Coordinator](http://aplus.rs/2017/highly-maintainable-app-architecture/)

```swift
extension UIViewController {
	private struct AssociatedKeys {
		static var ParentCoordinator = "ParentCoordinator"
	}

	public var parentCoordinator: Any? {
		get {
			return objc_getAssociatedObject(self, &AssociatedKeys.ParentCoordinator)
		}
		set {
			objc_setAssociatedObject(self, &AssociatedKeys.ParentCoordinator, newValue, .OBJC_ASSOCIATION_ASSIGN)
		}
	}
}

open class Coordinator<T: UIViewController>: UIResponder, Coordinating {
	open var parent: Coordinating?

	override open var coordinatingResponder: UIResponder? {
		return parent as? UIResponder
	}
}
```

**FlowController**

Since `FlowController` is `UIViewController`, which inherits from `UIResponder`, responder chain happens out of the box

> Responder objects—that is, instances of UIResponder—constitute the event-handling backbone of a UIKit app. Many key objects are also responders, including the UIApplication object, UIViewController objects, and all UIView objects (which includes UIWindow). As events occur, UIKit dispatches them to your app's responder objects for handling.

## 8. FlowController and trait collection

**FlowController**

I very much like how [Kickstarter](https://github.com/kickstarter/ios-oss/blob/master/Kickstarter-iOS.playground/Sources/playgroundController.swift#L43) uses trait collection in testing. Well, since `FlowController` is a parent view controller, we can just override its trait collection, and that will affect the size classes of all view controllers inside that flow.

As in [A Better MVC, Part 2: Fixing Encapsulation](https://davedelong.com/blog/2017/11/06/a-better-mvc-part-2-fixing-encapsulation/)

> The huge advantage of this approach is that system features come free. Trait collection propagation is free. View lifecycle callbacks are free. Safe area layout margins are generally free. The responder chain and preferred UI state callbacks are free. And future additions to UIViewController are also free.

From [setOverrideTraitCollection](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621406-setoverridetraitcollection)

> When implementing a custom container view controller, you can use this method to change the traits of any embedded child view controllers to something more appropriate for your layout. Making such a change alters other view controller behaviors associated with that child

```
let trait = UITraitCollection(traitsFrom: [
  .init(horizontalSizeClass: .compact),
  .init(verticalSizeClass: .regular),
  .init(userInterfaceIdiom: .phone)
])

appFlowController.setOverrideTraitCollection(trait, forChildViewController: loginFlowController)
```

## 9. FlowController and back button

**Coordinator**

One problem with `UINavigationController` is that clicking on the default `back button` pops the view controller out of the navigation stack, so `Coordinator` is not aware of that. With `Coordinator` you needs to keep `Coordinator` and `UIViewController` in sync, add try to hook up `UINavigationControllerDelegate` in order to clean up. Like in [Back Buttons and Coordinators](http://khanlou.com/2017/05/back-buttons-and-coordinators/)

```
extension Coordinator: UINavigationControllerDelegate {
	func navigationController(navigationController: UINavigationController,
		didShowViewController viewController: UIViewController, animated: Bool) {

		// ensure the view controller is popping
		guard
		  let fromViewController = navigationController.transitionCoordinator?.viewController(forKey: .from),
		  !navigationController.viewControllers.contains(fromViewController) else {
			return
    	}

		// and it's the right type
		if fromViewController is FirstViewControllerInCoordinator) {
			//deallocate the relevant coordinator
		}
	}
}
```

Or creating a class called `NavigationController` that inside manages a list of child coordinators. Like in [Navigation coordinators](http://irace.me/navigation-coordinators)

```
final class NavigationController: UIViewController {
  // MARK: - Inputs

  private let rootViewController: UIViewController

  // MARK: - Mutable state

  private var viewControllersToChildCoordinators: [UIViewController: Coordinator] = [:]

  // MARK: - Lazy views

  private lazy var childNavigationController: UINavigationController =
      UINavigationController(rootViewController: self.rootViewController)

  // MARK: - Initialization

  init(rootViewController: UIViewController) {
    self.rootViewController = rootViewController

    super.init(nibName: nil, bundle: nil)
  }
}
```

**FlowController**

Since `FlowController` is just plain `UIViewController`, you don't need to manually manage child `FlowController`. The child `FlowController` is gone when you pop or dismiss. If we want to listen to `UINavigationController` events, we can just handle that inside the `FlowController`

```
final class LoginFlowController: UIViewController {
  private let dependencyContainer: DependencyContainer
  private var embeddedNavigationController: UINavigationController!
  weak var delegate: LoginFlowControllerDelegate?

  init(dependencyContainer: DependencyContainer) {
    self.dependencyContainer = dependencyContainer
    super.init(nibName: nil, bundle: nil)

    embeddedNavigationController = UINavigationController()
    embeddedNavigationController.delegate = self
    add(childController: embeddedNavigationController)
  }
}

extension LoginFlowController: UINavigationControllerDelegate {
  func navigationController(_ navigationController: UINavigationController, willShow viewController: UIViewController, animated: Bool) {

  }
}
```

## 10. FlowController and callback

We can use `delegate` pattern to notify `FlowController` to show another view controller in the flow

```
extension ProductFlowController: ProductListControllerDelegate {
  func productListController(_ controller: ProductListController, didSelect product: Product) {
    let productDetailController = ProductDetailController(
      productNetworkingService: dependencyContainer.productNetworkingService,
      productEditService: dependencyContainer.productEditService,
      imageDownloaderService: dependencyContainer.imageDownloaderService
    )

    productDetailController.delegate = self
    embeddedNavigationController.pushViewController(productDetailController, animated: true)
  }
}
```

Another approach is to use `closure` as callback, as proposed by [@merowing\_](https://twitter.com/merowing_/status/930755494502981632), and also in his post [Improve your iOS Architecture with FlowControllers](http://merowing.info/2016/01/improve-your-ios-architecture-with-flowcontrollers/)

> Using closures as triggers rather than delegate allows for more readable and specialized implementation, and multiple contexts

```
final class ProductFlowController {
  func start() {
    let productListController = ProductListController(
      productNetworkingService: dependencyContainer.productNetworkingService
    )

    productListController.didSelectProduct = { [weak self] product in
      self?.showDetail(for: product)
    }

    embeddedNavigationController.viewControllers = [productListController]
  }
}
```

## 11. FlowController and deep linking

TBD. In the mean while, here are some readings about the UX

- [Navigation with Back and Up](https://developer.android.com/design/patterns/navigation.html)
- [UserNotification](https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate/1649518-usernotificationcenter)
