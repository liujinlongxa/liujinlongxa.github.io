---
title: 如何获取WebView的UserAgent
categories: iOS Tips
tags:
  - 其他
date: 2016-10-25 11:46:04
---

项目中用到了WebView的UserAgent，可以通过以下方式获取到：

```objectivec
UIWebView* webView = [[UIWebView alloc] init];
NSString* secretAgent = [webView stringByEvaluatingJavaScriptFromString:@"navigator.userAgent"];
```

如果是WKWebView，可以通过以下方式获取

```objectivec
WKWebView *webView = [[WKWebView alloc] init];
[webView evaluateJavaScript:@"navigator.userAgent" completionHandler:^(id _Nullable userAgent, NSError * _Nullable error) {
    NSLog(@"userAgent: %@", userAgent);
}];
self.webView = webView; // 注意，一定要对webView进行强引用
```

获取到的值的为：

```
Mozilla/5.0 (iPhone; CPU iPhone OS 10_0_1 like Mac OS X) AppleWebKit/602.1.50 (KHTML, like Gecko) Mobile/14A403
```
