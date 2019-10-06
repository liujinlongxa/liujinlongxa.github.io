---
title: Swift标准库：Sequence协议
categories: Swift
tags:
  - Swift
date: 2019-10-06 20:36:45
---

Sequence 协议为类型提供了迭代访问的能力，凡是实现了 Sequence 的类型，都快进行迭代访问，即使用`for-in`语句进行访问，标准库中的`Array`, `Dictionary`, `Set`等类型都实现了 Sequence 协议。下面介绍一下自定义类型如何实现 Sequence 协议。

实现 Sequence 协议的集合有以下两个特点：

1. 集合的个数可以是有限的也可以是无限的
2. 迭代过程中元素不能重复

```swift
struct Countdown: Sequence, IteratorProtocol {
    var count: Int

    mutating func next() -> Int? {
        if count == 0 {
            return nil
        } else {
            defer { count -= 1 }
            return count
        }
    }
}
```

使用的时候就可以使用`for-in`语句：

```swift
let threeToGo = Countdown(count: 3)
for i in threeToGo {
    print(i)
}
```

对于更为复杂的例子，需要实现自定义的`Iterator`，下面的例子中实现了一个自定义的链表，并且自定义了迭代器`LinkedListIterator`，这样链表就可以使用`for-in`语句进行访问

```swift
class LinkedList<E: Equatable> {
    class Node {
        public var e: E?;
        public var next: Node?;
        init(e: E?, next: Node?) {
            self.e = e
            self.next = next
        }
    }

    struct LinkedListIterator: IteratorProtocol {
        typealias Element = E
        var head: Node?
        mutating func next() -> E? {
            if let next = head?.next {
                defer { head = head?.next }
                return next.e
            } else {
                return nil
            }
        }
    }

    private var dummyHead: Node?
    private var size: Int

    init() {
        dummyHead = Node.init()
        size = 0
    }

    // other method...

}

// 实现Sequence协议
extension LinkedList: Sequence {
    __consuming func makeIterator() -> LinkedListIterator {
        return LinkedListIterator(head: self.dummyHead)
    }
}

let list = LinkList<Int>(arrayLiteral: 1, 2, 3, 4)
for item in list {
    print(item)
}

```

参考资料：

[Sequence](https://developer.apple.com/documentation/swift/sequence)
[Everything You Ever Wanted to Know on Sequence & Collection](https://academy.realm.io/posts/try-swift-soroush-khanlou-sequence-collection/)
