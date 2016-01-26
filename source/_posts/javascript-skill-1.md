title: "Web前端你应该了解的知识(1)"
date: 2016-01-26 08:06:58
description: "收集作为前端开发人员应该知道的知识,分享.包括position,"
tags:
- css
---

## 定位position的值

### position 4种值:

```plain
static: 默认值,没有定位,元素正常出现在文档中
relative: 生成相对定位的元素,相对于其在普通流中的位置进行定位
absolute: 生成绝对定位的元素,相对于最近一级的定位不是static的父元素来进行定位
fixed: (老IE不支持）生成绝对定位的元素，相对于浏览器窗口(html元素)进行定位。
```

> 1. fixed固定定位在IE6,IE7(怪异(quirk)模式),IE8(怪异模式)浏览器下会将`fixed`值当做错误值处理.从而导致其position值为默认的`static`.

> 2. 如果position的值fixed或absolute但没有设置top,bottom,left,right中的任何一个值.则该元素会在值为position:static的地方脱离文档流.如果有多个并列的fixed或absolute(都未设置),则会重叠在一起.后一个position值为static或relative的元素将会在向前占位

```plain
<body style="position: relative;margin-top:100px;">
  <div>AAA</div>
  <div style="position:relative;">BBB</div>
  <div style="position:fixed;">III</div>
  <div style="position:absolute;">CCC</div>
  <div style="position:absolute;">DDD</div>
  <div style="">EEE</div>
  <div style="position:absolute;">GGG</div>
</body>
```

> 以上排版结果为: AAA在第1行;BBB在第2行;III,CCC,DDD,EEE在第3行(EEE会向前占位),GGG在第4行.

### 验证absolute的定位

```plain
<body style="position: relative;margin-top:100px;">
  <div>AAA</div>
  <div style="position: fixed;top: 0;">
    <div>
      WWW
    </div>
    <div style="position: absolute;top: 0;">
      XXX
    </div>
  </div>
</body>
```plain

> 结果是: WWW和XXX重叠在一起,顶着浏览器;而AAA在距离顶部100像素处.此时元素div的宽度不再是占满一行,而是随内容撑开,设置width:100%无效,需要给元素设置width属性和height属性.

```plain
<body style="position: relative;margin-top:100px;">
  <div>AAA</div>
  <div style="position: absolute; left: 100px;top: 0;">
    <div>
      WWW
    </div>
    <div style="position: absolute;left: 100px;top: 0">
      XXX
    </div>
  </div>
</body>
```

> 此时AAA,WWW,XXX在一行,WWW距离body左边100px, XXX距离WWW左边100px;上面对于absolute的描述是正确的.

### position:absolute和float属性的异同

共同点：对内联元素设置float和absolute属性，可以让元素脱离文档流，并且可以设置其宽高。
不同点：float仍会占据位置，position会覆盖文档流中的其他元素。

## 创建Ajax过程

```plain
(1)创建XMLHttpRequest对象,也就是创建一个异步调用对象.

(2)创建一个新的HTTP请求,并指定该HTTP请求的方法、URL及验证信息.

(3)设置响应HTTP请求状态变化的函数.

(4)发送HTTP请求.

(5)获取异步调用返回的数据.

(6)使用JavaScript和DOM实现局部刷新.
```

## 对闭包的理解

使用闭包主要是为了设计私有的方法和变量。闭包的优点是可以避免全局变量的污染，缺点是闭包会常驻内存，会增大内存使用量，使用不当很容易造成内存泄露。

闭包有三个特性：

> 1.函数嵌套函数
> 2.函数内部可以引用外部的参数和变量
> 3.参数和变量不会被垃圾回收机制回收
