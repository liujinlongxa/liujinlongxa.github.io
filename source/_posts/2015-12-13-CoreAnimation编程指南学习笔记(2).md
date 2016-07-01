---
layout: post
title: "CoreAnimation编程指南学习笔记(2)"
categories: "读书笔记"
---

## 第六章 动画

### 动画类

* CABasicAnimation提供了在图层的属性值间简单的插入。
* CAKeyframeAnimation提供支持关键帧动画。你指定动画的一个图层属性的关键路径，一个表示在动画的每个阶段的价值的数组，还有一个关键帧时间的数组和时间函数。
* CATransition提供了一个影响整个图层的内容过渡效果。在动画显示过程中采用淡出（fade）、推出(push)、显露(reveal)图层的内容。 常用的过渡效果可以通过提供你自己定制的核心图像滤镜来扩展。

### 隐式动画

隐式动画指所有修改CALayer的可动画属性，默认会都会带有动画。例如`self.layer.position = CGPointMake(200, 200);`默认是带有动画效果的。注意一定是修改layer属性，修改UIView的属性，默认是不带有隐式动画的。而且UIView默认带有的layer属性也是不带隐式动画的。

### 显示动画

```objectivec
	CABasicAnimation *anim = [CABasicAnimation 	animationWithKeyPath:@"opacity"];
    anim.fromValue = @(1.0);
    anim.toValue = @(0.5);
    anim.autoreverses = YES;
    anim.repeatCount = 10;
    anim.duration = 2;
    [self.layer addAnimation:anim forKey:@"anim"];
```

## 第七章 事务

图层的每一个改变都是事务的一部分。隐式分为显示事务和隐式事务。隐式事务由系统自动创建和提交，在修改图层属性时，系统会自动创建一个事务，在下一次运行循环时自动提交。显示事务由用户程序控制创建和提交。一旦创建了显示事务，系统就不会再创建隐式事务。

### 使用显式事务禁止隐式动画

```objectivec
	[CATransaction begin];
    [CATransaction setDisableActions:YES];
    self.layer.position = CGPointMake(200, 200);
    [CATransaction commit];
```

### 修改隐式动画的动画时长

```objectivec
	[CATransaction begin];
    [CATransaction setAnimationDuration:2.0];
    self.layer.position = CGPointMake(200, 200);
    [CATransaction commit];
```

## 第八章 KVC

CoreAnimation是通过KVC来实现对不同字段或关键路径进行动画的，使用方法如下

```objectivec
[theLayer setValue:[NSNumber numberWithInteger:50] forKey:@"someKey"];
```

使用`defaultValueForKey:`可以获取字段的默认值。

CAAnimation提供支持使用关键路径访问选择的结构字段。目前支持的字段如下

![](http://7xn88v.com1.z0.glb.clouddn.com/2015-12-13_10-42-53.jpg)
