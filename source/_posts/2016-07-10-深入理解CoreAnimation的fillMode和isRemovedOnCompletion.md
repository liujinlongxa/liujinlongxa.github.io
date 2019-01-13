---
title: 深入理解CoreAnimation的fillMode和isRemovedOnCompletion
categories: iOS开发
tags:
  - CoreAnimation
date: 2016-07-10 14:34:06
---

一般在使用CoreAnimation写动画是，在定义CAAnimation时，都要加入如下两句来防止动画完成后视图的位置又还原成了原来的位置。

```swift
animation.fillMode = kCAFillModeBoth
animation.isRemovedOnCompletion = false
```

很少有文章能详细说明这两个属性的作用，今天我们就来深究一下这两个属性。

## fillMode

fillMode是一个枚举值，用于表示动画在开始和结束时的状态，这样说可能不太好理解，直接看例子比较直观。

### kCAFillModeRemoved

这个是fillMode的默认值，表示在到达beginTime时才显示动画的第一帧，动画结束时，删除CALayer做的变化

实例代码：

```swift
override func touchesBegan(touches: Set<UITouch>, withEvent event: UIEvent?) {
    let move = CABasicAnimation(keyPath: "position.y")
    move.fromValue = 100
    move.toValue = 500
    move.duration = 1.0
    move.beginTime = CACurrentMediaTime() + 2.0
    move.fillMode = kCAFillModeRemoved
    self.myView.layer.add(move, forKey: nil)
}
```

代码中，我们将一个View的y坐标从100移动到500，历时1s，注意开始时间我设定为了`CACurrentMediaTime() + 2.0`，即添加动画2s后才开始执行动画，这样才能看到效果。运行结果如下：

![kCAFillModeRemoved](http://oldblog.shicishuzhai.com/kCAFillModeRemoved.gif)

可以看到，在鼠标点击触发后，2s之后myView才移动到了动画的初始位置100开始执行动画，并且在动画结束后视图又立刻恢复到原来的位置。

### kCAFillModeBackwards

这个值表示无论是否到达beginTime，动画开始后，立刻显示动画的第一帧，修改代码如下：

```swift
override func touchesBegan(touches: Set<UITouch>, withEvent event: UIEvent?) {
    let move = CABasicAnimation(keyPath: "position.y")
    move.fromValue = 100
    move.toValue = 500
    move.duration = 1.0
    move.beginTime = CACurrentMediaTime() + 2.0
    move.fillMode = kCAFillModeBackwards
    self.myView.layer.add(move, forKey: nil)
}
```

运行效果如下：

![kCAFillModeBackwards](http://oldblog.shicishuzhai.com/kCAFillModeBackwards.gif)

可以看到，鼠标点击后，myView立刻移动到了动画的初始位置100，然后2s后开始执行动画。

### kCAFillModeForwards

这个值表示在到达beginTime时显示动画的第一帧，在动画结束时，保持动画最后一帧的状态，直到动画被删除，修改代码如下：

```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    let move = CABasicAnimation(keyPath: "position.y")
    move.fromValue = 100
    move.toValue = 500
    move.duration = 1.0
    move.beginTime = CACurrentMediaTime() + 2.0
    move.fillMode = kCAFillModeForwards;
    move.isRemovedOnCompletion = false
    move.delegate = self;
    self.myView.layer.add(move, forKey: nil)
}

override func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
    DispatchQueue.main.after(when: .now() + 3) {
        self.myView.layer.removeAllAnimations()
    }
}
```

代码中添加了`move.isRemovedOnCompletion = false`，即动画执行完后不立刻删除动画，默认情况下，动画执行完后会立刻删除动画。`isRemovedOnCompletion`属性设置为false后，就不会立刻删除。然后我们还添加了`animationDidStop`方法，这个是`CABasicAnimation`的代理方法，在动画执行结束时会被调用。我们在这个方法里延时3s删除动画。运行效果如下：

![kCAFillModeForwards](http://oldblog.shicishuzhai.com/kCAFillModeForwards.gif)

可以看到，鼠标点击后，myView等了2s才移动到动画初始位置，动画结束后，等待了3s，myView才恢复到原来的位置。也就是动画结束后会保持最后一帧的状态，直到动画被删除掉。

### kCAFillModeBoth

这个值相当于kCAFillModeBackwards与kCAFillModeForwards的共同合集，即无论是否到达beginTime，动画开始后，立刻显示动画的第一帧，并且在动画结束时保持最后一帧的状态，直到动画被删除。修改代码如下：

```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    let move = CABasicAnimation(keyPath: "position.y")
    move.fromValue = 100
    move.toValue = 500
    move.duration = 1.0
    move.beginTime = CACurrentMediaTime() + 2.0
    move.fillMode = kCAFillModeBoth;
    move.isRemovedOnCompletion = false
    move.delegate = self;
    self.myView.layer.add(move, forKey: nil)
}

override func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
    DispatchQueue.main.after(when: .now() + 3) {
        self.myView.layer.removeAllAnimations()
    }
}
```

运行效果如下：

![kCAFillModeBoth](http://oldblog.shicishuzhai.com/kCAFillModeBoth.gif)

可以看到，鼠标点击后，myView立刻移动到了动画的初始位置，然后动画结束后，等待了3s，myView才恢复到原来的位置。

## 更进一步的思考

以上就是我对fillMode的一个探究，下面再回到文章开头的那段代码

```swift
animation.fillMode = kCAFillModeBoth
animation.isRemovedOnCompletion = false
```

这时，就很明白了，设置fillMode为`kCAFillModeBoth`，可以使视图在动画结束时，仍然停留在动画结束的位置，直到动画被删除。设置`isRemovedOnCompletion`为false，可以使动画在结束后不被删除，这样视图就会永远停留在动画结束的位置。

乍一看没什么问题，但仔细想想还是觉得有点奇怪，动画已经结束了为什么不删除，还占着内存。如果动画少还好，如果动画非常多，每个动画执行完了都不删除的话，那内存岂不是会爆掉，这是不能接受的。有没有办法能够既让视图停留在动画结束的位置，又能在动画结束后删除动画呢？

当然有，直接上代码：

```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    let move = CABasicAnimation(keyPath: "position.y")
    move.fromValue = 100
    move.toValue = 500
    move.duration = 1.0
    move.beginTime = CACurrentMediaTime() + 2.0
    move.fillMode = kCAFillModeBoth;
    move.delegate = self;
    self.myView.layer.add(move, forKey: nil)

    self.myView.layer.position.y = 500
}
```

解决方法就是在视图添加动画后，手动修改视图的位置为动画结束后的最终位置，这样动画结束后，动画就会被删除并且视图会停留在动画结束时的位置。

注意：这里应该使用layer的position，不能直接修改UIView的frame，否则动画效果会不正确。

## 参考资料

[iOS Animations by Tutorials](https://www.raywenderlich.com/store/ios-animations-by-tutorials)
