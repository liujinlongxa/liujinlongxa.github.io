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
