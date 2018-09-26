---
title: Shell
date: 2018-09-25 23:51:53
category: Softwares
---

在我的 macOS 日常使用中，真正经常使用的 GUI 软件不外乎 Xcode、Chrome、QQ、SourceTree 等，其他的操作大部分还是基于命令行来的，命令行的软件当然是标配的 [iTerm2](https://www.iterm2.com/downloads.html) + [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh.git) 套装了。很多零碎的操作，如果要特别找一个 GUI 软件的话也比较难，毕竟不比 Windows 上庞大的软件数量，有时用几个简单的命令会让你的工作效率增加数倍。

<!-- more -->

# 查看占用某端口的进程

命令：

```bash
lsof -n -P | grep :8123
```

输出：

```bash
polipo    17200   zw    3u     IPv4 0x14984bc28dba98bd        0t0      TCP 127.0.0.1:8123 (LISTEN)
```

解释：

- `lsof`：列出打开的文件，Unix 的文件含义范围很大，端口也是文件的一种，如果不加参数会列出很多东西，有点卡住终端。
- `-n -P` 过滤网络端口，加快查询速度。
- `grep :8123` 然后再搜索带有":8123"的那行，8123 就是端口。

# curl 命令

- 查看当前的外网 IP：`curl ip.gs`
