---
title: 【源码解析】Kingfisher
categories: Swift
tags:
  - Swift
date: 2019-04-23 22:58:39
---

这是一篇关于开源框架[Kingfisher](https://github.com/onevcat/Kingfisher)源码解析的文章。

## 1. `imageView.kf`中`kf`是什么？

`Kingfisher.swift`中定义了`KingfisherCompatible`和`KingfisherCompatibleValue`两个协议，两个协议中都定义了`kf`这个属性，如下：

```swift
extension KingfisherCompatible {
    /// Gets a namespace holder for Kingfisher compatible types.
    public var kf: KingfisherWrapper<Self> {
        get { return KingfisherWrapper(self) }
        set { }
    }
}

extension KingfisherCompatibleValue {
    /// Gets a namespace holder for Kingfisher compatible types.
    public var kf: KingfisherWrapper<Self> {
        get { return KingfisherWrapper(self) }
        set { }
    }
}
```

紧接着一个名为`Image`的类实现了`KingfisherCompatible`协议，`Image`在`Kingfisher.swift`的头部有定义，在 iOS 系统中代表`UIImageView`，如下：

```swift
#if os(macOS)
import AppKit
public typealias Image = NSImage
public typealias View = NSView
public typealias Color = NSColor
public typealias ImageView = NSImageView
public typealias Button = NSButton
#else
import UIKit
public typealias Image = UIImage
public typealias Color = UIColor
#if !os(watchOS)
public typealias ImageView = UIImageView
public typealias View = UIView
public typealias Button = UIButton
#else
import WatchKit
#endif
#endif
```

这样就实现了为`UIImageView`添加`kf`属性。`kf`的定义如下：

```swift
public var kf: KingfisherWrapper<Self> {
    get { return KingfisherWrapper(self) }
    set { }
}
```

`KingfisherWrapper`的定义如下：

```swift
public struct KingfisherWrapper<Base> {
    public let base: Base
    public init(_ base: Base) {
        self.base = base
    }
}
```

这样，`kf`就有一个代表`imageView`的属性`base`，`pring(imageView.kf.base);`打印出来的就是`imageView`。

`KingfisherCompatible`除了被`ImageView`实现，还被`Image`(UIImage),`Button`(UIButton),`WKInterfaceImage`,`CGImage`,`UIApplication`实现，`KingfisherCompatibleValue`被`Data`,`String`,`CGSize`这三个结构体类型实现。

## 2. `imageView.kf.setImage(with: url, options: optionsInfo)`的调用过程是怎样的？

![Kingfisher](http://blog.shicishuzhai.com/Kingfisher.png)

## 3. `Result`类型解析

## 4. Kingfisher 缓存策略？

## 5. Kingfisher 异步加载时怎么处理的？
