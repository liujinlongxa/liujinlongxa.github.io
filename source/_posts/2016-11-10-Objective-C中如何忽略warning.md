---
title: Objective-C中如何忽略warning
categories: iOS Tips
tags:
  - Objective-C
date: 2016-11-10 21:05:51
---

经常会在有些第三方框架里看到这样的代码：

```objectivec
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wgnu"
    NSString *pathToBeMatched = nil;
    if (method && path) {
        pathToBeMatched = [[[self requestWithMethod:(method ?: @"GET") path:path parameters:nil] URL] path];
    }
#pragma clang diagnostic pop
```

这段代码的前两句和最后一句的作用是忽略中间的代码的某种警告（warning），如果我们的代码中有些警告希望被忽略，可以使用这种方法忽略警告。

其中第二句代码的`-Wgnu`是警告类型，可以通过以下方法查看警告类型。

首先Command+b编译代码，然后Command+8来到报告导航栏，选中刚才的那次编译，如下图：

![1](http://7xn88v.com1.z0.glb.clouddn.com/ba9ee4f9e59a474670f251598e7978d6.png)

然后在左侧，找到相应的警告详情，点击左边的![2](http://7xn88v.com1.z0.glb.clouddn.com/f12763686090a1cec73de963526971be.png)按钮可以打开详情，然后就可以在最底部的警告描述里找到警告的类型，如下图：

![3](http://7xn88v.com1.z0.glb.clouddn.com/4d8b40432de09a56e50622b099f002bb.png)

以上就是OC忽略警告的方法。
