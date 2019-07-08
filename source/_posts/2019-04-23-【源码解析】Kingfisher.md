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

Swift 5 中新增了 Result 类型，而 Kingfisher 中的 Result 类型和 Swift 5 中的 Result 类型从接口上看是完全一致的。基本用法如下：

```swift
enum NetworkError: Error {
    case invalidUrl
}

func fetchData(urlString: String, completionHandler: ((Result<Int, NetworkError>) -> Void)?) {
    guard let url = URL(string: urlString) else {
        completionHandler?(.failure(.invalidUrl))
        return
    }

    print("Fetching \(urlString)")
    completionHandler?(.success(5))

}

fetchData(urlString: "http://www.baidu.com") { (result) in
    switch result {
    case .failure(let error):
        print(error.localizedDescription)
    case .success(let data):
        print(data)
    }
}
```

除了基本用法外，Result 还有一些其他方法

`public func get() throws -> Success`允许你直接获取结果为成功时的数据，如果结果是失败，则抛出异常

```swift
fetchData(urlString: "http://www.baidu.com") { (result) in
    if let data = try? result.get() {
        print(data)
    }
}
```

Result 中提供了一系列转换结果的方法，

```swift
public func map<NewSuccess>(_ transform: (Success) -> NewSuccess) -> Result<NewSuccess, Failure>
public func mapError<NewFailure>(_ transform: (Failure) -> NewFailure) -> Result<Success, NewFailure> where NewFailure : Error
public func flatMap<NewSuccess>(_ transform: (Success) -> Result<NewSuccess, Failure>) -> Result<NewSuccess, Failure>
public func flatMapError<NewFailure>(_ transform: (Failure) -> Result<Success, NewFailure>) -> Result<Success, NewFailure> where NewFailure : Error
```

这些方法允许你将一种错误或成功的结果数据转换成另一种数据，具体用法：

```swift
enum FactorError: Error {
    case belowMinimum
    case isPrime
}

func generateRandomNumber(maximum: Int) -> Result<Int, FactorError> {
    if maximum < 0 {
        // creating a range below 0 will crash, so refuse
        return .failure(.belowMinimum)
    } else {
        let number = Int.random(in: 0...maximum)
        return .success(number)
    }
}

let result1 = generateRandomNumber(maximum: 11)
let stringNumber = result1.map { "The random number is: \($0)." }
if case .success(let msg) = stringResult {
    print(msg) // The random number is: 11.
}
```

## 4. Kingfisher 缓存策略？

## 5. Kingfisher 异步加载时怎么处理的？
