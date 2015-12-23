title: "读《JavaScript高级程序设计》总结4"
date: 2015-12-23 17:08:55
description: "读《JavaScript高级程序设计 -- 第4章 变量、作用域和内存问题》总结,本章主要介绍基本类型、引用类型、执行环境、作用域、垃圾收集等"
tags:
- JavaScript
- "js"
- "随笔"
---

## 基本类型 和 引用类型

- 基本数据类型：指简单数据段（包含：`Undefined`, `Null`, `Boolean`, `Number`, `String` ）
- 引用数据类型：指那些可能有多个值构成的对象 (Object, null)

在将一个值赋给变量时，解析器必须确定这个值是基本类型还是引用类型
`基本数据类型`是按值访问，而`引用数据类型`是按引用访问。
引用类型值保存在内存对象中，和其他语言不同，JS不允许直接访问内存中的位置。实际上是在操作对象的引用。

### 动态属性

可以为引用类型的值添加、删除、改变属性和方法。但是不能给基本类型的值添加属性，虽然不报错，但是是undefined
```js
var person = new Object();
person.name = 'Jack';
alert(person.name);  // "Jack"

var name = 'Chan';
name.age = 27;
alert(name.age); // undefined
```
### 复制变量值

一个变量向另一个变量复制基本类型和引用类型存在不同。
基本数据类型赋值：会在变量对象上创建新值，然后把改值复制到新变量分配的位置上，赋值后互不影响。

基本数据类型赋值：赋值的副本是一个指针，指向存储在堆中的一个对象，其中一个改变会影响到另外一个。
```js
var person1 = new Object();
var person2 = person1;
person1.name = 'Jack';

alert(person2.name);  // "Jack"
```

### 传递参数

ECMAScript中所有的函数`参数都是按值传递`。
```js
function addTen( num ) {
  num += 10;
  return num;
}

var count = 20;
var rs = addTen(count);
alert(count);  // 20, 无变化
alert(rs);  // 30
```

```js
function setName( obj ) {
  obj.name = 'Jack';
}

var person = new Object();
setName(person);
alert(person.name);  // "Jack"
```
因为obj 和 person引用的是同一个对象，即使obj这个对象是按值传递的，obj也会按引用来访问同一个对象。
所以内部改变会影响外部。因此错误的认为`参数是按引用传递`的。证明如下：

```js
function setName( obj ) {
  obj.name = 'Jack';
  obj = new Object();
  obj.name = "Chan";
}

var person = new Object();
setName(person);
alert(person.name);  // "Jack"
```
如果参数是按引用传递，那么person.name 的值应该是Chan
So,可以将ECMAScript函数的参数想象成局部变量。

### 检测类型

检测变量是不是基本数据类型，可以使用`typeof`操作符是最好的工具。如果变量是对象或者null, 会返回`object`
检测函数会返回`function`。

ECMAScript在检测引用类型的值用 `instanceof`操作符，基本类型不是对象，用`instanceof`检测会始终返回`false`
规定所有引用类型的值都是`Object`的实例，用instanceof检测始终返回`true`.

![typeof注意事项](/img/js-high-level-4.png)

## 执行环境及作用域

执行环境定义了变量或函数有权访问其他的数据。每一个执行环境都有一个与之关联的变量对象，
环境中定义的所有变量和函数都保存在这个对象中。我们编写的的代码无法访问这个对象，但解析器在处理数据时会在后台使用它。

### 延长作用域链

1、try-catch语句中的catch块
2、with语句

### 没有块级作用域

在js中，`if`语句中的变量声明会将变量添加到当前的执行环境中。

## 垃圾收集机制

js具有自动垃圾收集机制，垃圾收集器会按照固定的时间间隔（或在代码中执行预定的收集时间）周期性地执行这一操作。
通常有两种策略。

### 标记清除
JavaScript中最常用的垃圾收集方式就是`标记清除`（mark-and-sweep）
当变量进入环境（在函数中声明一个变量）时，就将这个 这个变量标记为"进入环境",离开执行环境，则将其标记为"离开环境"。

到2008年为止，IE,Firfox, Opera, Chrome和Safari的JavaScript实现使用的标记清除式的垃圾收集策略，
只不过垃圾收集时间间隔互有不同而已。

### 引用计数器
追踪记录每个值被引用的次数，
有个BUG就是循环引用，
```js
function problem() {
  var oA = new Object();
  var oB = new Object();

  oA.someObj = oB;
  oB.anothObj = oA;
}
```

对于不用变量设置为null，解除变量引用，下次垃圾收集器就会自动清掉。

### 性能问题
在IE中，调用window.CollectGarbage()方法会立即执行垃圾收集
Opera 7及以上高版本中，调用window.opera.collect()也会启动垃圾收集例程。

## 参考文档

- [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)

-----------------------

> 文章若有纰漏，请大家补充指正，谢谢！
> 如有疑问可留言，我会在第一时间回复。
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
