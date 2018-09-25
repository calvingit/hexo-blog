
---
title: UISwitch
date: 2018-09-25 23:51:53
category: iOS
---

    
# 修改开关的背景色

修改开的背景色：

```
mSwitch.onTintColor = onColor
```

修改关的背景色：

```
mSwitch.tintColor = offColor
mSwitch.layer.cornerRadius = mSwitch.frame.height / 2
mSwitch.backgroundColor = offColor
```