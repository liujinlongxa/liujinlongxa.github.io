---
title: 如何写出一个优雅的UserDefaults扩展
categories: Swift
tags:
  - Swift
date: 2019-10-07 11:00:21
---

`UserDefaults`是 iOS 中非常常用的一种数据持久化方式，在实际开始中我们一般不会直接使用系统提供的 API 去操作`UserDefaults`，而是会创建一些扩张，利用扩展的方法来使用`UserDefaults`。利用 Swift 语言的一些特性，我们可以写出非常优化的`UserDefaults`扩展。

利用 Swift 的泛型以及`subscript`，可以实现一个不需要关心存储数据类型的`UserDefaults`扩展，代码如下：

```swift
extension UserDefaults {
    struct Key<Value> {
        var name: String
    }

    subscript<T>(key: Key<T>) -> T? {
        get {
            return value(forKey: key.name) as? T
        }
        set {
            setValue(newValue, forKey: key.name)
        }
    }

    subscript<T>(key: Key<T>, default defaultProvider: @autoclosure () -> T) -> T {
        get {
            return value(forKey: key.name) as? T ?? defaultProvider()
        }
        set {
            setValue(newValue, forKey: key.name)
        }
    }
}
```

然后使用另外一个扩展定义 Key

```swift
extension UserDefaults.Key {
    static var bookmarks: UserDefaults.Key<[String]> {
        return .init(name: "bookmarks")
    }

    static var likeCount: UserDefaults.Key<Int> {
        return .init(name: "likeCount")
    }
}
```

这样，在使用时就不需要关心类型了

```swift
UserDefaults.standard[.likeCount] = 12
let bookmarks = UserDefaults.standard[.bookmarks, default: []]
```

Swift5 中新增了`Property Wrappers`特性，利用`Property Wrappers`也可以使得`UserDefaults`的使用变得非常优雅：

```swift
struct UserDefault<T> {
    let key: String
    let defaultValue: T

    init(_ key: String, defaultValue: T) {
        self.key = key
        self.defaultValue = defaultValue
    }

    var wrappedValue: T {
        get {
            return UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
        }
        set {
            UserDefaults.standard.set(newValue, forKey: key)
        }
    }
}
```

定义`UserDefaults`的 Key

```swift
struct UserDefaultsConfig {
    @UserDefault("likeCount", defaultValue: 12)
    static var likeCount: Int
}
```

在使用时就可以直接赋值

```swift
UserDefaultsConfig.likeCount = 13
```

上面两种方式唯一不同的点是，第一种方式每次都可以指定不同的`defaultValue`，而第二种方式只能在定义属性时指定一次`defaultValue`，使用时不能动态修改`defaultValue`。

个人建议两种方式取其一即可，如果不需要动态修改`defaultValue`，个人建议使用第二种方式，应为更加简洁明了。

参考资料：

[Property wrappers to remove boilerplate code in Swift](https://www.avanderlee.com/swift/property-wrappers/)
[Property Wrappers](https://github.com/DougGregor/swift-evolution/blob/property-wrappers/proposals/0258-property-wrappers.md)
[The power of subscripts in Swift](https://www.swiftbysundell.com/articles/the-power-of-subscripts-in-swift/)
