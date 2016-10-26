---
layout: post
title: "iOS中的富文本技术(1)TextKit简介"
categories: "iOS开发"
---

## 前言
 最近项目中用到了图文混排，所以就研究了一下iOS中的富文本，打算把研究的结果分享一下，也是对自己学习的一个总结。初步打算写两篇，这是第一篇，主要介绍iOS7新出的TextKit的简单实用。

## 简介
TextKit是iOS7新推出的文字排版技术，使用TextKit可以很方便的实现富文本、表情混排和图文混排等效果。TextKit中的几个关键的类：
* `NSAttributeString`和`NSMutableAttributeString`：属性字符串和可变属性字符串，这个TextKit中最基础的类，文字中的所有富文本属性都是通过属性字符串来表现出来的
* `NSTextAttachment`：字符串的附件，将图片，可以将图片等内容当做一个附件插入到属性字符串中，可以实现表情混排，链接等效果

## 示例
废话不多说，直接上代码，先看一下效果图：
![这里写图片描述](http://img.blog.csdn.net/20150402235843040)

```objectivec
#import "TextKitEmojiTextVC.h"

NSString * str = @"asfasfa阿斯顿发生大发撒放大离开家撒旦法按时付款就阿里；双方均asfasdfasfdalkjsflakj阿斯顿发生大发撒旦法asdfasdfaasfdaasa撒旦法；拉斯克奖发了奥斯卡奖罚洛杉矶的法律；看见谁发的阿斯利康就发；了数据库等法律按实际开发；阿里就开始放到了；安家费阿里山科技发达了开始将对方拉开始交电费了卡双方的空间啊发送卡飞机阿里开始就放暑假了罚款就是浪费";

@interface TextKitEmojiTextVC ()<UITextViewDelegate>

@property (nonatomic, strong) UITextView * textView;

@end

@implementation TextKitEmojiTextVC

- (void)viewDidLoad {
    [super viewDidLoad];

    self.title = @"普通文字排版&表情混排";


    //初始化textView
    self.edgesForExtendedLayout = UIRectEdgeNone;
    self.textView = [[UITextView alloc] initWithFrame:CGRectMake(20, 20, [UIScreen mainScreen].bounds.size.width - 40, [UIScreen mainScreen].bounds.size.height - 40 - 44 - 20)];
    self.textView.backgroundColor = [UIColor cyanColor];
    self.textView.text = str;
    self.textView.font = [UIFont systemFontOfSize:15];
    self.textView.editable = NO;
    self.textView.delegate = self;
    [self.view addSubview:self.textView];

    //设置常规属性
    [self setupNormalAttribute];

    //链接和表情
    [self setupEmojiAndLink];

}

/**
 *  向文本中添加表情，链接等
 */
- (void)setupEmojiAndLink
{

    NSMutableAttributedString * mutStr = [self.textView.attributedText mutableCopy];

    //添加表情
    UIImage * image1 = [UIImage imageNamed:@"010"];
    NSTextAttachment * attachment1 = [[NSTextAttachment alloc] init];
    attachment1.bounds = CGRectMake(0, 0, 30, 30);
    attachment1.image = image1;
    NSAttributedString * attachStr1 = [NSAttributedString attributedStringWithAttachment:attachment1];
    [mutStr insertAttributedString:attachStr1 atIndex:50];

    //添加表情
    UIImage * image2 = [UIImage imageNamed:@"011"];
    NSTextAttachment * attachment2 = [[NSTextAttachment alloc] init];
    attachment2.bounds = CGRectMake(0, 0, 15, 15);
    attachment2.image = image2;
    NSAttributedString * attachStr2 = [NSAttributedString attributedStringWithAttachment:attachment2];
    [mutStr insertAttributedString:attachStr2 atIndex:100];

    //添加链接
    NSURL * url = [NSURL URLWithString:@"http://www.baidu.com"];
    [mutStr addAttribute:NSLinkAttributeName value:url range:NSMakeRange(70, 10)];

    self.textView.attributedText = [mutStr copy];
}

/**
 *  设置常规属性
 */
- (void)setupNormalAttribute
{
    NSMutableAttributedString * mutStr = [self.textView.attributedText mutableCopy];

    //颜色
    [mutStr addAttribute:NSForegroundColorAttributeName value:[UIColor redColor] range:NSMakeRange(10, 10)];
    //字体
    [mutStr addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:25] range:NSMakeRange(20, 5)];
    //下划线
    [mutStr addAttribute:NSUnderlineStyleAttributeName value:@(NSUnderlineStyleSingle | NSUnderlinePatternDot) range:NSMakeRange(32, 8)];
    //空心字
    [mutStr addAttribute:NSStrokeWidthAttributeName value:@(2) range:NSMakeRange(42, 5)];
    self.textView.attributedText = [mutStr copy];
}

/**
 *  点击图片触发代理事件
 */
- (BOOL)textView:(UITextView *)textView shouldInteractWithTextAttachment:(NSTextAttachment *)textAttachment inRange:(NSRange)characterRange
{
    NSLog(@"%@", textAttachment);
    return NO;
}

/**
 *  点击链接，触发代理事件
 */
- (BOOL)textView:(UITextView *)textView shouldInteractWithURL:(NSURL *)URL inRange:(NSRange)characterRange
{
    [[UIApplication sharedApplication] openURL:URL];
    return YES;
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end
```

图片环绕效果图：
![这里写图片描述](http://img.blog.csdn.net/20150403000308303)

```objectivec
#import "TextKitImageTextVC.h"

NSString * str1 = @"asfasfa阿斯顿发生大发撒放大离开家撒旦法按时付款就阿里；双方均asfasdfasfdalkjsflakj阿斯顿发生大发撒旦法asdfasdfaasfdaasa撒旦法；拉斯克奖发了奥斯卡奖罚洛杉矶的法律；看见谁发的阿斯利康就发；了数据库等法律按实际开发；阿里就开始放到了；安家费阿里山科技发达了开始将对方拉开始交电费了卡双方的空间啊发送卡飞机阿里开始就放暑假了罚款就是浪费asfasfa阿斯顿发生大发撒放大离开家撒旦法按时付款就阿里；双方均asfasdfasfdalkjsflakj阿斯顿发生大发撒旦法asdfasdfaasfdaasa撒旦法；拉斯克奖发了奥斯卡奖罚洛杉矶的法律；看见谁发的阿斯利康就发；了数据库等法律按实际开发；阿里就开始放到了；安家费阿里山科技发达了开始将对方拉开始交电费了卡双方的空间啊发送卡飞机阿里开始就放暑假了罚款就是浪费asfasfa阿斯顿发生大发撒放大离开家撒旦法按时付款就阿里；双方均asfasdfasfdalkjsflakj阿斯顿发生大发撒旦法asdfasdfaasfdaasa撒旦法；拉斯克奖发了奥斯卡奖罚洛杉矶的法律；看见谁发的阿斯利康就发；了数据库等法律按实际开发；阿里就开始放到了；安家费阿里山科技发达了开始将对方拉开始交电费了卡双方的空间啊发送卡飞机阿里开始就放暑假了罚款就是浪费";

@interface TextKitImageTextVC ()

@property (nonatomic, strong) UITextView * textView;

@end

@implementation TextKitImageTextVC

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.

    self.title = @"文字环绕";

    //初始化textLabel
    self.edgesForExtendedLayout = UIRectEdgeNone;
    self.textView = [[UITextView alloc] initWithFrame:CGRectMake(20, 20, [UIScreen mainScreen].bounds.size.width - 40, [UIScreen mainScreen].bounds.size.height - 40 - 44 - 20)];
    self.textView.backgroundColor = [UIColor cyanColor];
    self.textView.text = str1;
    self.textView.font = [UIFont systemFontOfSize:15];
    self.textView.editable = NO;
    [self.view addSubview:self.textView];

    //图文混排，设置图片的位置
    UIImageView * imageView = [[UIImageView alloc] initWithFrame:CGRectMake(120, 120, 100, 110)];
    imageView.image = [UIImage imageNamed:@"011"];
    imageView.backgroundColor = [UIColor redColor];
    [self.view addSubview:imageView];
    CGRect rect = CGRectMake(100, 100, 100, 100);

    //设置环绕的路径
    UIBezierPath * path = [UIBezierPath bezierPathWithRect:rect];
    self.textView.textContainer.exclusionPaths = @[path];

}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}
@end
```

## 总结
TextKit极大的简化了文字排版的复杂度，但是它的缺点也很明显，就是只能在iOS7之后的系统中使用，很多需要兼容iOS7以前的系统的应用都没法使用。不过随着技术的发展，相信它的应用也会越来越广泛。
