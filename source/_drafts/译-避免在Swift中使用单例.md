---
title: (译)避免在Swift中使用单例
categories: Swift
tags:
  - Swift
  - 单例
  - 翻译
date: 2019-10-28 23:06:51
---

> 原文地址：[Avoiding singletons in Swift](https://www.swiftbysundell.com/articles/avoiding-singletons-in-swift/)

"我知道单例不好，但是..."，这是开发者在讨论代码时经常会说的一句话。单例模式用多了不好似乎已经成为了一种社区的共识，但与此人同时，包括苹果以及诸多第三方开发者在他们的内部应用以及开源框架中都在使用单例模式。

这周，让我们来看一下使用单例模式会遇到哪些问题以及如何通过一些技术来避免这些问题。让我们开始吧。

## 为什么单例模式这么流行

首页，我们要讨论一下为什么单例模式会这么流行。如果大多数开发者都认为单例模式是有害的，为什么还要持续不断的使用单例模式。

我想主要有两方面的原因。第一，我认为开发者在开发苹果平台应用时不断的使用单例模式的一个最大原因是因为苹果自己也在大量使用单例模式。作为第三方开发者，我们经常会期待苹果定义出所谓的“最佳实践”，任何苹果自己常用的的设计模式也都会在整个社区广泛传播。

第二点，我认为是便捷性。单例通常可以用作访问某些核心值或对象的捷径，因为它们基本上可以从任何地方被访问。看一下这个例子，我们想在`ProfileViewController`中展示当前登录的用户的名字，并且可以点击按钮退出当前用户。

```swift
class ProfileViewController: UIViewController {
    private lazy var nameLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        nameLabel.text = UserManager.shared.currentUser?.name
    }

    private func handleLogOutButtonTap() {
        UserManager.shared.logOut()
    }
}
```

完成像上面这样的任务，非常方便（而且很常见）的做法就是使用`UserManager`这样一个单例来封装用户和账号的处理逻辑。那么单例模式究竟有什么缺点呢？🤔

## 单例模式的缺点

在讨论诸如模式和架构之类的问题时，我们很容易陷入过于理论化的陷阱。使我们的代码在理论上是“正确的“并遵循所有最佳实践和原则，这是很好的做好。但现实往往会打击我们，我们需要在两者之间寻求某种平衡。

那么单例模式通常会引起什么具体问题，为什么要避免这些问题？ 我倾向于避免使用单例模式的三个主要原因是：

- **它们是全局可变的共享状态。**它们的状态会自动在整个应用程序中共享，通常很多 Bug 都是由于程序状态发生意外改变而导致的。
- **单例和依赖于它们的代码之间的关系通常没有很好地定义。**由于单例非常方便且全局可访问，大量使用单例模式通常会导致对象之间耦合太重，从而导致代码很难维护。
- **管理单例对象的生命周期通常很困难。** 由于单例对象在应用程序的整个生命周期中都不会被销毁，因此对其进行管理可能非常困难，并且它们通常会依赖很多变量。 这也使得依赖单例对象的代码变得很难测试，因为你很难使得每一个测试用例都从”干净的初始状态“开始执行。

在之前的 `ProfileViewController` 示例中，我们已经可以看到这 3 个问题的迹象了。`ProfileViewController`需要依赖`UserManager`中的可选属性`currentUser`，但是我们无法保证在编译时`currentUser`一定有值，也就是当`ProfileViewController`视图控制器展现时是否一定有数据。这种情况听起来像一个随时都可能发生的 Bug 😬！

## 依赖注入

与其让`ProfileViewController`依赖单例对象，不如将这些对象注入到`ProfileViewController`的初始化方法中。 在这里，我们将当前用户作为非可选对象，和一个用于执行注销操作的`LogOutService`对象一起注入到`ProfileViewController`中。

```swift
class ProfileViewController: UIViewController {
    private let user: User
    private let logOutService: LogOutService
    private lazy var nameLabel = UILabel()

    init(user: User, logOutService: LogOutService) {
        self.user = user
        self.logOutService = logOutService
        super.init(nibName: nil, bundle: nil)
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        nameLabel.text = user.name
    }

    private func handleLogOutButtonTap() {
        logOutService.logOut()
    }
}
```

上面的代码看上去更加清晰，更易于管理。 现在，我们的代码可以安全地访问始终存在的对象，并且登出操作也有了很清晰的 API。 通常，为了使程序的核心对象之间的关系更加清晰，将各种单例和管理器重构为清晰的相互分离的服务是一种很好方法。

## 服务

作为示例，让我们仔细看看`LogOutService`是如何实现。 为了完成“注销”这一个操作，它依然使用依赖项注入的方式提供了一个清晰明确的 API。

```swift
class LogOutService {
    private let user: User
    private let networkService: NetworkService
    private let navigationService: NavigationService

    init(user: User,
         networkService: NetworkService,
         navigationService: NavigationService) {
        self.user = user
        self.networkService = networkService
        self.navigationService = navigationService
    }

    func logOut() {
        networkService.request(.logout(user)) { [weak self] in
            self?.navigationService.showLoginScreen()
        }
    }
}
```

## 封装

从大量使用单例转变为充分利用服务，依赖项注入和本地状态可能需要花费很多时间，有时甚至需要巨大的重构。

幸运的是，我们可以运用[Testing Swift code that uses system singletons in 3 easy steps](https://www.swiftbysundell.com/articles/testing-swift-code-that-uses-system-singletons-in-3-easy-steps)提到的技术，使我们能够以更简单的方式摆脱单例。 就像在许多其他情况下一样-协议可以挽救！

无需一次性重构所有单例并创建新的服务类，我们可以简单地将服务定义为协议，如下所示：

```swift
protocol LogOutService {
    func logOut()
}

protocol NetworkService {
    func request(_ endpoint: Endpoint, completionHandler: @escaping () -> Void)
}

protocol NavigationService {
    func showLoginScreen()
    func showProfile(for user: User)
    ...
}
```

然后，我们可能轻松的将单例改造为实现了这些协议的服务类。 在许多情况下，我们甚至不需要进行任何改动，只需将其共享实例作为服务传递即可。

同样的技术也可以用于改造我们应用程序中的其他”类似单例“的核心对象，例如使用 AppDelegate 进行导航。

```swift
extension UserManager: LoginService, LogOutService {}

extension AppDelegate: NavigationService {
    func showLoginScreen() {
        navigationController.viewControllers = [
            LoginViewController(
                loginService: UserManager.shared,
                navigationService: self
            )
        ]
    }

    func showProfile(for user: User) {
        let viewController = ProfileViewController(
            user: user,
            logOutService: UserManager.shared
        )

        navigationController.pushViewController(viewController, animated: true)
    }
}
```

现在，我们可以通过使用依赖项注入和服务的方式使所有视图控制器都不再依赖单例，而无需进行大量的重构！ 然后，我们可以使用[Replacing legacy code using Swift protocols](https://www.swiftbysundell.com/articles/replacing-legacy-code-using-swift-protocols/)中提到的技术逐一替换代码中的单例和其他类似单例的类型。

## 总结

单例并不总是有问题的，但是在许多情况下，单例所导致的问题都可以通过在对象之间使用依赖注入的方式来避免这些问题。

如果您正在开发的项目中大量使用了单例模式，并且遇到一些相关的问题，那么希望这篇文章能对您有所启发。
