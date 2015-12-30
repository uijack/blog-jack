title: "读《JavaScript高级程序设计》总结12"
date: 2015-12-30 19:18:24
description: "读《JavaScript高级程序设计 -- 第12章 DOM2和DOM3》主要介绍DOM变化,样式,遍历,范围"
tags:
- "JavaScript"
- "随笔"
---

## DOM 变化

`DOM1级`主要定义的是HTML和XML文档的底层结构,
`DMO2`和`DOM3级`则在`DOM1级`基础上引入了更多的交互能力,也持之更高级的XML特性.

DOM2和DOM3分为许多模块:

1. DOM2级核心: 增添更是多属性和方法.
2. DOM2级视图: 为文档定义了基本样式信息的不同视图.
3. DOM2级事件: 说明如何使用事件与DOM交互.
4. DOM2级样式: 如何以编程方式来访问和改变CSS样式信息.
5. DOM2级遍历和范围: 引入遍历DOM文档和选择其特定部分接口.
5. DOM2级HTML: 在一级HTML基础上构建,添加更多的属性,方法和新接口.

## 样式

css属性----------------JavaScript属性
background-image------style.backgroundImage
color-----------------style.color
display---------------style.display

但是`float`是JavaScript中的保留字,因此不能做属性名.`DOM2`规定为对应的属性名应该是`cssFloat`,唯独IE支持`styleFloat`.

- DOM样式属性和方法

cssText, length, parentRule, getPropertyCSSValue(propertyName), getPropertyValue(propertyName), item(index),
removeProperty(propertyName), setProperty(propertyName, value, priority).

- 样式计算

### 操作样式表

只做了解.

### 遍历

## NodeIterator

只做了解.

## TreeWalker

只做了解.

## 范围

- createRange()



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
