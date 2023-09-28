---
layout: post
title:  "一种可交互（JS触发）vue组件的实现方式"
date:   2017-08-24 08:44:56 +0800
categories: dev vue
---

前端组件化的一个重要的问题就是ui状态的产生和复用，vue中提供了讲一个功能/模块组件化的高效方案--vue单文件组件；在使用vue作为开发框架的过程当中，使用vue单文件组件作为ui组件的实现方案可以大大提高代码的复用程度。

但是，vue组件中也面临了一个无法解决和处理的问题：

1. 对于有些组件，比如alert等弹出层是通过js代码交互的方式触发和改变组件状态/行为的
2. 如果对于每一个此种组件的使用都需要对应组件标签的书写和声明，这样就导致了页面中大量已声明但是未使用的UI组件
3. 每一个UI组件的使用都是不复用的（复用有可能造成状态的残留，轻易的导致问题的产生），但是不可能为每一个操作都事先声明一个组件

本文描述的方案就是将，vue单文件组件转化为可以交互控制（JavaScript控制的组件），以下对这一个类型的组件统称为**通用编程式组件**，其中的典型是：弹出层/交互式组件！

以下是关键实现的代码：

1. 声明并实现一个ui组件（通过vue单文件组件）

```html
<template>
    <div>
        <div class="alert">
            {{msg}}
        </div>
    </div>
</template>
<script>
    export default {
        data(){
            return {
            };
        },
        props:['msg']
    }
</script>
<style lang="less">
    .alert{
        position: fixed;
        width: 100px;
        top: 50%;
        left:50%;
        margin-left: -100px;
        z-index: 100;
        color:red;
        font-size:30px;
    }
    .alert:before{
        content: ' ';
        display: inline-block;
        position: fixed;
        z-index: 99;
        top:0;
        bottom: 0;
        left: 0;
        right: 0;
        background-color: rgba(0,0,0,0.6);
    }
</style>
```

2. 将此UI组件包装成为`通用编程式组件`

```javascript
window.Alert = {
    alert:function (msg) {
        const dom = `
            <div>
                <vue-alert :msg="${msg}"></vue-alert>
            </div>
        `;

        const wrap = document.createElement("div");
        wrap.innerHTML = dom;
        document.body.appendChild(wrap);

        new Light.Model({
            el:wrap,
            component:{
                vueAlert:require("../ui/alert.vue")//如果是在lighting<1.4的情况下，由于组件都是全局的，这一句就不需要的，只要组件的目录结构正确就可以了！vue/alert.vue
            }
        })
    }
}
```

3. 接下来的步骤，就可以直接使用第二部暴露成功的全局变量来调用和触发alert组件

```javascript
Alert.alert("hello,world");
```

总结，使用vue单文件组件在解决我们代码复用的问题的同时，也改变了之前我们处理问题的方案，因地制宜的选择合适的方法和流程程序的开发和健壮性的达成具有至关重要的作用！