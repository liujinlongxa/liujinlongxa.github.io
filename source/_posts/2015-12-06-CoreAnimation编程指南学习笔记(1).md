---
layout: post
title: "CoreAnimation编程指南学习笔记(1)"
categories: "读书笔记"
---

## 第一章：核心动画概念

CoreAnimation框架中类的分类

* 提供显示内容的图层类
* 动画和计时类
* 布局和约束类
* 事务类，在原子更新的时候组合图层类


### 图层类（Layer Classes）

CALayer是所有层类的父类，几个常见的图层子类：CAScrollLayer,CATextLayer,CATiledLayer等

### 动画类和计时类

CAAnimation是动画类的基类，他实现了CAMediaTiming协议，提供了动画的持续时间，速度，和重复计数等。他也实现了CAAction协议，该协议为图层触发一个动画动作提供了标准化响应。

动画类同时定义了一个使用贝塞尔曲线来描述动画改变的时间函数。

* CATransition：提供了一个图层改变的过渡效果，他能影响图层的整个内容。
* CAAnimationGroup:组动画
* CAPropertyAnimation:属性动画
* CABasicAnimation:基础动画
* CAKeyframeAnimation:关键帧动画

### 布局管理类

CAConstraint类是一个布局管理类，他可以指定子图层类限制于你指定的约束集合。

### 事务管理类

CATransaction是核心动画里面负责协调多个动画原子更新显示操作。事务支持嵌套使用。核心动画支持两种事务：隐式事务和显式事务。

## 第二章：渲染架构

每个可见的图层树由两个相应的树组成，一个是呈现树，一个是渲染树。修改图层的一个属性，图层树里相应的值会被马上更新，但是呈现树里面的值在将要显示给用户的时候才会被更新为新值。

## 第三章：几何变换

anchorPoint:锚点，他指定了bounds相对于position的值，同时也作为变换时候的支点。在Autolayout设置约束时，锚点的值也会影响视图显示的位置。

几何变换：CATransform3D，可以使用变换函数进行变换，例如CATransform3DMakeTranslation，CATransform3DTranslate等，也可以通过键值路径修改变换。

## 第四章：图层树结构

图层树与视图的结构很类似：

* 每个图层定义了一个基于其父图层的坐标系的坐标系，当一个图层变换的时候，他的子图层同样变换
* 一个动态的图层树，可以在程序运行的时候重新设置，图层可以添加，也可以删除

ios中，每个UIView的实例会自动创建一个CALayer类的实体，视图为图层提供了底层的事件处理，图层为视图提供了显示的内容。

## 第五章：图层的内容

使用CALayer无需创建继承子类，直接设置CALayer的contents属性即可，contents属性是一个CGImageRef类型，这样子创建比使用UIImageView效率要高

```objectivec
self.myLayer = [[CALayer alloc] init];
self.myLayer.bounds = CGRectMake(0, 0, 100, 100);
self.myLayer.position = CGPointMake(100, 100);
self.myLayer.backgroundColor = [UIColor redColor].CGColor;
UIImage *image = [UIImage imageNamed:@"1.JPG"];
self.myLayer.contents = (__bridge id _Nullable)(image.CGImage);
[self.view.layer addSublayer:self.myLayer];
```


也可以通过代理方法，提供CALayer的内容，有两个代理方法：`- (void)displayLayer:(CALayer *)theLayer`与`- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx`, 要通知系统重绘图层，只需要调用CALayer的`setNeedsDisplay`或者`setNeedsDisplayInRect:`方法或者把图层的needsDisplayOnBoundsChange属性值设置为YES即可。


```objectivec
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx {
    CGMutablePathRef thePath = CGPathCreateMutable();
    CGPathMoveToPoint(thePath,NULL,15.0f,15.f);
    CGPathAddCurveToPoint(thePath,
                          NULL,
                          15.f,250.0f,
                          295.0f,250.0f,
                          295.0f,15.0f);
    CGContextBeginPath(ctx);
    CGContextAddPath(ctx, thePath );
    CGContextSetLineWidth(ctx, 5);
    CGContextStrokePath(ctx);
    // release the path
    CFRelease(thePath);
}

- (void)displayLayer:(CALayer *)layer {
    UIImage *image = [UIImage imageNamed:@"1.JPG"];
    layer.contents = (__bridge id _Nullable)(image.CGImage);
}
```


除了设置代理方法，也可以通过继承CALayer，重写`display`方法或`- (void)drawInContext:(CGContextRef)theContext`方法来自定义图层

CALayer的contentsGravity属性相当于UIView的contentMode属性。
