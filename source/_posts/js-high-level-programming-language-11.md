title: "读《JavaScript高级程序设计》总结11"
date: 2015-12-30 07:11:58
description: "读《JavaScript高级程序设计 -- 第11章 DOM扩展》主要介绍选择符API,元素遍历，HTML5以及专有扩展"
tags:
- "JavaScript"
---

## 选择符API

jQuery的核心就是同通过CSS选择符查询DOM文档取得元素的引用。从而抛开`getElementById()`和`getElementByTagName()`
Selectors API Level1 的两个核心方法： `querySelector()`和`querySelectorAll()`.
在兼容浏览器中可以通过`Document`及`Element`类型实例调用。IE8+,Ff3.5+,Chroem,Opera10+支持。

### querySelector()方法

接收一个css选择符（id, 元素, class名）。返回与该模式匹配的第一个元素，没有则返回`null`.如果传入不支持的选择符会抛错。

### querySelectorAll() 方法

同querySelector()一样接受一个css选择符,返回一个`NodeList`的实例。没有就为空。

访问NodeList实例可以用item()方法或者使用方括号语法。

### matchesSelector() 方法

Selector API Level 2 规范为Element类型新增了一个方法`matchesSelector(css选择符)`匹配返回true,否则返回false.
该方法各大浏览器支持性不是很好.

## 元素遍历

Element Traversal 规范 IE9+ Ff3.5 Saf4+ Chrome Opera10+支持。

- childElementCount: 返回子元素(不包括文本节点和注释)的个数
- firstElementChild: 指向第一个子元素, firstChild的元素版
- lastElementChild: 指向最后一个子元素
- previousElementSibling: 指向前一个同辈元素
- nextElementSibling: 指向后一个同辈元素

利用这些元素不必担心空白文本节点.

## HTML5

- getElementsByClassName(). 最受欢迎的

### 焦点管理

document.activeElement属性;

### HTMLDocument 的变化.

- readyState 属性

两个值: 1. loading 正在加载文档. 2. complete, 已经加载完文档.

- 兼容模式

标准模式下: document.compatMode == 'CSS1Compat'
混杂模式下: document.compatMode == 'BackCompat'

- head 属性

H5新增document.head属性.

var head = document.head || document.getElementByTagName('head')[0];

- 字符集属性

默认为: UTF-16.
> document.charset = 'UTF-8'; //IE, Safari, Chrome, Firefox, Opera.
> document.defaultCharset属性.文档默认字符集. // IE, Safari, Chrome

- 自定义数据属性 (很重要)

HTML5规定可以为元素添加非标准属性,但要添加`前缀data-`.目的是为元素提供与渲染无关的信息.或者语义信息.

```plain
<div id="myDiv" data-appId="12345" data-myname="Jack"></div>
```

```js
var div = document.getElementById('myDiv');

// 取得自定义属性的值
var appId = div.dataset.appId;
var myName = div.dataset.myname;

// 设置值
div.dataset.appId = 23456;
div.dataset.myname = 'Chan';

// 没有为空
```
可以通过元素的`dataset`属性来访问自定义属性值.dataset属性的值是`DOMStringMap`的一个实例.也就是`键值对`的映射.
每个data-name形式的属性都会对应一个属性,只不过属性名没有`data-前缀`.

### 插入标记

- innerHTML 属性 (IE下注意script)

最好先把所有的html代码拼接成字符串,在一次性插入,避免单次插入影响性能损失.

- outerHTML 属性

### scrollIntoView()() 方法

HTML5最终选择`scrollIntoView()`作为标准方法. document.forms[0].scrollIntoView(). 支持的有IE,Firefox,Safsri,Opera.

## 专有属性

### 文档模式

IE8引入一个新的概念叫`文档模式`.页面的文档模式决定可以使用什么功能.即决定你使用那个级别的CSS,在JS中可以使用那些API,
以及如何对待文档类型(doctype).
1. IE5,混杂模式渲染页面.IE8及更高版本新功能都无法使用
2. IE7, 以IE7标准模式渲染页面,IE8及更高版本新功能都无法使用
3. IE8,以IE8标准模式渲染页面, 可以使用更过的CSS2和某些CSS3功能.一些HTML5功能
4. IE9,以IE9标准模式渲染页面,支持ECMAScript 5,完整CSS3.

强制浏览器以某种模式渲染页面,可以使用HTTP头部信息`X-UA-Compatible`,通过meta标签设置:

```plain
<meta http-equiv="X-UA-Compatible" content="IE=IEVersion" />
```

`IEVersion`的值有些不同:
1. Edge: 始终以最新文档模式渲染页面
2. 9, 8, 7, 5

想知道页面采取的是什么文档模式,有助于理解页面的行为.无论什么文档模式下都可以访问这个属性:

`var mode = document.documentMode`;

> children 属性
> contains()
> 插入文本: innerTEXT,outerText 属性

### 滚动

> scrollIntoViewIfNeeded(alignCenter);
> scrollByLines(lineCount);
> scrollByPages(pagesCount);



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
