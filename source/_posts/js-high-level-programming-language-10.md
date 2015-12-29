title: "读《JavaScript高级程序设计》总结10"
date: 2015-12-29 09:14:43
description: "读《JavaScript高级程序设计 -- 第10章 DOM》主要各种节点类型和DOM操作技术"
tags:
- "JavaScript"
- "随笔"
---

## 节点层次

DOM是针对HTML和XML文档的一个API(应用程序编程接口),描述的是一个层次化的节点树.
IE中的所有对象都是以COM对象的形式实现,这意味着IE中的DOM对象与原生JavaScript对象的行为或活动特点并不一致.
总共有`12种节点类型`,都继承自一个`基类Node`.

### Node 类型

`DOM1级`定义了一个`Node接口`,由DOM中的所有节点类型实现.这个Node除了IE之外, 其他浏览器都可以访问到这个类型.
所有节点类型都共享相同的属性和方法.

每个节点都有一个`nodeType`属性,用于表明节点类型.

```plain
1,  Node.ELEMENT_NODE(1);
2,  Node.ATTRIBUTE_NODE(2);
3,  Node.TEXT_NODE(3);
4,  Node.CDATA_SECTION_NODE(4);
5,  Node.ENTITY_REFERENCE_NODE(5);
6,  Node.ENTITY_NODE(6);
7,  Node.PROCESSING_INSTRUCTION_NODE(7);
8,  Node.COMMENT_NODE(8);
9,  Node.DOCUMENT_NODE(9);
10, Node.DOCUMENT_TYPE_NODE(10);
11, Node.DOCUMENT_FRAGMENT_NODE(11);
12, Node.NOTATION_NODE(12);
```
比较这12种类型,可以确定节点类型:

```js
if (someNode.nodeType == Node.ELEMENT_NODE) { //在IE中不能访问Node, 所以在IE中无效
  alert('Node is an element.');
}

// 可以进行数值比较 推荐
if (someNode.nodeType == 1) {  // 使用所有浏览器
  alert('Node is an element.');
}
```

使用下面两个属性前,先确定类型: 是元素节点,nodeName保存的是元素的标签名,而nodeValue始终为null.

> nodeName  
> nodeValue

每个节点都有一个childNodes属性.保存着NodeList对象.
访问方法:
1, 通过方括号访问,
2, 使用item()方法.
NodeList也有length属性,但它不是Array的实例.

```js
var firstChild  = someNode.childNodes[0];
var secondChild = someNode.childNodes.item(1);
var count       = someNode.childNodes.length;
```
`arguments`对象使用Array.prototype.slice()将其转化为数组;`NodeList`也可以.
var arrayNodes = Array.prototype.slice.call(someNode.childNodes, 0); // IE8及之前版本都无效

```js
function converToArray( nodes ) {
  var array = null;
  try {
    array = Array.prototype.slice.call(someNode.childNodes, 0);// 针对非IE浏览器
  } catch (ex) {
    array = new Array();
    for (var i=0, len=nodes.length; i < len; i++) {
      array.push(nodes[i]);
    }
  }

  return array;
}
```

每个节点都有一个`parentNode`属性.改属性指向文档树中的父亲节点.包含在childNodes列表中的所有节点都具有相同的父节点.

> parentNode属性: 父节点
> previousSibling属性: 同级的上一个节点,没有为null
> nextSibling属性: 同级的下一个节点,没有为null
> hasChildNodes(): 有机返回true.
> ownerDocument属性: 该属性指向表示整个文档的文档节点.

![节点类型关系图](/img/js-high-level-10-1.png)

- 操作节点:

> appendChild()
> insertBefore()
> replaceChild()
> removeChild()

- 其他方法:

> cloneNode(); 不会复制添加到DOM节点中的JavaScript属性,如事件处理,
> normalize();

### Document 类型

在浏览器中,document对象是HTMLDocument(继承自Document)的一个实例,表示整个html页面,documen对象也是window对象的一个属性.
Document节点特征:

```plain
nodeType:9
nodeName: '#document'
nodeValue: null
parentNode: null
ownerDocument: null
其子节点可能是一个DocumentType(最多一个),Element(最多一个),ProcessingInstruction或Comment.
```

- 文档的子节点

内置2个访问其子节点的快捷方式:

> documentElement属性: 始终指向页面中的<html>元素
> childNodes列表访问文档元素
> body属性
> doctype属性:用处有限,因为浏览器支持不一.

documentElement,firstChild,childNodes[0]都指向`<html>`元素

- 文档信息

> docuemnt.title;  // 取得标题
> document.URL;    // 取得完整的URL
> document.domain; // 取得域名
> document.referrer; // 取得来源页面的URL

- 查找元素

> getElementById()
> getElementByTagName()
> getElementByName() // 单选按钮

- 特殊集合

> document.anchors
> document.applets
> document.forms;
> document.images;
> document.links;

- DOM一致性检测

检测浏览器实现了DOM的那些部分就十分必要. document.implementation属性
- hasFeature(要检测的DOM功能名称, 版本号);

- 文档写入

> write()
> writeln()
> open()
> close()

### Element 类型

```plain
nodeType:1
nodeName: 元素标签名(全是大写)
nodeValue: null
parentNode: Document或Element
其子节点可能是: Element,Text,Comment,ProcessingInstruction,CDATASection或EntityReference.
```

- HTML元素

