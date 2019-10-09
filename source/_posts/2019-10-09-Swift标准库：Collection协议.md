---
title: Swift标准库：Collection协议
categories: Swift
tags:
  - Swift
date: 2019-10-09 23:57:12
---

`Collection`协议是 Swift 标准库中一个重要的协议，它定义了集合类型的一些特性。与`Sequence`协议相比，`Collection`协议拥有以下几个特点：

1. 不同于`Sequence`，`Collection`的元素个数是有限
2. 可以重复迭代多次，每次`for-in`迭代都是从头开始迭代。
3. `Collection`协议继承自`Sequence`，可以说`Collection`是一种特殊的`Sequence`。

下面就用一个具体的实例实现一个`Collection`。

这里我们定义一个`Class`用于表示一个班级，`Student`用于表示班级里的一个学生。

```swift
struct Student {
    var name: String
    var age: Int
    var no: Int ///< 学号
}

struct Class: Collection {

    var students: [Student]

    var startIndex: Int {
        return 0
    }

    var endIndex: Int {
        /// 最大学号为1000
        return 1000
    }

    func index(after i: Int) -> Int {
        return i + 1
    }

    subscript(position: Int) -> Student? {
        return students.first(where: {$0.no == position})
    }
}
```

使用如下：

```swift
let s1 = Student(name: "aa", age: 10, no: 100)
let s2 = Student(name: "bb", age: 11, no: 101)
let s3 = Student(name: "cc", age: 11, no: 99)
let class1 = Class(students: [s1, s2, s3])

// 按照学号顺序遍历所有学生
for student in class1 {
    guard let student = student else {
        continue
    }
    print(student.name) // cc aa bb
}
```

实现`Collection`协议有几个关键的地方：

1. `startIndex`与`endIndex`两个计算属性的复杂度应该是 O(1)
2. `index(after:)`这个方法的实现，针对相同的输入必须要有相同的输出

以上就是`Collection`协议的介绍，Swift 标准库中还有很多其他协议，之后会一一介绍。

参考资料：

[Collection](https://developer.apple.com/documentation/swift/collection)
[Everything You Ever Wanted to Know on Sequence & Collection](https://academy.realm.io/posts/try-swift-soroush-khanlou-sequence-collection/)
