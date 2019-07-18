---
title: "SwiftUI:在UIKit中使用SwiftUI"
categories: SwiftUI
tags:
  - SwiftUI
  - Swift
date: 2019-07-19 00:10:33
---

上一篇介绍了如何在 SwiftUI 中使用 UIKit，这篇文章介绍一下如何在 UIKit 中使用 SwiftUIUI。

在 UIKit 中使用 SwiftUI 要比在 SwiftUI 中使用 SwiftUI 简单很多，只需要使用`UIHostingController`对 SwiftUI 的控件进行一下包装就可以，代码如下：

```swift
let vc = UIHostingController(rootView: Text("Hello World"))
```

包装出来的结果是一个`UIViewController`，这样就可以在其他`UIView`或`UIViewController`中使用了。可以将这个`UIViewController`作为一个`childViewController`加到其他的`UIView`或`UIViewController`中去。
