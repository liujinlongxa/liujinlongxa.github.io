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
