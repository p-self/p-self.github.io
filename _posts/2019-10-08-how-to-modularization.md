---
layout: post
title:  "如何组织组件的模块化开发"
date:   2019-10-08 14:34:12 +0800
categories: dev vue
---
## 绪论

前端模块化开发是前端工程化的一部分，是为实现应用**功能相互独立**的一组方法。

通过前端模块化可以实现几个明显的好处：

1. 简化，模块之间相互独立与可追溯的关联关系给代码的阅读和管理提供了显而易见的方便，提高了代码的可读性和易维护性。
2. 隔离，模块具有明确的功能，不同模块之间完全符合封装的原则，单个模块只对外暴露有限的接口与方法。
3. 复用，模块的最大优势在于提高代码的复用程度，模块之间的引用关系就是代码复用的方式。

模块化主要处理了前端资源，分别为代表页面行为的JavaScript，代表页面页面结构的html，代表页面样式的css。

不同的资源类型有着各不相同的模块化拼装方案，应该因地制宜的解决对应资源的组织及使用问题。下面就在逐一的了解一下当今面对不同的资源类型所使用的模块化解决方案。

## 资源模块化

模块化规范是规范代码组织方式和模块接口的一组标准，通过这些标准的遵循可以提高组件间交互操作的便捷性和便利性。

### JavaScript

当前共存这多种不同的模块化规范，每种模块化规范都有着众多的支持者和使用者，市面上大多数的js框架/库都同时支持多种模块化规范的使用方式。

#### CMD,Common Module Definition

CMD是nodejs代码的通用写法，通过`module.exports`来暴露模块API，通过require来引入并使用模块。

```javascript
//模块定义
var Light = require("light");
var logger = new Light.Logger({level:Light.Logger.LEVEL_INFO});

var M = {
    logInfo : function(msg){
        return logger.info(msg);
    }
};

module.exports = M;
```

```javascript
//模块使用
var Logger = require("logger");

Logger.logInfo("hello,light");
```

#### AMD,Asynchronous Module Definition

CMD来源于RequireJS，通过define来定义模块，通过require来引入并使用模块。

```javascript
//模块定义
define(["light"], function( Light ){
    var logger = new Light.Logger({level:Light.Logger.LEVEL_INFO});
    return {
        logInfo : function(msg){
            return logger.info(msg);
        }
    }
});
```

```javascript
//模块使用
require("logger",function(Logger) {
    Logger.info("hello，light");
})
```

#### UMD,Universal Module Definition

UMD出现的意义是兼容AMD和CMD的写法，成为一种统一的规范的前端模块化范式。

```javascript
//模块定义与使用
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['light'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS之类的
        module.exports = factory(require('light'));
    } else {
        // 浏览器全局变量(root 即 window)
        root.returnExports = factory(Light);
    }
}(this, function (Light) {
    var logger = new Light.Logger({level:Light.Logger.LEVEL_INFO});
 
    return function(msg){
       return logger.info(msg);
    };
}));
```

### CSS

和JavaScript不同的是，浏览器原生的支持了小部分能力的样式模块化，那就是`@import`，通过`@import`可以实现简单的模块化处理。但是，通过原生的`@import`来引入资源会带来一些性能损耗，如：

1. 变相增加解析成本
2. 导致页面的http连接数增加
3. 不同浏览器的不同写法有可能带来不同的加载顺序

所以，在性能要求严格的环境下原生支持的`@import`并不适用。

由此，出现了大量的的各式各样的css预处理器，这些预处理器为css的模块化书写和使用提供了便利，比如less/sass/stylus。

#### less

在标准的css写法当中，`@import`必须存在于所有代码之前，但在less中`@import`可以存在于任何位置。

```less
@import "base.less";
@import "header.less";

body{
background-color: #fff;
}

@import "others.less";
```

#### sass

在sass中`@import`，后缀名`.sass`都可以省略。

基础的文件命名方法以`_`开头，如`_mixin.scss`，在使用`@import`的时候可以不携带`_`。

```sass
@import "mixin";
@import "base";
@import "header";
```

#### stylus

在stylus中提供了一种脚本化书写css样式的方案，在stylus中，任何不带`.css`的`@import`引入都会被认为是stylus片段。

同时stylus的`@import`还支持与node类似的文件查找方案。例如当`@import light`，会先寻找`light.styl`，然后寻找`light/index.styl`.

```stylus
@import 'base'
body
  background: #fff;
```

### UI组件化

UI组件通常包含行为/结构/样式，相对于单纯资源和复杂业务而言，是最合适的复用单元。比如在前端开发经常碰到的轮播图/弹出窗体等都适合作为UI组件来进行封装和复用。

## 通过工具实现代码模块化

