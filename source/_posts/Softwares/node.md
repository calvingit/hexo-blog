---
title: node
date: 2018-09-25 23:51:53
category: Softwares
---

# 环境搭建

## 安装

- 安装 [nvm](https://github.com/creationix/nvm)
- 安装 node，使用 **lts** 稳定版本: `nvm install --lts`。
  > 如果已经用 nvm 安装过 node 了，现在要升级 node 并把已经安装过的全局 packages 一并迁移到新版本，那么需要使用一下命令：`nvm install v8.4.0 --reinstall-packages-from=v6.10.0`

<!-- more -->

## 配置淘宝镜像

把 registry 和 disturl 都改成淘宝的镜像，下载速度上更快。

官方的：

```bash
npm config set registry https://registry.npmjs.org --global
npm config set disturl https://npmjs.org/dist --global
```

淘宝的

```bash
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```

## 更新全局包

`npm update -g`

# 模块

node 模块使用 CommonJS 规范。

> CommonJS 规范
>
> 这种模块加载机制被称为 CommonJS 规范。在这个规范下，每个.js 文件都是一个模块，它们内部各自使用的变量名和函数名都互不冲突，例如，hello.js 和 main.js 都申明了全局变量 var s = 'xxx'，但互不影响。
>
> 一个模块想要对外暴露变量（函数也是变量），可以用 module.exports = variable;，一个模块要引用其他模块暴露的变量，用 var ref = require('module_name');就拿到了引用模块的变量。

**导出模块**

```javascript
'use strict';

var s = 'Hello';

function greet(name) {
  console.log(s + ', ' + name + '!');
}

// 导出greet函数
module.exports = greet;
/*
// 导出对象
module.exports = {
    foo: function () { 
        return 'foo'; 
    }
};
*/
```

**引入模块**

```javascript
// 可以省略.js后缀
var greet = require('./hello');
```
