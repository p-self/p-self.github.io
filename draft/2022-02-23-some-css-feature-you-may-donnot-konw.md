---
layout: post
title:  "你可能并不知道的CSS特性"
date:   2022-02-23 11:34:12 +0800
categories: dev css
---

css变量

CSS变量是以`--`开头的CSS属性，可以以`var()`的形式应用在当前元素或者子元素中，CSS变量中的属性名称忽略大小写；

```css
:root {
  --first-color: #16f;
  --second-color: #ff7;
}
```〔筆畫〕

`-webkit-line-clamp` 可以声明内容元素所占用的行数，智能应用在`-webkit-box` or `-webkit-inline-box`的元素上，并且`-webkit-box-orient ` 必须设置为`vertical`；

`accent-color`可以设置checkbox、radio、range、progress能浏览器内置组件的外观颜色；

