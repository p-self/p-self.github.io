---
layout: post
title:  "vue在v-for遍历的数组中存在重复值情况下执行出错的问题处理"
date:   2017-05-20 14:34:12 +0800
categories: dev vue
---

此问题由某部门同事开发中所发现，并提交给了lighting的开发团队跟进处理。

## 问题描述

从一个console的截图开始：

![](./Screenshot_2017-05-12_08-43-29.png)

lighting的开发框架是以`vue`为模型绑定的核心，根据错误可以进行一个简单的判断：

1. `removeChild`操作既然不是发生在开发者显示书写的代码中，就应该是**模型销毁后vue引擎移除dom节点**导致的
2. 错误栈信息都在框架的代码之内，此操作不可能是有用户代码触发导致的

开发者某一流程的操作，会100%稳定的触发出这一错误，此错误导致js执行终端，整个程序陷入瘫痪无法工作，开发者的操作流程可以简化为如下的步骤：

1. 访问视图A
2. 访问视图B
3. 回退历史记录到A（**错误发生在这里**）

以上的跳转关系都是视图跳转，也就是发生在lighting的路由系统之内的路由跳转，按照lighting的路由逻辑，第三步的时候会依次执行视图的声明周期函数，包括：

1. B视图的unRender逻辑，包括beforeUnRender和afterUnRender
2. A视图的Render，包括beforeRender和afterRender

开发者只在beforeRender的阶段进行了模型重置的操作，几乎可以确定无疑报错就是由这几行模型重置和赋值的操作引起的。层层排除，我们得到了可以使用简单代码重新此问题的方式。

## 重现方式

准备一个简单的空工程，新建视图`test`，一下的代码分别为`js/view/test.js`和`html/view/test.html`，`js/view/test.js`中视图对模型的操作可以完整反映重现此问题的流程。

其中，`setTimeout`模拟的是`ajax`操作以让数据在多个`tick`之后设置到模型以观察vue对dom节点的创建和销毁。

`$nextTick`之后，将`test_arr`置空的操作是为了使`vue`将此数据对应的`dom`节点销毁，也就是对应与`html/view/test.html`中`<div v-for="val in test_arr">{{val}}</div>`这一段代码。

```javascript
;(function(){
  Light.defineViewModel("#test",{
       data:{
        test_arr:["","1","","2","3",""]
       }
     },
     afterRender:function (params){
      var that =this;
      setTimeout(function(){
         that.model.test_arr = ["","4","","5","6",""]
         that.model.$nextTick(function(){
          that.model.test_arr = null;
         })
      },1000*10)
     }
  });
})();
```

```html
<div id="test" style="display:none;">
	<div>
		<div v-for="val in test_arr">{{val}}</div>
	</div>
</div>
```

以上的代码可以稳定重新问题，下面是我们的解题思路。

## 解决方案

在不求甚解的状态下，这个问题是比较容易解决的，我们想到了几个临时的解决方案。

### 方案一

从报错信息`Uncaught TypeError: Cannot read property 'removeChild' of null`可知，之所以发生这个问题是因为在`null`的对象上执行了`removeChild`，那么事情就简单了。

修改vue框架代码，将这里的代码：

```JavaScript
 function remove(el) {
    el.parentNode.removeChild(el);
  }
```

修改为：

```JavaScript
 function remove(el) {
    if(el.parentNode)el.parentNode.removeChild(el);
  }
```

### 方案二

我们深入的分析了一下为什么`el.parentNode`会是`null`，通过上面重现的步骤发现，当` that.model.test_arr = ["","4","","5","6",""]`这段代码设置发生后，`v-for`产生的dom节点之后3个，而不是5个，这种情况下`el.parentNode`就是不存在的，所以产生了第二种解决方案，强制不给空数据的元素生成dom节点。

```html
<div id="test" style="display:none;">
	<div>
		<div v-for="val in test_arr" v-if="val">{{val}}</div>
	</div>
</div>
```

### 方案三

问题并不算是圆满解决，正常的情况下框架应该具有鲁棒性，适应不同的使用场景，不应该出现js报错的问题，所以还有深入研究下去的必要！

在`vue`中针对`v-for`指令有一个`track-by`的可选配置

	无track-by情况：数据修改时，无论值是否被修改，dom都被重新渲染
	有track-by属性：数据修改时，不变数据所在的dom不被重新渲染，已改变的数据所在dom才被重新渲染

因为 v-for 默认通过数据对象的特征来决定对已有作用域和 DOM 元素的复用程度，这可能导致重新渲染整个列表。但是，如果每个对象都有一个唯一 ID 的属性，便可以使用 track-by 特性给 Vue 一个提示，Vue因而能尽可能地复用已有实例。所以就有了第三种解决方案。

```html
<div id="test" style="display:none;">
	<div>
		<div v-for="val in test_arr" track-by="$index">{{val}}</div>
	</div>
</div>
```

## 原因分析

`v-for`遍历数组中存在空值导致页面报错，主要是遍历条件里对值的判断有问题。vue为了保证对dom节点的复用，内置了一份按照id存取的dom缓存，通过对数据分析出dom_id，然后根据此id从缓存中获取dom节点。由于不同的数据取到了相同的dom_id，所以没有创建dom节点出来。但是，在最终数组置空，模型变更之后dom节点移除的时候却为这些dom节点触发了remove操作，也就是方案一中兼容的那些代码。

```JavaScript
id = getTrackByKey(index, key, value, trackByKey);
if (!cache[id]) {
	cache[id] = frag;
}
```

所以问题必定出现在`getTrackByKey`这个函数的执行上，以下是`getTrackByKey`的代码：

```JavaScript
function getTrackByKey(index, key, value, trackByKey) {
    return trackByKey ? trackByKey === '$index' ? index : trackByKey.charAt(0).match(/\w/) ? getPath(value, trackByKey) : value[trackByKey] : key || value;
}
```

`return`之后的判断条件导致了如果`v-for`的目标是一个数组，则最终就会按照`value`来存储你dom节点，`value`就会作为此dom节点的id标识，那就导致了如果一个**非对象数组**中的值有重复的则必然会发生此问题，我们验证后也确实如此，不仅仅是数组中有空值，数组中有重复值也会报错并中断js的执行。修复此问题可以通过下面的代码：

```JavaScript
function getTrackByKey(index, key, value, trackByKey) {
    return trackByKey ? trackByKey === '$index' ? index : trackByKey.charAt(0).match(/\w/) ? getPath(value, trackByKey) : value[trackByKey] : key || (typeof value=="object"?value:index);
}
```

## 总结

`vue`中对数据绑定的操作大大提高了开发者应用开发的效率，但与此同时也伴随着一些不易察觉的问题，尤其如本文中问题的重现条件比较复杂的情况下，测试不一定可以覆盖到问题的触发条件，这个时候就需要我们开发人员多一分警惕。

本文中的修复方法在lighting中已经做了集成，将在下一个版本中包含此补丁。