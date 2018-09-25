---
title: yarn
date: 2018-09-25 23:51:53
category: Softwares
---

# 安装

[yarn](https://yarnpkg.com/zh-Hans/) 可以用 Homebrew 安装，分两种情况：

- 如果没有安装 node，或者 node 是用 Homebrew 安装的:  
  `brew install yarn`
- 如果 node 是用 nvm 等第三方工具管理的，应该排除安装 node：
  `brew install yarn --without-node`

# 配置 registry

官方的：

```bash
yarn config set registry https://registry.npmjs.org --global
yarn config set disturl https://npmjs.org/dist --global
```

淘宝的：

```bash
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```

# 使用

- 初始化项目：`yarn init XXX`
