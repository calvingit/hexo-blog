
---
title: UITableView.md
date: 2018-09-25 23:51:53
category: iOS
---

    
# iPad 横屏时 UITableViewCell 无法沾满整行
UITableView在iOS9后新增了一个属性cellLayoutMarginsFollowReadableWidth，该属性会影响UITableViewCell在iPad下的显示，且该属性默认为YES。

当你发现在iPad下UITableViewCell不能占满整个UITableView的宽度时，只需将该属性设置为NO即可。

在viewDidLoad方法里添加以下代码即可：

```
[self.tableView setCellLayoutMarginsFollowReadableWidth:NO];
```