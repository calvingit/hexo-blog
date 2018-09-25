---
title: borderRadius 圆角设置
date: 2018-09-25 23:51:53
category: React-Native
---

    除了设置`borderRadius`参数，还需要设置`overflow`为`hidden`

在 CSS 里面, `overflow` 属性指定当内容溢出其块级容器时,是否剪辑内容，显示滚动条或显示溢出的内容。

```
/* 默认值。内容不会被修剪，会呈现在元素框之外 */
overflow: visible;

/* 内容会被修剪，并且其余内容不可见 */
overflow: hidden;

/* 内容会被修剪，浏览器会显示滚动条以便查看其余内容 */
overflow: scroll;

/* 由浏览器定夺，如果内容被修剪，就会显示滚动条 */
overflow: auto;

/* 规定从父元素继承overflow属性的值 */
overflow: inherit;
```
