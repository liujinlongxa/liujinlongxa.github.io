---
layout: post
title: "自定义NSOperation的那些事"
categories: "iOS开发"
---

最近在项目中负责一个下载模块，使用的是NSOperation+NSOperationQueue实现的，以前总以为自己对NSOperation已经很熟悉了，也写过很多Demo，但是到真正使用的时候，才发现里面还有很多细节的东西自己没有注意到，这篇博客算是对NSOperation的用法的一个总结。比较简单的NSBlockOperation以及NSInvocationOperation就不介绍，本文主要讲解一下自定义NSOperation的方法以及注意事项。

很多技术Blog在介绍自定NSOperation时，思路都是大同小异，大致可以分为一下几步：

1. 创建一个集成自NSOperation的类
2. 重写NSOperation的main()方法，在main()方法中实现耗时操作
3. 然后使用时创建自定义的NNSOperation对象，把它添加到NSOperationQueue中，这样就可以自动执行了

其实，这只是自定义NSOperation的一种方法，而且是比较简单的一种方法，不需要自己去控制NSOperation的完成，取消等。另外一种方式是重写NSOperation的start方法，这种方法就需要你自己去控制NSOperation的完成，取消，执行等，而且有许多需要注意的地方。下面着重介绍一些第二种方法。

废话不多说，直接上代码：

```objectivec

#import "CustomOperation.h"

@implementation CustomOperation

- (void)start {
    [NSThread sleepForTimeInterval:3.0];
    static NSInteger i = 0;
    NSLog(@"这是一个耗时操作：%@", @(++i));
}

@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {

        NSOperationQueue *queue = [[NSOperationQueue alloc] init];
        queue.maxConcurrentOperationCount = 1;

        CustomOperation *op1 = [[CustomOperation alloc] init];
        CustomOperation *op2 = [[CustomOperation alloc] init];
        CustomOperation *op3 = [[CustomOperation alloc] init];

        [queue addOperation:op1];
        [queue addOperation:op2];
        [queue addOperation:op3];

        [[NSRunLoop mainRunLoop] run];
    }
    return 0;
}
```

上述代码创建了一个NSOperation的子类CustomOperation，然后重写了start方法，在start方法中模拟了一个耗时操作，然后在main方法中创建一个NSOperationQueue并添加三个CustomOperation，执行结果是只有op1被执行，op2和op3一直没有被执行。这是因为，一个NSOperationQueue在判断一个Operation是否执行完成的标志是Operation的`- (BOOL)isFinished`方法返回YES，所以这里需要有一个变量来记录当前Operation执行的状态，修改代码，如下：

```objectivec
#import "CustomOperation.h"

typedef NS_ENUM(NSUInteger, OperationState) {
    OperationStateReady,
    OperationStateExecuting,
    OperationStateFinished,
};

@interface CustomOperation ()

@property (nonatomic, assign) OperationState state;

@end

@implementation CustomOperation

- (instancetype)init {
    if (self = [super init]) {
        _state = OperationStateReady;
    }
    return self;
}

- (BOOL)isFinished {
    return self.state == OperationStateFinished;
}

- (void)start {
    self.state = OperationStateExecuting;
    [NSThread sleepForTimeInterval:3.0];
    static NSInteger i = 0;
    NSLog(@"这是一个耗时操作：%@", @(++i));
    self.state = OperationStateFinished;
}

@end
```

这里先定义个一个表示Operation状态的枚举，共有三种状态：

1. OperationStateReady：准备状态，创建Operation后默认就是准备状态
2. OperationStateExecuting：执行状态，Operation在执行过程中的状态
3. OperationStateFinished：完成状态，Operation执行完成后的状态

然后在main方法中先将状态置为执行状态，然后在耗时操作执行完后，将状态置为完成状态。有了状态之后，重写NSOperation的`- (BOOL)isFinished`方法，当状态为OperationStateFinished时返回YES，否则返回NO。这样，NSOperationQueue就可以知道操作已经执行完成。运行代码，现在三个Operation就可以按顺序执行了。

继续完善代码

```objectivec
@implementation CustomOperation

- (instancetype)init {
    if (self = [super init]) {
        _state = OperationStateReady;
    }
    return self;
}

- (BOOL)isFinished {
    return self.state == OperationStateFinished;
}

- (BOOL)isExecuting {
    return self.state == OperationStateExecuting;
}

- (BOOL)isReady {
    return self.state == OperationStateReady && [super isReady];
}

- (void)cancel {
    if (![self isExecuting] && ![self isFinished]) {
        [super cancel];
    }
}

- (void)start {

    if (![self isReady]) {
        return;
    }

    if ([self isCancelled]) {
        return;
    }

    self.state = OperationStateExecuting;
    [NSThread sleepForTimeInterval:3.0];
    static NSInteger i = 0;
    NSLog(@"这是一个耗时操作：%@", @(++i));
    self.state = OperationStateFinished;
}

@end
```

这样，一个重写start方法的自定义NSOperation就完成了，这是CustomOperation支持Cancel操作，测试代码如下：

```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepool {

        NSOperationQueue *queue = [[NSOperationQueue alloc] init];
        queue.maxConcurrentOperationCount = 1;

        CustomOperation *op1 = [[CustomOperation alloc] init];
        CustomOperation *op2 = [[CustomOperation alloc] init];
        CustomOperation *op3 = [[CustomOperation alloc] init];

        [queue addOperation:op1];
        [queue addOperation:op2];
        [queue addOperation:op3];

        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [op3 cancel];
        });

        [[NSRunLoop mainRunLoop] run];
    }
    return 0;
}
```

运行代码，会发现只执行了op1和op2，达到了我们预期的效果。

这就是一个完整的自定义NSOperation的方法，这里的关键点就是那几个状态。

比较一下重写start函数的方法和重写main函数的方法，重写main函数实现起来更加简单，不需要自己控制Operation的状态，但是重写start函数更加灵活，可以根据自己的需求定制不同的状态。更加完善的实现可以参看[AFNetworking](https://github.com/AFNetworking/AFNetworking)的AFURLConnectionOperation实现，里面有很多值得学习的东西。

参考资料：
[NSOperation](http://nshipster.com/nsoperation/)
