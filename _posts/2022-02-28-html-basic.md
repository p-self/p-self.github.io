---
layout: post
title:  "一些关于HTML的基础知识（一）"
date:   2022-02-28 14:34:12 +0800
categories: dev html
---

## HTML标签的全局属性

html标签（tag）中存在一些所有的tag都可以使用但作用不同的属性（attribute），经常使用的属性如：style、class、id等，以下整理了一些可能没那么经常使用的属性。

###  aria-* 

html中所有的`aria-*`相关的属性都是为了提供视觉障碍人士的使用（accessibility），从功能设计的角度所有的标签都应该配带`aria-*`信息以支持少数群体对页面的阅读。

### event handler

全部的事件绑定属性为：onabort, onautocomplete, onautocompleteerror, onblur, oncancel, oncanplay, oncanplaythrough, onchange, onclick, onclose, oncontextmenu, oncuechange, ondblclick, ondrag, ondragend, ondragenter, ondragleave, ondragover, ondragstart, ondrop, ondurationchange, onemptied, onended, onerror, onfocus, oninput, oninvalid, onkeydown, onkeypress, onkeyup, onload, onloadeddata, onloadedmetadata, onloadstart, onmousedown, onmouseenter, onmouseleave, onmousemove, onmouseout, onmouseover, onmouseup, onmousewheel, onpause, onplay, onplaying, onprogress, onratechange, onreset, onresize, onscroll, onseeked, onseeking, onselect, onshow, onsort, onstalled, onsubmit, onsuspend, ontimeupdate, ontoggle, onvolumechange, onwaiting。

以上的全部事件绑定属性在所有的标签中均可以使用，尽管有些事件对此标签而言并不会触发。

### autocapitalize

此属性用于声明段落的字母大写转换的信息，在英语环境下比较常用，`autocapitalize`的值有三种取值情况：

1. `off` 或者 `none`, 不起用大写转换
2. `on` or `sentences`，对首字母进行大小写转换
3. `words` ，对段落中的每一个单词的首字母都大写
4. `characters`，段落中的所有字母都需要大写

### contextmenu

此属性声明此标签右键点击后的右键菜单；右键菜单可以通过`<menu>`标签来 声明。

### enterkeyhint

此属性用于声明回车键在虚拟键盘上的提示文本，在移动端编程时，可用于定义键盘回车键的文本内容；

### hidden

此属性是标准属性，在任意的html标签中均可以使用，代表元素隐藏。

hidden应类似等同于`display:none`，hidden的元素不参与页面渲染，也没有渲染的机会。

### is

此属性在支持`web components`的浏览器中常用，可以将任意的html标签声明为自定义的web组件。

### nonce

A cryptographic nonce ("number used once") which can be used by Content Security Policy to determine whether or not a given fetch will be allowed to proceed.

### tabindex

`tabindex`声明的元素都可以被聚焦（focus），此属性定义在`tab`按键时依次聚焦的顺序。`tabindex`有三种取值状态：

1. 负数，声明元素可以被聚焦，但是不能被`tab`按键聚焦
2. 零，声明元素可以被聚焦，也可以被`tab`按键聚焦，但实际行为有平台的会话状态确定
3. 正数，声明元素可以被聚焦，也可以被`tab`按键聚焦，被聚焦的顺序从小到大，如果使用了相同的值，在先出现者先聚焦


## `<base>`

`<base>`标签用于声明文档中的所有相对路径资源的路径基础地址，一个document中只能有一个`<base>`标签。

```html
<base href="https://pself.net">
<img src="images/DeepinScreenshot_select-area_20170928143205.png" alt="">
```

如以上的案例所展示，`<img>`标签所引用的地址为相对路径，在声明`<base>`的情况，会自动以`href`的定义为基础路径。在未声明`<base>`的情况，会自动以`location.href`的定义的路径为基础路径。

`<base>`有两个扩展属性，`href`用于定义相对路径的资源的base路径，target用于定义 `<a>`, `<area>`, or `<form>`的默认`target`，`target`有以下几个可选值：

