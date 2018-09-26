---
title: Ruby-Gem 使用
date: 2018-09-25 23:51:53
category: Softwares
---

# 使用[rvm](https://github.com/rvm/rvm)来安装指定版本的 ruby

首先确保已经安装下面两个库:

- `curl`
- `gpg2`

<!-- more -->

然后使用运行命令安装：

```bash
\curl -sSL https://get.rvm.io | bash -s stable
```

如果想要升级`rvm`到新的稳定版本，可以使用命令：

```bash
rvm get stable
```

安装指定版本的 ruby：

```bash
rvm install 2.3.1
```

切换 ruby 版本：

```bash
rvm use 2.4.1
```

# 安装或更新 gem 时忽略文档

在`~/.gemrc`配置文件里面加入下面一句话：

```bash
gem: --no-rdoc --no-ri
```

# 迁移一个 ruby 版本的 gems 到另一个版本

```bash
gem migrate 2.1.1 2.4.1
```
