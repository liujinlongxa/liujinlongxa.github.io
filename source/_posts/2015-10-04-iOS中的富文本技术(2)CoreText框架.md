---
layout: post
title: "iOS中的富文本技术(2)CoreText框架"
categories: "iOS开发"
---


## 前言
前一篇介绍了iOS7新出的TextKit，这一篇打算介绍一下更底层的CoreText框架

## 简介
CoreText是iOS3.2推出的一套文字排版和渲染框架，可以实现图文混排，富文本显示等效果。CoreText中的几个重要的概念：
* CTFrameRef：画布，包含多个CTLine
* CTLineRef：每一行就是一个CTLine
* CTRunRef：每一行可以分为多个属性相同的小段，每一个小段就是一个CTRun
如下图展示了这三者的关系：

![这里写图片描述](http://img.blog.csdn.net/20150403235844681)

## 示例
废话不多说，直接看示例，第一个示例展示了一些常规属性的的设置，效果图如下：

![这里写图片描述](http://img.blog.csdn.net/20150404001114995)

代码如下，这里使用了一个自定义View，然后把要展示的内容绘制在view上，注意代码中使用了很多CoreFoundation框架的类（实际上是结构体），需要使用__bridge进行桥接：

```objectivec
#import "DisplayView.h"
@import CoreText;

NSString * str4 = @"asfasfa阿斯顿发生大发撒放大离开家撒旦法按时付款就阿里；双方均asfasdfasfdalkjsflakj阿斯顿发生大发撒旦法asdfasdfaasfdaasa撒旦法；拉斯克奖发了奥斯卡奖罚洛杉矶的法律；看见谁发的阿斯利康就发；了数据库等法律按实际开发；阿里就开始放到了；安家费阿里山科技发达了开始将对方拉开始交电费了卡双方的空间啊发送卡飞机阿里开始就放暑假了罚款就是浪费";

@interface DisplayView ()

@end

@implementation DisplayView

- (void)drawRect:(CGRect)rect {

    [super drawRect:rect];

    CGContextRef context = UIGraphicsGetCurrentContext();

    //变换坐标
    CGContextSetTextMatrix(context, CGAffineTransformIdentity);
    CGContextTranslateCTM(context, 0, self.bounds.size.height);
    CGContextScaleCTM(context, 1.0, -1.0);

    //设置绘制的路径
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathAddRect(path, NULL, self.bounds);

    //创建属性字符串
    NSMutableAttributedString * attStr = [[NSMutableAttributedString alloc] initWithString:str4];

    //颜色
    [attStr addAttribute:(__bridge NSString *)kCTForegroundColorAttributeName value:(__bridge id)[UIColor redColor].CGColor range:NSMakeRange(5, 10)];

    //字体
    UIFont * font = [UIFont systemFontOfSize:25];
    CTFontRef fontRef = CTFontCreateWithName((__bridge CFStringRef)font.fontName, 25, NULL);
    [attStr addAttribute:(__bridge NSString *)kCTFontAttributeName value:(__bridge id)fontRef range:NSMakeRange(20, 10)];

    //空心字
    [attStr addAttribute:(__bridge NSString *)kCTStrokeWidthAttributeName value:@(3) range:NSMakeRange(36, 5)];
    [attStr addAttribute:(__bridge NSString *)kCTStrokeColorAttributeName value:(__bridge id)[UIColor blueColor].CGColor range:NSMakeRange(37, 10)];

    //下划线
    [attStr addAttribute:(__bridge NSString *)kCTUnderlineStyleAttributeName value:@(kCTUnderlineStyleSingle | kCTUnderlinePatternDot) range:NSMakeRange(45, 15)];


    CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString((CFAttributedStringRef)attStr);
    CTFrameRef frame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, attStr.length), path, NULL);

    //绘制内容
    CTFrameDraw(frame, context);

    CFRelease(fontRef);
    CFRelease(frame);
    CFRelease(path);
    CFRelease(framesetter);

}
@end
```

第二个示例展示了一个带表情符号的文本，效果图如下：

![这里写图片描述](http://img.blog.csdn.net/20150404002937477)

先介绍一下CoreText实现表情混排的原理，在简介中介绍过，一个CTLine代表一行，而一个CTLine又由多个CTRun组成，这里实现表情混排的原理其实就是把CTLine中的某一个CTRun替换成空白字符，然后再根据这个CTRun的具体位置，把图片绘制到这个位置上，下面是具体代码：
```objectivec
#import "ImageDisplayView.h"
@import CoreText;

NSString * str5 = @"asfasfa阿斯顿发生大发撒放大离开家撒旦法按时付款就阿里；双方均asfasdfasfdalkjsflakj阿斯顿发生大发撒旦法asdfasdfaasfdaasa撒旦法；拉斯克奖发了奥斯卡奖罚洛杉矶的法律；看见谁发的阿斯利康就发；了数据库等法律按实际开发；阿里就开始放到了；安家费阿里山科技发达了开始将对方拉开始交电费了卡双方的空间啊发送卡飞机阿里开始就放暑假了罚款就是浪费";

@interface ImageDisplayView ()

@property (nonatomic, strong) NSAttributedString * attStr;

@end

//回调方法，返回要替换的CTRun的下边缘距离本行文字下边缘的高度
static CGFloat descentCallback(void *ref)
{
    return 0;
}

//回调方法，返回要替换的CTRun的上边缘距离本行文字上边缘的高度
static CGFloat ascentCallback(void *ref)
{
    return 30;
}

//回调方法，返回要替换的CTRun的宽
static CGFloat widthCallback(void *ref)
{
    return 30;
}

@implementation ImageDisplayView
{
    CTRunDelegateRef _delegate;
}

- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        NSString * name = @"010";
        unichar objectReplacementChar           = 0xFFFC;//占位字符
        NSString *objectReplacementString       = [NSString stringWithCharacters:&objectReplacementChar length:1];
        NSMutableAttributedString *attachText   = [[NSMutableAttributedString alloc]initWithString:objectReplacementString];

        //创建CTRun代理回调
        CTRunDelegateCallbacks callbacks;
        callbacks.version       = kCTRunDelegateVersion1;
        callbacks.getAscent     = ascentCallback;
        callbacks.getDescent    = descentCallback;
        callbacks.getWidth      = widthCallback;

        //给属性字符串添加代理回调
        CTRunDelegateRef delegate = CTRunDelegateCreate(&callbacks, (void *)name);
        NSDictionary *attr = [NSDictionary dictionaryWithObjectsAndKeys:(__bridge id)delegate,kCTRunDelegateAttributeName, nil];
        [attachText setAttributes:attr range:NSMakeRange(0, 1)];
        self.attStr = attachText;
    }
    return self;
}

- (void)drawRect:(CGRect)rect {

    [super drawRect:rect];

    CGContextRef context = UIGraphicsGetCurrentContext();

    //变换坐标
    CGContextSetTextMatrix(context, CGAffineTransformIdentity);
    CGContextTranslateCTM(context, 0, self.bounds.size.height);
    CGContextScaleCTM(context, 1.0, -1.0);

    //设置文本绘制的路径
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathAddRect(path, NULL, self.bounds);

    NSMutableAttributedString * attStr = [[NSMutableAttributedString alloc] initWithString:str5];
    [attStr insertAttributedString:self.attStr atIndex:50];

    CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString((CFAttributedStringRef)attStr);
    CTFrameRef frame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, attStr.length), path, NULL);

    //计算要替换的CTRun的具体位置
    NSArray * lines = (NSArray *)CTFrameGetLines(frame);
    NSUInteger lineCount = lines.count;
    CGPoint lineOrigins[lineCount];
    CTFrameGetLineOrigins(frame, CFRangeMake(0, 0), lineOrigins);
    //循环遍历每一个CTLine中的每一个CTRun，找到要替换的那个CTRun，并且计算它的位置
    for (int i = 0; i < lineCount; i++) {
        CTLineRef line = (__bridge CTLineRef)lines[i];
        CFArrayRef runs = CTLineGetGlyphRuns(line);
        CFIndex runCount = CFArrayGetCount(runs);
        CGPoint lineOrigin = lineOrigins[i];
        CGFloat lineAscent;
        CGFloat lineDescent;
        CTLineGetTypographicBounds(line, &lineAscent, &lineDescent, NULL);
        for (CFIndex k = 0; k < runCount; k++)
        {
            CTRunRef run = CFArrayGetValueAtIndex(runs, k);
            NSDictionary *runAttributes = (NSDictionary *)CTRunGetAttributes(run);
            CTRunDelegateRef delegate = (__bridge CTRunDelegateRef)[runAttributes valueForKey:(id)kCTRunDelegateAttributeName];
            if (nil == delegate)
            {
                continue;
            }

            CGFloat xOffset = CTLineGetOffsetForStringIndex(line, CTRunGetStringRange(run).location, nil);
            CGRect rect = CGRectMake(lineOrigin.x + xOffset, lineOrigin.y - lineDescent, 30, 30);

            //绘制图片
            CGContextDrawImage(context, rect, [UIImage imageNamed:@"010"].CGImage);
        }

    }

    CTFrameDraw(frame, context);

    CFRelease(frame);
    CFRelease(path);
    CFRelease(framesetter);

}

@end
```

## 总结

下面总结一下CoreText的优缺点：
* 优点：功能强大，支持iOS3.2以后的所有系统
* 缺点：纯C的接口，使用复杂，需要自己管理内存

CoreText更加详细的介绍推荐大家可以看看唐巧大神的《iOS开发进阶》第十七章的内容。
