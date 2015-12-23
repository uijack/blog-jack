title: "读《JavaScript高级程序设计》总结1"
date: 2015-12-21 19:51:59
description: "读《JavaScript高级程序设计 -- 第1、2章 JavaScript和HTML》总结, 包括js简洁、版本、引入等"
tags:
- JavaScript
- "js"
- "随笔"
---

## 第1章 JavaScript简介

`JavaScript`诞生于1995年，

在电话拨号上网年代（js问世之前），必须把表单数据发送到服务器才能确定用户是否填写某个必填域，是否输入无效值。数据交换慢。

`JS`出现的目的：处理由服务器语言（如Perl）负责的一些输入验证操作。后来具备与浏览器窗口及其内容交互能力。

`当前局势`：已成为一门功能全面的编程语言，能够处理复杂的计算和交互，拥有了闭包、匿名（lamda）函数、元编程等特性。总之一句话，`JavaScript`是一门简单而又复杂的语言。

### JavaScript简史

由于Web日益流行，网速为28.8kbit/s的猫（调制解调器）上网外加网页大小和复杂性不断增加，和服务器交互频繁。就职于Netscape的布兰登.艾奇（Brendan Eich）计划1995年2月发布了名为LiveScript的脚本语言。为了搭上媒体热炒`Java`的顺风车，临时改名为`JavaScript`.

后来微软也推出JScript（名字避开有关授权）3个不同版本的脚本语言，双方竞争激励，差异较大。JavaScript标准化被提上日程。

1997年，以JavaScript 1.1为蓝本的建议被提交给`欧洲计算机制造商协会（ECMA，European Computer Manufacturers Association）`。该协会指定39号技术委员会（TC39，Technical Committee #39）负责“标准化、跨平台、供应商中立的脚本语言的语法和语义”。TC39由来自NetScape、Sun、微软等的程序员组成。经过数月完成`ECMA-262(定义一种名为ECMAScript[发音“ek-ma-script”])`的新脚本语言的标准。

第二年，ISO/IEC(国际标准化组织和国际电工委员会)也采用ECMAScript作为标准（ISO/IEC-16262）。自此，浏览器开发商就开始致力于将ECMASCript作为各自JavsaScript实现的基础。

### JavaScript实现

虽然JavaScript 和 ECMAScript通常表达相同含义。但JavaScript的含义比ECMA-262中规定的多。一个完整的JavaScript实现有三部分组成：

```bash
核心（ECMAScript）
文档对象模型（DOM）
浏览器对象模型（BOM）
```

`ECMAScript`：ECMA-262与Web浏览器没有依赖关系，Web浏览器只是ECMAScript实现可能的`宿主环境之一`，Node（服务端JavaScript平台）和 Adobe Flash也是其宿主环境。到2008年五大主流浏览器（IE、Firefox、Safari、Chrome、Opera）全部做到了与ECMA-262兼容。

`DOM`：Document Object Model。由负责制定通信标准的W3C（World Wide Web Consortium 万维网联盟）规划。DOM1主要是映射文档结构。DOM2增加DOM视图、DOM事件、DOM样式、DOM遍历和范围。DOM3对DOM核心进行扩展，开始支持XML 1.0规范。

`BOM`：Brower Object Model。包括：弹出新浏览器窗口、移动缩放关闭浏览器、提供浏览器详细信息（Navigator）对象、浏览器加载页面详细信息（location）对象、用户显示器分分辨率（screen）对象、Cookie、XMLHttpRequest和IE的ActiveXObject对象。

## 第2章 在 HTML 中 使用 JavaScript

###  script 元素


HTML4.01 为 `script` 定义六个属性：

```plain
1、async：可选。表示立即下载，不妨碍页面中的其他操作。如继续加载脚本和下载资源。只对外部有效

2、charset：可选。很少用

3、defer：可选。脚本推迟到文档完全被解析和显示之后再执行。只对外部文件有效

4、language：已废弃

5、src：可选。包含要执行代码的外部文件

6、type： 可选，一般为text/javascript.
服务器传送JavaScript文件时MIME类型通常为：application/x-javascript.
```

注意：
```plain
<script type="text/javascript">
  function sayScript() {
    alert("</script>");
  }
</script>
```

以上会报错

```plain
<script type="text/javascript">
  function sayScript() {
    alert("<\/script>");
  }
</script>
```
以上才是正确，在代码中任何地方都不要出现`</script>`

### 标签加载方式

将js放在`head`标签中的坏处：一位置必须等到全部js代码被下载解析和执行后才开始呈现页面内容及遇到`body`标签。会导致页面出现明显延迟。窗口一片空白。

正确做法：放在`body`元素页面的内容后面。

如果想要放在head又希望最后加载可用添加`defer=“defer”`属性。而`sync`并不能保证依照先后顺序执行。

在XHTML中CData片段是文档的一个特殊区域。为了兼容XHTML的浏览器。
应该这样做：
```js
<script type="text/javascript">
//<![CDATA[
  function sayScript() {
    alert("<\/script>");
  }
//]]>
</script>
```

保证HTML和XHTML都正常执行。

### 文档模式

IE5.5 引入 混杂模式（quirks mode，包含非ECMA-262标准特性）和 标准模式（sytandard mode）

`标准模式`可以通过使用下面任何一种文档类型来开启：

```plain
<!--  HTML 4.01 严格型 -->
<! DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
```

```plain
<!--  XHTML 1.0 严格型 -->
<! DOCTYPE html PUBLIC "-//W3C//DTD HTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
```

```plain
<!--  HTML 5 -->
<! DOCTYPE html5>
```

而对于准标准模式，可以通过渡型（transitional）和框架集型（frameset）文档类型来触发，如下：

```plain
<!--  HTML 4.01 过渡型 -->
<! DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
```

```plain
<!--  HTML 4.01 框架集型 -->
<! DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">
```

```plain
<!--  XHTML 1.0 过渡型 -->
<! DOCTYPE html PUBLIC "-//W3C//DTD HTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
```

```plain
<!--  XHTML 1.0 框架集型 -->
<! DOCTYPE html PUBLIC "-//W3C//DTD HTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">
```

### <noscript> 元素

一下两种情况：
1、浏览器不支持脚本
2、浏览器支持，但脚本被禁用

出现以上一种情况，`noscript`标签的内容才会出现。

```plain
<body>
  <noscript>
    <p>本页面需要浏览器支持（启用）JavaScript。</p>
  </noscript>
</body>
```

## 参考文档

- [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)

-----------------------

> 文章若有纰漏，请大家补充指正，谢谢！
> 如有疑问可留言，我会在第一时间回复。
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
