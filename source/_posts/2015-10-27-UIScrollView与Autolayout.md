---
layout: post
title: "UIScrollView与Autolayout"
categories: "iOS开发"
---

在Autolayout中应用UIScrollView一直以来都是一个难点或易错点，本文是我在参考了很多资料后，总结的两种在Autolayout中正确使用UIScrollView的方法。本文主要以两个
Demo为例，讲解一下如何在Autolayout中应用UIScrollView，希望对大家能有帮助。

先看一下最终的效果吧！

![image](http://7xn88v.com1.z0.glb.clouddn.com/AutolayoutDemo.gif)

## 使用Storyboard布局

首先创建一个Single View Application项目，然后在Storyboard中完成如下布局。

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-10-27%20at%2022.29.54.png)

第三个Controller就是我们要实现的界面（以下简称主界面），然后向主界面中拖入一个UIScrollView和一个UIView（姑且叫做A），并且设置UIScrollView的约束如下：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-10-27%20at%2022.38.34.png)

设置A的约束如下，注意不要添加底边距，或者高度约束

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-10-27%20at%2022.45.57.png)

**注意，这个A和UIScrollView是同一级的，并不是父子视图的关系。A主要是用来辅助计算UIScrollView中内容的高度的。**

此时设置完成后会有约束冲突（A缺少高度约束），先不用在意。接着向UIScrollView中拖入一个UIView（姑且叫做B），然后设置B的宽度和高度约束与A相等，如下图，可以在左侧
导航视图中拖线设置。

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-10-27%20at%2022.56.15.png)

然后设置B的边界约束如下：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-10-27%20at%2022.38.34.png)

设置完成后，此时的主界面的结构如下图：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-10-27%20at%2022.58.06.png)

**B其实就是作为UIScrollView的contentView存在的，把所有原本是UIScrollView中的子控件都应该放在B中。**

接下里就是布局主要界面，像最终效果图那样，拖入三个UIImageView和一个UILabel，并且设置对应的约束。（此处设置省略，具体可以参考我项目中的设置）注意不要设置UILabel
的高度约束，并且要设置底部两个UIImageView的下边距约束，这样B就会根据显示的内容自适应高度，并且UIScrollView也会根据B的高度设置contentSize。

整个过程过程的关键点就在于A，B两个UIView，A作为一个辅助视图，辅助UIScrollView计算UIScrollView中内容的高度。B的作用是分离UIScrollView和其子控件之间的关系，
使得UIScrollView仅与B产生约束关系。

## 使用Masonry进行布局

下面使用第三方框架Masonry进行布局，完成上述功能。

首先使用CocoaPods安装Masonry，然后在UItableViewController中添加一个cell，并在SB中创建一个新的UIViewController，绑定对应的类AutolayoutViewController，
如下图：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-10-28%20at%2000.05.28.png)

在AutolayoutViewController中添加代码，核心代码如下

```objectivec

- (void)viewDidLoad {
    [super viewDidLoad];

    self.scrollView = [[UIScrollView alloc] init];
    [self.view addSubview:self.scrollView];

    // 把所有子视图都放在containerView中，然后把containerView加在UIScrollView中
    self.containerView = [[UIView alloc] init];
    [self.scrollView addSubview:self.containerView];

    self.topImage = [[UIImageView alloc] init];
    self.topImage.image = [UIImage imageNamed:@"1.JPG"];
    [self.containerView addSubview:self.topImage];

    self.descLabel = [[UILabel alloc] init];
    self.descLabel.numberOfLines = 0;
    self.descLabel.text = @"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa";
    [self.containerView addSubview:self.descLabel];

    self.leftImage = [[UIImageView alloc] init];
    self.leftImage.image = [UIImage imageNamed:@"2.JPG"];
    [self.containerView addSubview:self.leftImage];

    self.rightImage = [[UIImageView alloc] init];
    self.rightImage.image = [UIImage imageNamed:@"3.JPG"];
    [self.containerView addSubview:self.rightImage];

    [self.scrollView mas_updateConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self.view);
    }];

    [self.containerView mas_updateConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self.scrollView);
        make.width.equalTo(self.scrollView);
    }];

    [self.topImage mas_updateConstraints:^(MASConstraintMaker *make) {
        make.top.mas_equalTo(self.containerView).offset(20);
        make.left.mas_equalTo(self.containerView).offset(20);
        make.right.mas_equalTo(self.containerView).offset(-20);
        make.height.mas_equalTo(200);
    }];

    // 注意不要设置descLabel的高度约束
    [self.descLabel mas_updateConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.topImage);
        make.right.equalTo(self.topImage);
        make.top.mas_equalTo(self.topImage.mas_bottom).offset(20);
    }];

    [self.leftImage mas_updateConstraints:^(MASConstraintMaker *make) {
        make.size.equalTo(self.rightImage);
        make.left.equalTo(self.topImage);
        make.right.mas_equalTo(self.rightImage.mas_left).offset(20);
        make.top.mas_equalTo(self.descLabel.mas_bottom).offset(20);
        make.height.mas_equalTo(150);
        make.bottom.mas_equalTo(self.containerView.mas_bottom).offset(-20);
    }];

    [self.rightImage mas_updateConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(self.leftImage);
        make.right.mas_equalTo(self.containerView.mas_right).offset(-20);
    }];

}

```

这里的关键点是那个containerView，它分离了UIScrollView和子视图的关系。

以上就是本人总结的关于在Autolayout中应用UIScrollView的方法，如果有什么错误欢迎指出。

[代码在这里](https://github.com/liujinlongxa/AutolayoutDemo)
