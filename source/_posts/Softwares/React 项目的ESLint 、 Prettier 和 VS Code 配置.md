---
title: React 项目的ESLint 、 Prettier 和 VS Code 配置
date: 2018-09-25 23:51:53
category: Softwares
---

首先确保项目本地已经安装了`eslint`，而且 VC Code 装了 `eslint`和`Prettier` 的插件。

这里 js 的规范使用 Airbnb 的。

<!-- more -->

配置步骤如下：

- 安装 `eslint-config-airbnb` 包，以及相关联的包

  ```
  yarn add eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react -D
  ```

- 安装`prettier`相关的包
  ```
  yarn add prettier eslint-config-prettier eslint-plugin-prettier -D
  ```
- 创建 eslint 的配置文件`.eslintrc.json`，复制下面的配置

```
{
  "extends": [
    "airbnb",
    "prettier",
    "prettier/react"
  ],
  "rules": {
    "react/jsx-filename-extension": [
      1,
      {
        "extensions": [
          ".js",
          ".jsx"
        ]
      }
    ],
    "prettier/prettier": [
      "error",
      {
        "trailingComma": "es5",
        "singleQuote": true,
        "printWidth": 100
      }
    ]
  },
  "plugins": [
    "prettier"
  ]
}
```

- VS Code 的设置里面增加下面的配置：

```
{
  // other settings
  // formatting using eslint
  // let editor format using prettier for all other files
  "editor.formatOnSave": true,
  // disable editor formatting, so eslint can handle it
  "[javascript]": {
    "editor.formatOnSave": false,
  },
  // available through eslint plugin in vscode
  "eslint.autoFixOnSave": true,
  "eslint.alwaysShowStatus": true,
}
```
