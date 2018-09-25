---
title: Mac 系统
date: 2018-09-25 23:51:53
category: Softwares
---

# iMac 关闭显示器

快捷键：Ctrl + Shift + 光驱按钮（右上角）

# 截图

- 基础快捷键（+表示同时操作、-表示分步操作） - Cmd+Shift+3：全屏截图；截取整个屏幕，保存截图至桌面文件夹。 - Cmd+Shift+4：区域截图；鼠标光标变成带坐标的小十字，通过拖拽截取特定区域，保存截图至桌面文件夹。 - Cmd+Shift+4 - 单击空格键 - 鼠标单击指定窗口：窗口截图；出现小十字光标后对指定窗口按空格键，鼠标光标变成照相机，鼠标单击截取指定窗口，保存截图至桌面文件夹。

<!-- more -->

- 进阶快捷键 - Cmd+Shift+4 - 按住空格键拖动鼠标：区域截图；选取区域范围后，按住空格键并拖到鼠标可移动选取范围，释放按键后保存截图至桌面文件夹。 - Cmd+Shift+4 - 按住 Shift - 上下/左右移动鼠标：区域截图；选取区域范围后，按住 Shift 并上下/左右移动鼠标即可固定选取范围宽度/高度改变高度/宽度，释放按键后保存截图至桌面文件夹。 - Cmd+Shift+4 - 按住 Shift 和 Option: 区域截图；选取区域范围后，按住 Shift 和 Option 键在上一快捷键功能的基础上自由切换高度宽度，释放按键后保存截图至桌面文件夹。 - Cmd+Shift+4 - 按住 Option: 区域截图；选取区域范围后，按住 Option 键可按比例缩放选取范围，释放按键后保存截图至桌面文件夹。

> 以上介绍的快捷键配合按住 Control 键即可将截图文件保存在剪切板，以供调用。

# 去掉修改文件扩展名时的警告

在`Finder`的`偏好设置`——`高级`去掉勾选`更改扩展名之前显示警告`。

# Spotlight 进行单位计算

在做 App 的本地化时，需要对不同单位进行数值计算，如果使用计算器就太麻烦了，可以使用快捷键`Cmd + 空格`，打开 Spotlight 输入框，输入你的数据即可，立马出现结果。

其实用百度框计算也可以。

# 批量重命名文件

选中所有文件，右键——`给X个项目重新命名...`，修改需要的格式即可。

# 使用“预览”编辑图片

如果是轻量级的编辑图片，其实不需要 Photoshop、Sketch 之类的，使用系统的“预览”应用就可以了。打开图片之后，打开“工具——显示标记工具栏”。

# 使用 QuickTime 录制 iPhone 屏幕

首先用数据线连接 iPhone，打开 QuickTime，菜单栏选择`文件`——`新建影片录制`，然后点击小红点旁边的向下箭头，选择你的相机、麦克风等。

# 通过 AirDrop 发送文件到 iPhone

如果你只是发文件到 iPhone，千万别用什么 QQ 或微信了，太麻烦。使用 AirDrop 就可以做到了，前提是 mac 和 iPhone 都打开这个选项了。

使用方法：右键文件——`共享`——`AirDrop`，弹框里点选头像即可发送，完成之后 iPhone 端会弹出提示。

# 粘贴文本并匹配格式

经常复制网页的东西到邮件或者备忘录，直接使用`Cmd + V`会带有原始文本的格式，比如字体大小、颜色、下划线等，这些格式放在纯文本里看起来不友好。想要匹配格式粘贴，使用快捷键 `Option + Shift + Cmd + V`。

# 使用 Wireshark 抓包 iOS 设备的网络数据

方法如下：

> 用数据线将 iOS 设备连上 Mac，使用命令`rvictl -s UDID`来映射一个新的`rvi0`网卡，Wireshark 可以自动检测到。其中`UDID`是 iOS 设备的 UDID，通过 iTunes 可以获取。

之后就可以采用 Wireshark 的命令过滤一些东西来分析网络包了，比如：

```
(ip.src == 120.77.13.194 || ip.dst == 120.77.13.194) && http
```

过滤所有与服务器 120.77.13.194 通讯的 http 包，包括发送和接收的。

# 显示或隐藏 Finder 的所有文件

```bash
# 显示文件
alias showFiles='defaults write com.apple.finder AppleShowAllFiles YES; killall Finder /System/Library/CoreServices/Finder.app'
# 隐藏文件
alias hideFiles='defaults write com.apple.finder AppleShowAllFiles NO; killall Finder /System/Library/CoreServices/Finder.app'
```

把上面两行加入 `~/.zshrc`，其中 `killall Finder /System/Library/CoreServices/Finder.app` 会自动重启 Finder。

以后用的时候只要输入`showFiles`或`hideFiles`即可。

# 找回消失的「允许任何来源应用运行」选项

终端中输入：`sudo spctl --master-disable`

# Xcode 9 账号经常出现 session 失效

这是 Xcode 的 bug，使用下面的命令解决，禁止使用 Keychain

`defaults write com.apple.dt.Xcode DVTDeveloperAccountUseKeychainService -bool NO`

# 终端命令失效

修改`PATH`目录时，不小心增加了一个空格，如：

```
export PATH= /Application/flutter/bin:$PATH
```

然后出现了`-bash: ls :command not found`错误，所有命令都用不了，重启终端也没用，这时可以用下面的方法解决：

1. 使用`export`命令临时修改`PATH`

```
export PATH="/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin"
```

2. 使用`open -e ~/.zshrc`打开编辑器，解决问题。

# `open` 打开命令常用用法

- `open -a Sublime\ Text ~/.zshrc` 使用`Sublime Text`打开文件`~/.zshrc`
- `open -e ~/.zshrc` 使用 TextEdit 文本编辑打开文件
- `open .` 用`Finder`打开目录

# 参考链接

- [截图工具？省着点，系统帮你备着呢](http://www.tip4mac.com/2011/01/screencapture-for-os-x/)
- [The 50 best Mac tips, tricks and timesavers](http://www.techradar.com/how-to/software/operating-systems/50-best-mac-tips-tricks-and-timesavers-1310283)
- [10 essential Mac tips and tricks](https://blogs.which.co.uk/technology/apple/10-essential-mac-tips-and-tricks/)
