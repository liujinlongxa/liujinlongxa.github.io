---
layout: post
title: "iOS运行时初探 使用运行时机制向Category中添加属性"
categories: "iOS开发"
---

##前言
了解OC的都应该知道，在一般情况下，我们是不能向Category中添加属性的，只能添加方法，但有些情况向，我们确实需要向Category中添加属性，而且很多系统的API也有一些在Category添加属性的情况，例如我们属性的`UITableView`的`section`和`row`属性，就是定义在一个名为`NSIndexPath`的分类里的，如下

![这里写图片描述](http://img.blog.csdn.net/20150531230308155)

那这到底是怎么实现的呢？

##iOS运行时机制简介
iOS运行时机制，简单来说，就是苹果给开发这提供的一套在运行时动态创建类、添加属性/方法（不止这些，还有一些其他功能）的API，它是一套纯C语言的API，使用相应的API就可以通过Category给一个原本存在的类添加属性。

##实例

```objectivec
#import <Foundation/Foundation.h>
#import <objc/runtime.h>

@interface NSObject (CategoryWithProperty)

/**
 *  要在Category中扩展的属性
 */
@property (nonatomic, strong) NSObject *property;

@end

@implementation NSObject (CategoryWithProperty)

- (NSObject *)property {
    return objc_getAssociatedObject(self, @selector(property));
}

- (void)setProperty:(NSObject *)value {
    objc_setAssociatedObject(self, @selector(property), value, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

@end
```

这样就可以在Category中添加属性了。
