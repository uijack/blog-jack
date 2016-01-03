title: "读《JavaScript高级程序设计》总结16"
date: 2016-01-03 8:08:54
description: "读《JavaScript高级程序设计 -- 第16章 HTML5脚本编程》介绍原生拖动,媒体元素和历史状态管理"
tags:
- "JavaScript"
- "随笔"
---


## 跨文档消息传递

cross-document messaging 简称`XDM`,指来自不同域的页面间的传递消息。如www.wrox.com域中的页面与位于一个内嵌框架中的p2p.wrox.com域中的页面通信。核心方法就是`postMessage()`方法。

接受两个参数： 一条消息和一个表示消息接收来自那个域的字符串。第二个参数对保障安全通信非常重要。可以组织浏览器把消息发送到不安全的地方。

```js
// 所有支持XDM的浏览器也支持iframe的contentWindow属性
var iframeWindow = document.getElementById('myframe').contentWindow;
iframeWindow.postMessage('A secret', 'http://www.wrox,com'); // 来源不对什么也不做
```

> data: 作为psotMessage()第一个参数传入的字符串数据。
> origin: 发送消息的文档所在域，例如http://www.wrox.com
> source: 发送消息文档的window对象的代理。

```js
EventUtil.addHandleer(window, 'message', function( event ) {
  // 确保发送消息的域是已知的域
  if (event.origin == 'http://www.wrox.com') {
    // 处理接收消息的数据
    processMessage(event.data);

    // 可选：向来源窗口发送回执
    event.source.postMessage('Received!', 'http://p2p.wrox.com');
  }
});
```

## 原生拖放

网页中只有两种元素可以拖放：图像和某些文本。

### 拖放事件

依次触发下列事件：
1. dragstart
2. drag // 被拖动期间持续触发该事件
3. dragend

### 那个定义放置目标
### dataTransfer 对象
### 属性dropEffect 与 effectAllowed
### 可拖动
### 其他成员

## 媒体元素

`video`标签和`audio`标签

### 标签共有属性

### 事件

可以做音乐播放器

### 自定义媒体播放器

### 检测编解码器的支持情况

audio.canPlayType('audio/mpeg')

各种支持格式的字符串和浏览器支持情况。

### Audio类型

## 历史状态管理

首选`haschange事件`。




-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
