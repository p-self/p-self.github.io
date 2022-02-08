---
layout: post
title:  "使用weex开发native应用程序体验"
date:   2017-09-28 14:34:12 +0800
categories: dev module
---

app开发有几种不同的流派，分别为：

1. 原生开发，ios使用ObjectC，Android使用Java来开发应用程序
2. 混搭开发，使用webview来承载内容，页面使用浏览器引擎来渲染；比较有代表性的是PhoneGap
3. jsNative，使用JavaScript作为开发方式，但是程序是原生应用；比较有代表性的是ReactNative

今天的主角是第三种方案，因为其既跨平台的兼容了主要的运行环境，让一份代码运行在不同的设备和容器中；同时还具有媲美native程序的性能，在了解weex之前先说说reactNative。

## ReactNative与Weex

reactNative和react一样都是facebook开源的框架（使用Facebook私有协议，不过昨天还想听说有计划变更到MIT，这个协议的问题还引发了一场挺大的波澜），之前简单的了解过RN，也用RN写过简单的Helloworld级别的程序，总体上的感觉是：

1. 语法有点奇怪，jsx的东西不是习惯做JavaScript开发的前端开发者所能理解得了的
2. 和前端的开发方式相去甚远，关于样式/DOM的概念需要重新的学习和理解

和RN相比，weex是一个相同问题的第二种解决方案，其中使用了前端都很熟悉的vue框架，使用vue单文件组件作为程序的开发实现方式，学习成本更小，效率更高！

## 搭建开发环境

首先要做的就是安装weex的命令行工具，这个命令行工具可以帮助我们对付前端开发的各种事情：

```bash
yarn global add weex-toolkit --ignore-engines
```

接下来，需要Android的开发环境，包括AndroidSDK的下载安装以及可以开发Android应用的IDE环境，可用idea来开发。

## 集成weex

首先在dependencies下增加如下的依赖包，

```gradle
dependencies {
    compile 'com.android.support:recyclerview-v7:25.3.1'
    compile 'com.android.support:support-v4:25.3.1'
    compile 'com.alibaba:fastjson:1.1.46.android'
    compile 'com.taobao.android:weex_sdk:0.5.1@aar'
}
```

然后开发activity可以了，

```java
WXSDKInstance mWXSDKInstance = new WXSDKInstance(this);
mWXSDKInstance.registerRenderListener(this);
mWXSDKInstance.render("WXSample", WXFileUtils.loadFileContent("hello.js", this), null, null, -1, -1, WXRenderStrategy.APPEND_ASYNC);
```

```vue
<template>
  <div>
    <text style="font-size:100px;">Hello World.</text>
  </div>
</template>
```

## 开发流程

开发weex页面需要使用到weex的一个工具[Weex Playground](https://weex.apache.org/cn/playground.html)

![](/images/DeepinScreenshot_select-area_20170928143205.png)

使用我们上面安装的weex-toolkit工具，可以创建一个应用工程出来，

```bash
$ weex init weex-frontend
$ cd weex-frontend
$ yarn install --ignore-engines
$ npm run start
```

经过上面的几个步骤，一个可以被Playground预览扫描的工具就完成了！

## 写在最后

综合使用下来，觉得weex的初衷是好的，用应用开发都已经熟悉的一套东西做更多的事情，提高程序的开发效率，但是也暴露了这个框架一些不成熟的地方：

1. 官方的文档更新可能并不及时，demo中使用的api或者文档中介绍的api在实际代码上可能都已经移除了
2. RN提供了丰富好用的工具，包括手机端和开发端，尤其是摇一摇自动刷新的功能在开发过程中可以省却不少的问题
3. weex的使用门槛还是有点高，需要具备原生应用的开发知识，否则连开发环境都没法搞定