前端模块化不能简单的理解为JavaScript资源的模块化，或者CSS资源的模块化，而应该是包含了前端所有相关资源的模块化拆分，其中包括但不限于表示页面结构的html，表示页面样式的css，和表示页面行为的JavaScript。

如果纯手工的使用不同的模块化规范是非常耗时和不易维护的，一方面模块化规范所设计的代码大部分是程式化的，比较统一；另一方面模块化规范所引入的代码不应该进入源代码进行管理。

所以，就出现了一些可以达成自动实现前端模块化打包（module bundler）的工具，其中比较通用常用的是webpack和browserify。这些前端模块化的工具库已经发展成为一站式的前端构建工具，通过引入不同的插件对不同类型的前端资源进行预处理和后处理。

### webpack

webpack起源于React项目，随着react的广泛使用而逐渐流传起来。webpack天生支持CMD和AMD模块，可以无缝的使用使用现存的JS模块做功能开发。

webpack视所有前端资源都为模块，包括css和图片等资源文件，这就更符合前端工程化的思想。

![webpack](/images/what-is-webpack.png)
from http://webpack.github.io

```javascript
//配置文件
var path = require('path');

module.exports = {
  entry: './app/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

```javascript
//入口文件
import _ from 'lodash';

_.each(['light','is','awesome'],function(word) {
  console.log(word);
})
```

### browserify

browserify和webpack的功能颇为相似，但browserify为了让前端也可以使用npm中数以万计的CMD的模块，提供了针对浏览器打包的nodejs中的大部分原生模块的特殊版本。所以browserify默认兼容CMD的写法使用exports/require来组织模块之间的引用和依赖。

同时，browserify也可以通过引入插件对所有干涉的前端资源分别进行处理，将模块化切割之后的前端资源整合打包，输出完整的可运行内容。

```javascript
//入口文件
var styles = require('./styles.css');
var div = `<div class="${styles.inner}">...</div>`;
```

```javascript
//执行打包
var b = require('browserify')();
 
b.add('./main.js');
b.plugin(require('css-modulesify'), {
  rootDir: __dirname,
  output: './dist/styles.css'
});
 
b.bundle();
```

## Lighting下的模块化实践

lighting下的模块化方案有更多的考虑：

1. 提高开发协作的便利性
2. 模块层次化，面向单一业务
3. 分门别类管控前端资源

在lighting的开发项目中，既可以使用UMD中的JS模块化的组织方式，又可以使用vue单文件组件中对前端资源统一模块化封装的方案，还可以使用lighting对业务间隔离和复用的模块化方案。通过多层次相互关联的模块化复用方案将应用的逻辑/业务解耦，更大程度上满足应用开发的需要。

### lighting项目下使用import/export

`import/export`是es6提供的模块化方案，是一种同步的模块导出/引入的使用方式，通过启用`lighting-plugin-es6`插件可以在所有的`es6`文件当中使用es6的语法。

### lighting项目下使用vue进行UI组件化

vue框架下的单文件组件可以有效的组织UI组件的复用，一份vue文件就是一个合法的UI组件，可以通过html标签的方式引入进行使用。

lighting框架下可以使用`lighting-plugin-vue`插件来启用工程对于vue单文件组件的支持。

```html
<!--src/vue/loading.vue-->
<template>
    <div class="loading" v-show="showLoading">
        <div class="loading-text">数据加载中请稍等...</div>
        <!--<img src="/images/loading.svg" alt="">-->
    </div>
</template>
<script>
    export default{
        props:['showLoading']
    }
</script>
<style>
	.loading{
      background-color:#fff;
	}
</style>
```
定义完毕的vue单文件组件可以直接使用`<目录名-标签名>`的html标签进行使用。

### lighting项目下使用view进行业务模块化

![view](/images/light-view.png)

相对于其他的模块化方案，在lighting中引入了一种业务模块化的使用方式，在lighting框架的定义当中page代表了一份完整的业务流程，每一个view都是一个可以复用的业务环节，通过对业务环节（view）的串联/组合实现业务流程的开发是lighting框架基本的方法论和理念。

view是lighting框架中的最小业务复用单元，其包含可一份与之关联的js代码（控制行为）和一份html代码（控制结构），和UI组件不同的是，view的复用粒度更大，场景更加局限，只适合在同一应用内，或者相似业务间进行功能复用。

通过view的引入可以在更高层次进行代码的组织和使用。

## 总结

前端模块化方案存在着不同的层次，从资源的模块化组织，到UI的组件化复用，再到业务逻辑的分离分割，都是模块化解决方案在面对不同层次问题的处理方法。综合运用，才能让前端工程的开发更便捷/更高效/更弹性。