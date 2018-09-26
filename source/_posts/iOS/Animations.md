---
title: Animations
date: 2018-09-25 23:51:53
category: iOS
---

# Autolayout 动画

最简单的方法是修改`NSLayoutConstraint`的`constant`，设置之后调用`layoutIfNeeded`，例如：

```swift
menuHeightConstraint.constant = menuIsOpen ? 200.0 : 60.0
UIView.animate(withDuration: 0.3,
               delay: 0,
               options: .curveEaseIn,
               animations: {
                self.view.layoutIfNeeded()
                },
               completion: nil)
```

<!-- more -->

当涉及到要修改`multiplier`时，则必须先移除掉约束或者禁用，然后再添加新约束：

```swift
titleLabel.superview?.constraints.forEach { constraint in

	if constraint.identifier == "TitleCenterY" {
	    // 禁用旧的约束
		constraint.isActive = false

        // 添加新约束
		let newConstraint = NSLayoutConstraint(
			item: titleLabel,
			attribute: .centerY,
			relatedBy: .equal,
			toItem: titleLabel.superview!,
			attribute: .centerY,
			multiplier: menuIsOpen ? 0.67 : 1.0,
			constant: 5.0
		)
		newConstraint.identifier = "TitleCenterY"
		newConstraint.isActive = true
	}
}
```

# Spring 弹性动画

默认的动画曲线是比较平滑的，Spring 动画的曲线是开始的快，结束的慢。

![image](http://upload-images.jianshu.io/upload_images/690040-85d57be2e57ca3a3.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240)

主要用到下面的方法:

```
UIView.animate(
  withDuration: duration,
  delay: 0.0,
  usingSpringWithDamping: 0.5,
  initialSpringVelocity: 0.0,
  options: [],
  animations: {
    view.beachballConY.constant = view.beachballConY.constant * -1
    view.layoutIfNeeded()
  },
  completion: nil
)
```

注意两个参数：

- dampingRatio：阻尼系数，范围为 0.0 ~ 1.0，数值越小，弹簧振动的越厉害，Spring 的效果越明显，效果对比可以看下图

![image](http://upload-images.jianshu.io/upload_images/690040-a5bda1728541401a.gif?imageMogr2/auto-orient/strip)

- velocity：表示速度，数值越大移动的越快。值为 1.0 时，这个速度为 1 秒钟之内走完整个动画距离的速度。更大或更小的值会让 view 刚到达终点时的速度更大或更小。

![image](http://upload-images.jianshu.io/upload_images/690040-0040e3f098813c75.gif?imageMogr2/auto-orient/strip)

# UIView Transition 过渡动画

主要用到两个函数:

```
// 针对单个视图的动画
UIView.transition(
    with: imageView,
    duration: 1.0,
    options: [
      .curveEaseIn,
      .transitionFlipFromBottom
    ],
    animations: {
      imageView.isHidden = true
    },
    completion: {_ in
      imageView.removeFromSuperview()
    }
)

// 用from视图替换to视图
UIView.transition(
    from: oldView,
    to: newView,
    duration: 0.33,
    options: .transitionFlipFromTop,
    completion: nil)
```

这个函数没有`delay`参数，调用它会立即执行，如果需要延迟执行，需要自己写一个 delay 方法。

# UIView 的属性动画

可用于动画的属性包括：

- 位置和大小(Position & Size): bounds, frame, center
- 转换(Transformation): rotation, scale, translation
- 外观(Appearance): backgroundColor, alpha
