---
layout: post
title:  "5分钟搞定JSNative应用开发"
date:   2017-02-08 14:34:12 +0800
categories: dev jsnative
---
## H5应用开发的三种流派

app开发有几种不同的流派，分别为：

1. **原生开发**，ios使用ObjectC，Android使用Java来开发应用程序
2. **混搭开发**，使用webview来承载内容，页面使用浏览器引擎来渲染；比较有代表性的是PhoneGap/cordova
3. **JSNative**，使用JavaScript作为开发方式，但是程序是原生应用；比较有代表性的是ReactNative

| 方式       | 优势             | 劣势                         |
| -------- | -------------- | -------------------------- |
| 原生开发     | 性能好，运行效率高      | 开发/维护成本高                   |
| 混搭开发     | 开发/维护成本低，跨平台运行 | 性能查，运行速度慢，依赖于webview的运行时版本 |
| JSNative | 性能适中，开发成本适中    | 入门学习成本                     |

今天的主角是第三种方案，因为其既跨平台的兼容了主要的运行环境，让一份代码运行在不同的设备和容器中；同时还具有媲美native程序的性能。

## 初探JSNative

### ReactNative与Weex

在了解weex之前先说说ReactNative,ReactNative和react一样都是facebook开源的框架，之前简单的了解过RN，也用RN写过简单的Helloworld级别的程序，总体上的感觉是：

1. 语法有点奇怪，使用jsx写dom的方式不是习惯做JavaScript开发的前端开发者所能理解得了的
2. 和前端的开发方式相去甚远，关于样式/DOM的概念需要重新的学习和理解

和RN相比，weex是一个相同问题的第二种解决方案，其中使用了前端都很熟悉的vue框架，使用vue单文件组件作为程序的开发实现方式，学习成本更小，效率更高！但是，本身要开发使用weex技术需要做的功课比较多，步骤也比较复杂，简化来说至少需要一下的几个过程：

### 搭建开发环境

首先要做的就是安装weex的命令行工具，这个命令行工具可以帮助我们对付前端开发的各种事情：

```bash
yarn global add weex-toolkit --ignore-engines
```

接下来，需要Android的开发环境，包括AndroidSDK的下载安装以及可以开发Android应用的IDE环境，可用idea来开发。

### 集成weex

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

### 开发流程

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

### 小结

综合使用下来，觉得weex的初衷是好的，用应用开发都已经熟悉的一套东西做更多的事情，提高程序的开发效率，但是也暴露了这个框架一些不成熟的地方：

1. 官方的文档更新可能并不及时，demo中使用的api或者文档中介绍的api在实际代码上可能都已经移除了
2. RN提供了丰富好用的工具，包括手机端和开发端，尤其是摇一摇自动刷新的功能在开发过程中可以省却不少的问题，但weex本身提供的工具就相对原始一些
3. weex的使用门槛还是有点高，需要具备原生应用的开发知识，否则连开发环境都没法搞定

正是由于weex本身使用的极大的复杂性，`light`对外提供了一套更加简洁高效的**框架**和**工具**来支持`JSNative`应用尤其是`weex`应用的开发！


## lighting辅助JSNative应用

`lighting`是`light`平台对外发布**开发构建**工具，[点我查看详情](https://document.lightyy.com)，以下默认开发者已经安装有最新版本的`lighting`及其插件！

### 在light工程中集成weex

在正式开发之前需要分别使用以下两个指令检查当前操作系统安装的版本是否支持`JSNative`项目的开发。如果指令执行后输出的版本号大于以下所示的版本号，那么就可以继续下面的步骤了，否则可以使用`npm install -dg lighting`来升级**lighting**版本，使用`light plugin -a type-vue`来升级**type-vue**插件版本，使用`light plugin -a native`来升级**native**插件版本。

```bash
 ~> light -v


     _  _         _      _    _               
    | |(_)       | |    | |  (_)              
    | | _   __ _ | |__  | |_  _  _ __    __ _ 
    | || | / _` || '_ \ | __|| || '_ \  / _` |
    | || || (_| || | | || |_ | || | | || (_| |
    |_||_| \__, ||_| |_| \__||_||_| |_| \__, |
            __/ |                        __/ |
           |___/                        |___/      CLI 1.4.7-201701031


 ~> light plugin --list
lighting-plugin-type-vue:1.0.30
lighting-plugin-native:1.0.14
```

接下来，就可以开发`JSNative`代码了，可以从我们开源在github上的一份demo起步，体会一下无差别开发app的流畅享受，一份完全相同的代码全面覆盖h5/iOS/Android三端平台！demo的链接下载地址是：[https://github.com/HS-Light/light-demo-native](https://github.com/HS-Light/light-demo-native)

### 编译JSNative工程

对于`JSNative`工程的编译和普通的`h5`应用的编译的方式一致，同样是用`release`命令，其中`-w`代表开启源代码监听，`-b 3001`达标打开浏览器并将sever监听在3001端口，`--weex`代表当前的工程采用JSNative编译的方式在编译`h5`页面的同时并且输出`app.native.js`。

```bash
 ~/D/l/l/JSNative> light release -wb 3001 --weex
[2017-11-06 09:29:46][INFO] 清除/tmp/native目录
[2017-11-06 09:29:46][INFO] 拷贝/home/abel/Documents/light/lighting-demo/JSNative到/tmp/native目录
[2017-11-06 09:29:47][INFO] native插件加载成功
[2017-11-06 09:29:47][INFO] 阶段：prepare，调用插件type-vue开始
[2017-11-06 09:29:47][INFO] 阶段：prepare，调用插件native开始
[2017-11-06 09:29:47][INFO] 阶段：prepare，调用插件native成功
[2017-11-06 09:29:47][INFO] 阶段：prepare，调用插件type-vue成功
[2017-11-06 09:29:47][INFO] 处理资源装配：index
[2017-11-06 09:29:47][INFO] 处理脚本资源注入：index
[2017-11-06 09:29:47][INFO] 阶段：build，调用插件type-vue开始


  编译中 [====================] 100% (14.8 秒)

Build completed in 2.28s

[2017-11-06 09:30:03][INFO] 阶段：build，调用插件type-vue成功
[2017-11-06 09:30:04][INFO] 已开启本地资源监听...
[2017-11-06 09:30:04][INFO] 资源编译成功
```

github上的demo工程打开预览后是这样的效果：

![](/images/DeepinScreenshot_select-area_20171106093758.png)

用浏览器打开`http://localhost:3001/app.native.js`，可以看到JSNative编译后的内容，此内容可以支持使用weex的playground或者light的`lightview`来预览查看！在lightview中的预览效果是这样的：

![](/images/Screenshot_2017-11-06-09-48-02-137_com.hundsun.light.lightview.appstore.png)

## 总结

`lighting`开发套件为JSNative应用的开发提供了方便的支持，简化了`JSNative`应用开发的方法和流程，让开发者可以以最小的成本完成`三端统一`的应用程序的开发，大大提高了应用的开发效率！

