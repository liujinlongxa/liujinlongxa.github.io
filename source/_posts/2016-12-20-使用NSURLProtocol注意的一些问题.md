---
title: 使用NSURLProtocol时要注意的一些问题
categories: iOS Tips
tags:
  - 其他
date: 2016-12-20 23:34:03
---

项目中使用到了NSURLProtocol，用于拦截所有发出的请求并做一些处理。具体用法可以参考[这篇文章](http://nshipster.cn/nsurlprotocol/)，我还专门写了一个[Demo](https://github.com/liujinlongxa/NSURLProtocolDemo)。最近在使用过程中遇到了一些问题，记录如下。

## 拦截到的POST请求的HTTPBody为空

最近由于升级了AFNetworking，在使用NSURLProtocol过程中发现了一个问题，就是在拦截到POST请求后，HTTPBody是空的。以前使用旧版本的AFNetworking时是没有这问题的。分析了一下，新版的AFNetworking使用的是NSURLSession，旧版使用的是NSURLConnection，可能是由于这个原因导致的。网上查了一下，还真有这个问题，具体可以看[这个问题](http://stackoverflow.com/questions/36555018/why-is-the-httpbody-of-a-request-inside-an-nsurlprotocol-subclass-always-nil)以及[这里的讨论](https://bugs.webkit.org/show_bug.cgi?id=137299)。

网上有人提出了一种解决方案，就是不要使用HTTPBody，而使用HTTPBodyStream。具体实现如下：

```objectivec
NSMutableURLRequest * request = [[NSMutableURLRequest alloc] initWithURL:url];
request.allHTTPHeaderFields = self.request.allHTTPHeaderFields;
if ([self.request.HTTPMethod isEqualToString:@"POST"]) {
    request.HTTPMethod = @"POST";
    if (!self.request.HTTPBody) {
        uint8_t d[1024] = {0};
        NSInputStream *stream = self.request.HTTPBodyStream;
        NSMutableData *data = [[NSMutableData alloc] init];
        [stream open];

        while ([stream hasBytesAvailable]) {
            NSInteger len = [stream read:d maxLength:1024];
            if (len > 0 && stream.streamError == nil) {
                [data appendBytes:(void *)d length:len];
            }
        }
        request.HTTPBody = [data copy];
        [stream close];
    }
    else {
        request.HTTPBody = self.request.HTTPBody;
    }
}
```

通过这个方法就可以获得HTTPBody的内容。

## `+registerClass:`方法只适用于`sharedSession`

另外一个要注意的地方就是，只用在使用`[NSURLSession sharedSession]`时，注册NSURLProtocol才能使用`+registerClass:`方法，否则就需要使用`NSURLSessionConfiguration`来注册NSURLProtocol，代码如下：

```objectivec
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
configuration.protocolClasses = @[[MySessionURLProtocol class]];
self.session = [NSURLSession sessionWithConfiguration:configuration];
```

因此，对于新版的AFNetworking，由于它使用的不是`sharedSession`，所以就不能简单的通过类方法`+registerClass:`来注册自定义NSURLProtocol，也必须通过`NSURLSessionConfiguration`来设置。代码如下：

```objectivec
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
configuration.protocolClasses = @[[CustomUrlProtocol class]];
AFHTTPSessionManager *sessionManager = [[AFHTTPSessionManager alloc] initWithBaseURL:baseUrl sessionConfiguration:configuration];
[sessionManager GET:url parameters:params progress:^(NSProgress * _Nonnull downloadProgress) {
    // do somting
} success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
    // do somting
} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
    // do somting
}];
```

以上就是使用NSURLProtocol时要注意的两个问题，希望能对大家有所帮助。
