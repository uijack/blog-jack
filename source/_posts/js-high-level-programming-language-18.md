title: "读《JavaScript高级程序设计》总结18"
date: 2016-01-04 07:41:22
description: "读《JavaScript高级程序设计 -- 第18章 JavaScript 与 XML》介绍浏览器对XML DOM的支持,对XPath的支持，对XSLT的支持"
tags:
- "JavaScript"
---



## 浏览器对XML DOM的支持

### DOM2级核心

在`documen.implementation中引入`createDocument()`方法。IE9+和其他浏览器都支持。
创建空白XML文档：

```js
var xmldom = document.implementation.createDocument("", "root", null);
        
alert(xmldom.documentElement.tagName);  //"root"

var child = xmldom.createElement("child");
xmldom.documentElement.appendChild(child);
```

检测浏览器是否支持DOM2级XML：

```js
  var hasXmlDom = document.implementation.hasFeature('XML', '2.0');
```

### DOMParser 类型

解析XML

### XMLSerializer 类型

将DOM文档序列化为XML字符串。

### 跨浏览器处理XML

## 浏览器对XPath的支持

XPath是设计用来对DOM文档中查找节点的一种手段。XPath是在DOM3级才跻身推荐标准列。
检测浏览器是否支持DOM3级XML：

```js
  var hasXmlDom = document.implementation.hasFeature('XPath', '3.0');
```

### IE中的XPath

### 跨浏览器使用XML

## 浏览器对XSLT的支持

略。。。 没有成为标准。只有IE支持。




-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
