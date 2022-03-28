---
layout: post
title:  "扩展web中的元素世界：web components"
date:   2022-02-10 11:34:12 +0800
categories: dev html
---

## 简介

web components 的存在是为了解决组件化的复用代码的问题，对于相同的逻辑、结构、样式的重复书写会污染页面的代码，造成问题难以排查；web components有效的隔离了组件和业务逻辑的部分，将干扰降低；web components基于三项关键的技术构建而成：

1. Custom elements: 自定义元素，
2. Shadow DOM： 减少污染和相互影响
3. HTML templates： `<template>` and `<slot>` 的存在支持书写一些不被浏览器直接渲染的代码，此种代码可以在运行时被重复使用

## Custom elements

