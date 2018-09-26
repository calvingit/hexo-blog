---
title: UIPickerView
date: 2018-09-25 23:51:53
category: iOS
---

# 修改分割线颜色

分割线的高度是 0.5，不是 1

```objective-c
//设置分割线的颜色为白色
for (UIView *subview in self.pickerView.subviews) {
    if (subview.frame.size.height <= 1){
        subview.backgroundColor = [UIColor whiteColor];
    }
}
```

<!-- more -->

# 修改文本的颜色等属性

实现协议`UIPickerViewDelegate`的方法时, 使用下面属性字符串的方法设置标题:

```objective-c
- (NSAttributedString *)pickerView:(UIPickerView *)pickerView attributedTitleForRow:(NSInteger)row forComponent:(NSInteger)component{
  return [[NSAttributedString alloc] initWithString:self.ageStrings[row]
                                 attributes:@{NSForegroundColorAttributeName:[UIColor whiteColor],
                                              NSFontAttributeName:[UIFont systemFontOfSize:36]}];
}
```
