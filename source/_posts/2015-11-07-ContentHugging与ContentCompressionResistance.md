---
layout: post
title: "Content Hugging与Content Compression Resistance"
categories: "iOS开发"
---

最近在使用Autolayout遇到了一个问题，我设置了一个UILabel的宽度约束，希望这个UILabel在显示时，如果内容宽度没有超过约束的宽度，则以约束的宽度显示，如果
内容宽度超过约束的宽度，则以内容的宽度显示，这时，如果只设置宽度约束，UILabel会始终以约束的宽度显示，如果降低宽度约束，则UILabel就会使用以内容的宽度显示，
始终无法达到我想要的效果，于是一番Google，便发现了Content Hugging与Content Compression Resistance两个概念。

## 基本概念

**Content Hugging Priority** 字面意思为内容紧缩，通过改变它的优先级，可以设置当View的大小大于实际内容的大小时，是否将View缩小，使其紧紧包裹住内容

**Content Compression Resistance Priority** 字面意思为阻止内容紧缩，通过改变它的优先级，可以设置当View的大小小于实际内容的大小时，是否将View扩大，使其紧紧包裹内容

##  实例应用

本例将以一个UILabel为例，来说明一下ContentHugging与ContentCompressionResistance的具体设置方法。

首先创建一个空项目，打开SB，拖入一个UILabel，然后设置一下三个约束：

> 1. 水平居中对齐
> 2. 垂直居中对齐
> 3. 宽度为100

为类方便观察，给Label加一个背景颜色，设置完运行如下图：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-11-07%20at%2022.46.17.png)

然后改变宽度约束的优先级为500，如图：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-11-07%20at%2022.15.10.png)

然后就可以开始设置ContentHuggingPriority与ContentCompressionResistancePriority了，设置界面如下：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-11-07%20at%2022.30.26.png)

首先设置水平方向上的ContentHuggingPriority为501，运行项目，显示效果如下：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-11-07%20at%2022.44.05.png)

如果设置水平方向上的ContentHuggingPriority为499，运行项目，显示效果如下：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-11-07%20at%2022.46.17.png)

这是因为当宽度约束的优先级大于ContentHuggingPriority，优先以约束的宽度显示，当宽度约束的优先级小于ContentHuggingPriority时，如果Label的大小
大于内容的大小，则会按实际内容的大小显示。

接下来改变Label的宽度约束值为30，然后运行项目，显示效果如下：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-11-07%20at%2022.44.05.png)

水平方向上ContentCompressionResistancePriority的默认值为750，将其改为499，则显示效果如下：

![image](http://7xn88v.com1.z0.glb.clouddn.com/Screen%20Shot%202015-11-07%20at%2023.02.49.png)

这是因为当宽度约束的优先级大于ContentCompressionResistancePriority，优先以约束的宽度显示，当宽度约束的优先级小于ContentCompressionResistancePriority
时，如果Label的大小小于内容的大小，则会按实际内容的大小显示

除了在SB中设置外，还可以在代码中设置，代码如下：

```objectivec

// 设置水平方向上内容紧缩的优先级为高
[self.label setContentHuggingPriority:UILayoutPriorityDefaultHigh forAxis:UILayoutConstraintAxisHorizontal];

// 设置垂直方向上阻止内容紧缩的优先级为低
[self.label setContentCompressionResistancePriority:UILayoutPriorityDefaultLow forAxis:UILayoutConstraintAxisVertical];

```