所有HTML元素都是由HTMLElementl类型表示,不是直接通过这个类型,而是通过它的子类型来表示.HTMLElement类型直接继承Element并添加了一些属性.

> id
> title
> lang
> dir: 语言方向,值为'ltr',(left-to-right) 或 rtl (right-to-left)
> className

- 取得特性与设置特性

`getAttribute`, `setAttribute`.`removeAttribute`

var div = document.getElementById('myDiv');
div.getAttribute('id');
div.getAttribute('class');
div.getAttribute('title');
div.getAttribute('dir');

不存在则返回null;

- attribute属性

Element类型是使用attribute属性的唯一一个DOM节点类型,包含以下属性:

> getNamedItem(name)
> setNamedItem(name)
> removeNamedItem(name)
> item(pos)

- 创建元素:

> createElement();
> appendChild();

### Text 类型(重要)

```plain
nodeType:3
nodeName: '#text'
nodeValue: 节点所包含的文本
parentNode: Element
不支持(没有)子节点
```

可以通过nodeValue属性或者datas属性访问Text节点中包含的文本.这两个属性值中包含的值相同,
修改其中之一反映到另外一个.

> appendData(text)
> deleteData(offset, count)
> insertData(offset, text)
> replaceData(offset, count, text)
> splitText(offset)
> substringData(offset, count)
> length属性,保存节点字符的数目 nodeValue和data也有length属性

创建文本节点:document.createTextNode('文本内容');

### Comment 类型

```plain
nodeType:8
nodeName: '#commnet'
nodeValue: 注释的内容
parentNode: Document或Element
不支持(没有)子元素
```

`Commnet` 和  `Text`继承自相同的基类.

document.createComment('A Comment');

### CDATASection 类型

只针对基于XML的文档

```plain
nodeType: 4
nodeName: '#cdata-section'
nodeValue: CDATA区域的内容
parentNode: Document或Element
不支持(没有)子元素
```

### DocumentType 类型

并不常用仅有Firefox,Safari,Opera支持.Chrome4.0才支持

```plain
nodeType: 10
nodeName: doctype的名称
nodeValue: null
parentNode: Document
不支持(没有)子元素
```

### DocumentFragment 类型(很有用)

在文档中没有对应的标记,DOM规定文档片段是一种'轻量级'的文档,可以包含和控制节点.不会像完整文档那样占用资源.

```plain
nodeType: 11
nodeName: '#document-fragment'
nodeValue: null
parentNode: null
子节点: Element, ProcessingInstruction, Comment, Text, CDATASection, EntityReference
```
可以作为一个仓库,保存可能会添加到文档中的节点.

var gragment = document.createDocumentFragment();

设想我们想为ul元素添加3个列表项.逐个添加会导致浏览器反复渲染新信息,为避免问题.可以先将每一个li创建预存在fragment对象上,一次性添加到ul中即可.

### Attr 类型

```plain
nodeType: 11
nodeName: 特性的名称
nodeValue: 特性的值
parentNode: null
specified: 布尔值
子节点: 在HTML中不支持(没有)子节点,在XML中子节点可以是Text, EntityReference.
```

尽管是节点,但特性却不被认为是DOM文档树德一部分.常用方法:

> getAttribute()
> setAttribute()
> removeAttribute()

```js
var attr = document.createAttribute('align');
attr.value = 'left';
element.setAttribute(attr);

cosole.log(element.attributes['align'].value); // 'left'
cosole.log(element.getAttributeNode('align').value); // 'left'
cosole.log(element.getAttribute('align')); // 'left'

```

## DOM 操作技术

因为浏览器隐藏着陷阱和不兼容问题.

### 动态脚本(*****)

```js
// 创建 <!-- <script type="text/javascript" src="client.js"></script> -->
var script = document.createElement('script');
script.type = 'text/javascript';
script.src = 'client.js';
document.body.appendChild(script);
```
封装上面的代码成函数;

```js
fucntion loadScript( url ) {
  var script = document.createElement('script');
  script.type = 'text/javascript';
  script.src = url;
  document.body.appendChild(script);
}
```

加载内部js片段:

```js
function loadScriptString(code){
  var script = document.createElement("script");
  script.type = "text/javascript";
  try {
    // IE将script视为特殊的元素,不允许DOM访问其子节点,不过可以用text属性来指定js片段
    script.appendChild(document.createTextNode(code));
  } catch (ex){
    script.text = code;
  }
  document.body.appendChild(script);
}

function addScript(){
  loadScriptString("function sayHi(){alert('hi');}");
  sayHi();
}
```

### 动态样式

```js
function addStyle(){
function loadStyleString(css){
  var style = document.createElement("style");
  style.type = "text/css";
  try{
    style.appendChild(document.createTextNode(css)); // //error in IE
  } catch (ex){
    style.styleSheet.cssText = css;
  }
  var head = document.getElementsByTagName("head")[0];
  head.appendChild(style);
}

function addStyle(){
  loadStyleString("body{background-color:red}"); 
}
```

### 操作表格

省略...

### 使用NodeList

应该尽量减少访问NodeList的次数,因为每次访问NodeList,都会运行一次基于文档的查询, 所以,可以考虑将从NodeList中取得的值缓存起来.

## 参考文档

- [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)

-----------------------

> 文章若有纰漏，请大家补充指正，谢谢！
> 如有疑问可留言，我会在第一时间回复。
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
