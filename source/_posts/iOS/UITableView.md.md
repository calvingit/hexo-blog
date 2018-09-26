---
title: UITableView.md
date: 2018-09-25 23:51:53
category: iOS
---

# iPad 横屏时 UITableViewCell 无法沾满整行

UITableView 在 iOS9 后新增了一个属性 cellLayoutMarginsFollowReadableWidth，该属性会影响 UITableViewCell 在 iPad 下的显示，且该属性默认为 YES。

当你发现在 iPad 下 UITableViewCell 不能占满整个 UITableView 的宽度时，只需将该属性设置为 NO 即可。

<!-- more -->

在 viewDidLoad 方法里添加以下代码即可：

```
[self.tableView setCellLayoutMarginsFollowReadableWidth:NO];
```
