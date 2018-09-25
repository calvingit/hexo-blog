---
title: Android x86 虚拟机Genymotion 配置
date: 2018-09-25 23:51:53
category: Softwares
---

# 安装

首先到[官网]()下载最新版本

1. 首先点击菜单栏的**Add**，然后选择**Android version**和**Device model**，选择你想要的设备，记下型号，稍后需要用到。
2. 一直**Next**到下载界面，然后取消下载
3. 到目录`~/.Genymobile/Genymotion/ova`，找到`.partial`文件，这个是刚刚未下载完成的缓存文件
4. 用迅雷下载文件更快

## 虚拟机镜像 ova 文件下载

Genymotion ova 下载的 url 地址为：**http://dl.genymotion.com/dists/xxx/ova/xxxxxx**,可复制下载地址到迅雷中下载，速度会快很多,其中 xxx 为虚拟设备对应的 Android 系统版本号,如 4.2.2,7.0.0(4.3 则 xxx 为 4.3,5.0 则 xxx 为 5.0.0)，xxxxxx 为 ova 的文件名，如:

```
http://dl.genymotion.com/dists/4.2.2/ova/genymotion_vbox86p_4.2.2_170320_181617.ova

http://dl.genymotion.com/dists/4.3/ova/genymotion_vbox86p_4.3_170321_020053.ova

http://dl.genymotion.com/dists/7.0.0/ova/genymotion_vbox86p_7.0_170321_002642.ova
```

> 当把完整的链接复制到迅雷的输入框内时，无法立即看到预览的下载包大小，即表明链接有错，需要修改 dists 后面的版本号，可能就是少了`.0`，比如`5.1`版本的`genymotion_vbox86p_5.1_170928_222241.ova`，下载链接为:
> http://dl.genymotion.com/dists/5.1.0/ova/genymotion_vbox86p_5.1_170928_222241.ova
>
> 这样才有效，能预览到下载大小。

把下载好的文件放到目录`~/.Genymobile/Genymotion/ova`

然后重复上面的第 1 步，记住刚刚的型号，选择正确即可。

# 安装 ARM 兼容工具

[ARM_Translation_Marshmallow.zip](blob:https://mega.nz/7bdef2ec-96e3-4e02-aabf-e401a09236ad)，支持 Android 6.0 以上

# 出现的问题

1. `could not install *smartsocket* listener: Address already in use ADB server didn't ACK`
   配置 Android SDK 的`platform-tools`和`tools`放在 path
