---
title: UITextField
date: 2018-09-25 23:51:53
category: iOS
---

# 修改 UIAlertController 里面的 UITextField 高度

不能直接改 frame.size.height，这样无效，需要添加约束：

swift:

```swift
alertController.addTextField { textField in
    let heightConstraint = NSLayoutConstraint(item: textField, attribute: .height, relatedBy: .equal, toItem: nil, attribute: .notAnAttribute, multiplier: 1, constant: 100)
    textField.addConstraint(heightConstraint)
}
```

objective-c:

```objective-c
NSLayoutConstraint * heightConstraint = [NSLayoutConstraint constraintWithItem:textField attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1 constant:32];
[textField addConstraint: heightConstraint];
```

<!-- more -->
