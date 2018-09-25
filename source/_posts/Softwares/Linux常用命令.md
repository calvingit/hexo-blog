---
title: Linux常用命令
date: 2018-09-25 23:51:53
category: Softwares
---

# tar

主要用来打包的，附带压缩功能。

`tar (选项) (参数)`

### 选项：

```
-A或--catenate：新增文件到以存在的备份文件；
-B：设置区块大小；
-c或--create：建立新的备份文件；
-C <目录>：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。
-d：记录文件的差别；
-x或--extract或--get：从备份文件中还原文件；
-t或--list：列出备份文件的内容；
-z或--gzip或--ungzip：通过gzip指令处理备份文件；
-Z或--compress或--uncompress：通过compress指令处理备份文件；
-f<备份文件>或--file=<备份文件>：指定备份文件；
-v或--verbose：显示指令执行过程；
-r：添加文件到已经压缩的文件；
-u：添加改变了和现有的文件到已经存在的压缩文件；
-j：支持bzip2解压文件；
-v：显示操作过程；
-l：文件系统边界设置；
-k：保留原有文件不覆盖；
-m：保留文件不被覆盖；
-w：确认压缩文件的正确性；
-p或--same-permissions：用原来的文件权限还原文件；
-P或--absolute-names：文件名使用绝对名称，不移除文件名称前的“/”号；
-N <日期格式> 或 --newer=<日期时间>：只将较指定日期更新的文件保存到备份文件里；
--exclude=<范本样式>：排除符合范本样式的文件。
```

### 参数：

文件或目录：指定要打包的文件或目录列表。

### 例子

压缩：

```bash
tar -cvf log.tar log2012.log    仅打包，不压缩！
tar -zcvf log.tar.gz log2012.log   打包后，以 gzip 压缩
tar -jcvf log.tar.bz2 log2012.log  打包后，以 bzip2 压缩
```

在选项`f`之后的文件档名是自己取的，我们习惯上都用 `.tar` 来作为辨识。 如果加 z 选项，则以`.tar.gz`或`.tgz`来代表 gzip 压缩过的 tar 包；如果加`j`选项，则以`.tar.bz2`来作为 tar 包名。

解压：

```bash
tar -zxvf /opt/soft/test/log.tar.gz
tar -jxvf filename.tar.bz2
```

# 进程管理

**ps**

- 查看所有进程: `ps -ef`
- 查找进程的 id：`ps -ef | grep {关键字} | grep -v grep | awk '{print $2}'`
- 杀死进程：`kill -9 {进程id}`

**top**

**htop**

# sed

https://robots.thoughtbot.com/sed-102-replace-in-place
