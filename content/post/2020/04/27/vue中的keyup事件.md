---
title: vue中的keyup事件
date: 2020-04-27 18:59:06
tags:
- vue
categories:
- vue
---

## vue中的keyup事件

记录一下使用keyup遇到的几个问题

<!--more-->

### 组合键和单个键复用

有时候会需要同时用到组合键和单个键

这时对于复合键之间加上即可，但对于单个键则需要添加修饰符`exact`

```vue
@keyup.enter.exact="send" 
@keydown.ctrl.enter="getLine">
```

这样当按下ctrl+enter会触发getLine

只按下enter才会触发send

### 回车按键与输入法冲突问题

中文输入法中经常用到使用回车来打印英文，而不是中文。

当在js中又监听了回车按键的事件，因此会被触发

```vue
@keyup.enter.exact="send" 
```

使用keyup来触发回车弹起的事件：

- 在chrome浏览器中可以正常实现只把英文显示而不会触发事件
- 当在firefox浏览器中回车会把英文显示，然后直接触发事件

**因此把keyup修改为keydown事件**

```vue
@keydown.enter.exact="send"
```

该为keydown事件后两个浏览器都可以正常实现，只显示英文，而不会触发事件。

当在textarea中做文本框输入时，虽然不会触发事件，但在主动回车发送消息后会在文本框中留下一个换行符。

上面的问题可能是因为keydown是可以多次触发的缘故。

**因此尝试禁止默认事件**

```vue
event.preventDefault()
```

终于可以正常实现了。