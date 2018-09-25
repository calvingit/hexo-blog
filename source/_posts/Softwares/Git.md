
---
title: Git
date: 2018-09-25 23:51:53
category: Softwares
---

    
    
如果是GUI工具，Mac 上面一般是用[SourceTree](https://www.sourcetreeapp.com)或者[Tower](https://www.git-tower.com/mac/)。

这里主要介绍命令行的使用方法。

<!-- more -->


# 全局设置

## 用户信息,包含用户名和email

```bash
git config --global user.name calvin
git config --global user.email calvin@example.com
```

## 编辑器

```bash
git config --global core.editor /usr/bin/vim
```

这个最好设置一下，因为下面的`rebase`等命令会需要用来编辑文本

## gitignore

```bash
git config --global core.excludesfile /Users/zhangwen/.gitignore_global
```
github开源了一个收集各个语言、系统环境相关的[gitignore](https://github.com/github/gitignore.git)

## 设置代理
```bash
git config --global http.proxy socks5://127.0.0.1:1086
git config --global https.proxy socks5://127.0.0.1:1086
git config --global http.sslverify false
```


# 合并多个提交

使用命令`rebase`。假设有三段提交历史，如下:

> commit 7efebeb
>
> commit c2e9a92
>
> commit 4f4c6f8

现在要把`7efebeb`和`c2e9a92`合并为一个，使用命令:

```bash
git rebase -i 4f4c6f8
```

`-i`的参数表示不需要合并的提交，然后会自动进入vim的编辑界面:

```bash
 pick c2e9a92 设置数据的基类
 pick 7efebeb 修改model

 # Rebase 4f4c6f8..7efebeb onto 4f4c6f8 (2 command(s))
```

有几个命令需要注意：

- pick 使用这个提交，缩写p
- squash 使用这个提交合并到前一个提交，缩写s
- drop 丢弃提交，缩写d

现在把第二行的`pick`改成`s`，然后保存退出`:wq`。然后会自动进入编辑新commit记录的界面，非`#`开头的内容都当做历史记录，然后保存退出`:wq`，自动提交成功了。

取消合并使用命令：

```
git rebase --abort
```

# 获取最新的tag
tag一般是版本号，经常需要使用最新的版本号当做文件名的一部分，命令如下：

```bash
git describe --tags --abbrev=0
```

# 删除github的tag
网站上没有提供操作按钮，只能使用命令行操作在本地删除之后，再推送到服务器：

```shell
git tag -d [tag]
git push origin :[tag]
```
如果不行的话，第二条命令改成下面这种：

```shell
git push origin :refs/tags/[tag]
```

# 找回提交到分支`HEAD`里的代码

`HEAD`是指当前分支，每次`checkout`改变分支，它就会指向哪里。有时分支没有名字，此时就变成`detached(游离)`状态的。

经常在submodule里面，当你直接在submodule修改代码并提交后，再切换回`master`，刚刚提交的代码不见了，而且`HEAD`分支也不见了。

但刚刚提交的代码还在git里面，并没有丢失，此时可以用`git reflog`查看提交记录，找到刚刚的提交记录，假设为`247e11b`，执行下面的命令即可：

```bash
# 切换到 247e11b，此时是 detached
git checkout 247e11b
# 把 detached 分支保存下来，命名为 diff 分支
git checkout -b diff
# 切换到 master 分支
git checkout master
# 合并 diff 到 master
git merge diff
```