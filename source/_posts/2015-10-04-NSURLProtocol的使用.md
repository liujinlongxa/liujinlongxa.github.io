---
layout: post
title: "NSURLProtocol的使用"
categories: "iOS开发"
---

###NSURLProtocol的使用

NSURLProtocol可以拦截所有由[URL Loading System](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/10000165i)发起的请求，可用于实现以下功能：

 * 重定向网络请求，实现代理等功能
 * 实现网络缓存
 * 全局设置网络请求
 * 自定义网络请求返回结果

具体的实现步骤如下：

**第一步** 创建NSURLProtocol的子类，重写以下方法

`+ (BOOL)canInitWithRequest:(NSURLRequest *)request`这个方法用来返回是否需要处理这个请求，如果需要处理，返回YES，否则返回NO。在该方法中可以对不需要处理的请求进行过滤。

`+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request`重写该方法，可以对请求进行修改，例如添加新的头部信息，修改，修改url等，返回修改后的请求。

`+ (BOOL)requestIsCacheEquivalent:(NSURLRequest *)a toRequest:(NSURLRequest *)b`该方法主要用来判断两个请求是否是同一个请求，如果是，则可以使用缓存数据，通常只需要调用父类的实现即可

`- (void)startLoading`重写该方法，需要在该方法中发起一个请求，对于NSURLConnection来说，就是创建一个NSURLConnection，对于NSURLSession，就是发起一个NSURLSessionTask

`- (void)stopLoading`重写该方法，需要停止响应的请求

**第二步** 实现相应的协议代理方法，对NSURLConnection来说，要实现NSURLConnectionDataDelegate代理方法，对于NSURLSession来说，需要实现NSURLSessionTaskDelegate代理方法。在这些代理方法中，需要使用NSURLProtocolClient来通知URL Loading System。具体实现方式详见示例代码。

**第三步** 在合适的位置注册自定义的NSURLProtocol子类。调用`[NSURLProtocol registerClass:[MyURLProtocol class]]`进行注册，调用`[NSURLProtocol unregisterClass:[MyURLProtocol class]]`可以注销。注意，代码中可以注册多个NSURLProtocol子类，每个子类可以

以上就是整个实现过程，详细内容可以参见代码。

代码地址:[NSURLProtocolDemo](https://github.com/liujinlongxa/NSURLProtocolDemo)