1. _self (default): 在当前浏览的上下文显示
2. _blank: 在新建上下文中显示
3. _parent: 如果在iframe中，则是父亲浏览上下文显示；否则等同于_self
4. _top: 在定义的父亲浏览上下文显示，如果没有则等同于_self

当存在页面锚点的情况，则点击后发起以base中定义的href的页面的锚点请求。

1. Given <base href="https://example.com">
2. ...and this link: <a href="#anchor">To anchor</a>
3. ...the link points to https://example.com/#anchor.

## `<href>`

用于定义外部的资源，包括图片、字体、样式表等和当前文档的关系；既可以放置在head，也可以放置在body，通常的做法是在head中包含，以确保文件资源的分离管理；

`charset`和`rev`已经时过时用法，不能再使用了。

```html
<link href="main.css" rel="stylesheet">
```

`<href>`主要支持的属性有以下：

### `as`

当定义了`rel="preload"` or `rel="prefetch"`时，才可以使用`as`用于声明被预加载的资源的类型；

```
audio 	<audio> elements
document 	<iframe> and <frame> elements
embed 	<embed> elements
fetch 	fetch, XHR
font 	CSS @font-face
image 	<img> and <picture> elements with srcset or imageset attributes, SVG <image> elements, CSS *-image rules
object 	<object> elements
script 	<script> elements, Worker importScripts
style 	<link rel=stylesheet> elements, CSS @import
track 	<track> elements
video 	<video> elements
worker 	Worker, SharedWorker
```

### `crossorigin`

`crossorigin`定义请求的资源启用`CROS`机制，比如启用`crossorigin`的图片资源可以在`<canvas>`中使用，而不会污染画布，这样既可以导出base64图片数据以在前端分享和展示。

1. 取值为`anonymous`，此情况下浏览器发起携带`origin`的前端请求，但是不携带会话安全信息如：cookie、X.509 certificate, 或者 HTTP Basic authentication，如果后端的服务器不允许资源请求（Access-Control-Allow-Origin没有设置和响应），在图片获取后被污染，限制画布使用。
2. 取值为`use-credentials`此情况下浏览器发起携带`origin`的前端请求，携带会话安全信息如：cookie、X.509 certificate，如果后端没有正确响应`Access-Control-Allow-Credentials `，在图片被污染，限制画布使用。
3. 如果没有设置`crossorigin`属性，在发起不携带`origin`的前端请求，不交互CORS机制，阻止图片在污染状态下的使用。


### `disabled`

当`rel="stylesheet"`时，可以设置link的属性值为`disabled`，则在页面加载时此样式表不加载；在移除`disabled`属性，或者将`disabled`属性的值设置为`false`时，样式表才开始加载。

`document.styleSheets`中不包含link的属性值为`disabled`的样式表。

### `integrity`

`integrity`的值为引用资源的hash值的base64编码，浏览器可以根据此值判断资源是否遭遇篡改。

### `referrerpolicy`

`referrerpolicy`定义资源的referrer策略。

1. no-referrer，不适用referrer头
2. no-referrer-when-downgrade，当请求非tls请求时，不使用referrer头
3. origin，使用referrer头值为scheme, the host, and the port
4. origin-when-cross-origin，跨域时为origin，同域时为完整路径
5. unsafe-url，完整路径可能会暴露https下的访问路径

### `title`

和标准属性中的titile不同，link中的title配合`<link rel="stylesheet">`，使用配置可选样式表。

```
<link href="default.css" rel="stylesheet" title="Default Style">
<link href="fancy.css" rel="alternate stylesheet" title="Fancy">
<link href="basic.css" rel="alternate stylesheet" title="Basic">
```

经过上面的配置可以在**View > Page Style**中选择对应的样式使用多种主题。

## `<meta>`

用于定义文档中的元数据信息。

1. `name`属性，定义文档全局的元数据信息；
2. `http-equiv`，定义和http响应头等价；
3. `charset`，定义文档的编码字符集格式；

```html
<meta charset="utf-8">
<!-- Redirect page after 3 seconds -->
<meta http-equiv="refresh" content="3;url=https://www.mozilla.org">
```

### `charset`

charse的定义必须存在文档的前1024bytes，否则无法生效。

### `content`

使用`name`或者`http-equiv`时， `content`配合设置value值。