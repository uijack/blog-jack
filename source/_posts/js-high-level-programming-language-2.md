title: "读《JavaScript高级程序设计》总结2"
date: 2015-12-22 09:01:09
description: "读《JavaScript高级程序设计 -- 第3章 基本概念》总结, 本章主要介绍语法、变量、数据类型、操作符函数等"
tags:
- "js"
- "随笔"
---

## 语法

- 区分大小写
> `ECMAScript` 中的一切（变量、函数名和操作符）都区分大小写。

- 标示符
> 所谓`标识符`，就是指`变量`，`函数`，`属性`的名字，或者函数的参数。
  命名规则：
  1、第一个字符必须是一个字母、下划线（_）或者一个美元符（$）;
  2、其他字符可以是字母、数字、下划线或美元符。
  3、命名推荐采用`驼峰式`规则。

- 注释
> 单行注释 和 多行（块级）注释

```js
// 单行注释

/*
 *  这是一个多行
 *  （块级）注释
 *  企业级提高注释可读性
 */
```

- 严格模式
> ECMAScript 5 引入了严格模式，即在js文件首行或者函数内部第一行添加：

```js
'use strict';
```
这是一个编译指示（Pragram）,告诉支持的JS引擎切换到严格模式，
支持的浏览器版本：IE10+, Firefox4+, Safari5.1+,  Opera 12+ 和Chrome。

- 语句
> 推荐语句以一个分号`;`结尾, 可以 `减少代码压缩错误` 和 `降低解析器推测插入分号位置的时间`，
> 控制语句哪怕只有一行也要添加{}，降低修改代码出错率。

## 关键字和保留字

> 此处不做展示

## 变量

ECMAScript 变量是`松散类型`，可以用来保存任何类型数据，仅仅是一个`占位符`而已，
```js
var essage;  //该值为 undefined
```
> 省略var操作符声明的变量直接赋值，该变量则会变成`全局变量`

```js
  function test() {
    message = 'hi';  // 全局变量
  }

  test();
  alert(message);  // 'hi'
```

调用过一次test()函数后，这个变量就有了定义，就可以在函数外访问到。

## 数据类型

ECMAScript 中的五种简单数据类型（基本数据类型）
```plain
Undefined
Null
Boolean
Number
String
```
还有一种复杂数据类型`Object`,本质是由一组`无序的名值对`组成。

### typeof 操作符

用来检测给定变量的数据类型

- "undefined"  --  值未定义
- "boolean"    --  值是布尔型
- "string"     --  值是字符串
- "number"     --  值是数值
- "object"     --  值是对象或者`null`
- "funciton"     --  值是函数

> Test
```js
var ms = 'message';
alert(typeof ms) // 'string'
alert(typeof(ms)) // 'string'
alert(typeof 95) // 'number'
```

### Undefined 类型

> 只有唯一值 `undefined` ECMA-262第三版才引入，是为了正确区`空对象指针(null)`与`未初始化变量`。
> 不用显示声明 var ms = undefined; 默认就是`undefined`.

使用var 声明变量但对其初始化赋值，这个变量就是`undefined`

```js
var ms;
alert(ms == undefined); // true
```
注意：包含undefined值的变量与还未定义的变量不一样

```js
var ms;  // 
// age 未声明

alert(ms)   // 'undefined'
alert(age)  // 产生错误
```

令人困惑是的是，使用`typeof`操作符的时候，都会返回`undefined`.
```js
var ms;  // 
// age 未声明

alert(typeof ms)   // 'undefined'
alert(typeof age)  // 'undefined'
```
- 这个结果逻辑之上是合理的，因为实际上无论对哪一种变量也不可能执行真正的操作。

### Null 类型
> 只有唯一值 `null`，表示空对象指针

```js
var car = null;
alert(typeof car); // 'object'
```

如果定义变量准备用来保存对象，推荐初始化为null,而不是其他值。
只要直接检查null值就可以知道变量是否已经保存对象的引用

```js
var car = null;
if (car != null) {
  // do something
}
```
- 实际上，`undefined值是派生自null值的`，So,ECMA-262 规定对他们相等性测试要返回 `true`:
```js
alert(null == undefined); // true
```
### Boolean 类型

- 值只有`true` 和 `false`;

各种数据类型转化为boolean规则:

数据类型   |  转化为 true 值  |   转化为 false 值
--------- | -------------  | -------------
Boolean   |      true      |      false
String    |    任何非空字   |    ""(空字符串)
Number    |   任何非零数字   |     0 和 NaN
Object    |     任何对象    |       null
Undefined |       n/a      |    undefined

注：n/a (N/A) not application 缩写， 意思是"不适用"。

### Number 类型

- 采用IEEE754 格式表示整数和浮点数，
> 8进制第一位必须是零(0), 数字序列为0~7
> 字面值中的数值超过范围，那么前导零将被忽略，后面数值当做10进制解析。
> 八进制在严格模式下是无效的，会导致支持的JS引擎抛出错误。
> 16进制前两位必须是 0x, 数字序列为0~9 及A ~ F，字母支持大小写。
> 算术计算都将转化为10进制计算， +0和-0被认为相等。

Number.MIN_VALUE --> 5e-324
Number.MAX_VALUE --> 1.7976931348623157e+308
如果超过以上范围，自动转化为负无穷或者正无穷
Number.NEGATIVE_INFINITY = -Infinity;
Number.POSITIVE_INFINITY = Infinity;

`NaN`即非数值，是一个特殊的值。任何数除以0会返回NaN,涉及NaN的操作都会返回NaN.
NaN与任何数值都不想等，包括本身。alert(NaN == NaN); // 返回false;

`API: isNaN(任何类型参数)` 该函数会帮我们确定这个参数是否是“不是数值”。先转化成数值
```js
alert(isNaN(NaN))    // true
alert(isNaN(10))     // false 10是一个数值
alert(isNaN('10'))   // false 可以被转化成数值10
alert(isNaN('blue'))  // true 不能转化成数值
alert(isNaN(true))  // false 可以被转化成1
```

注：基于对象调用isNaN(),首先会调用对象valueOf(),不能转化则基于这个值在调用toString()在测试返回值。

数值转化函数： `Number()` `parseInt()` `parseFloat()` 转化规则详情官网。`很重要`

### String 类型

由零或多位16位Unicode字符串组成的字符序列，即字符串。

数值、布尔值、对象和字符串值都有toString()方法，但`null`和`undefined`值没有这个方法。
数值调用toString()方法默认以10进制格式返回数值的字符串表示。可以用过传递基数参数，转化为其他格式字符串

```js
var num = 10;
alert(num.toString())    // "10"
alert(num.toString(2))   // "1010"
alert(num.toString(8))   // "12"
alert(num.toString(10))  // "10"
alert(num.toString(16))  // "a"
```
String()转化，会先调用改参数的toString()并返回相应结果
如果值是null  String(null)  ,则返回"null"，
如果是undefined  String(undefined)  ,则返回"undefined"，

### Object 类型

var o = new Object();
var o = new Object;  //无参，不推荐

## 操作符

### 一元操作

- "++" 和 "--"
### 位操作

- NOT AND OR XOR << >> >>>
### 布尔值操作

- ! &&  ||
### 乘性操作

- * / % 
### 加性操作

- "+"  "-" 
### 关系操作

- "<" ">" "<=" ">="
### 相等操作

- "==" "===" "!=="
> 相等和不相等（强制转换） -- 先转化在比较
> 全等和不全等 -- 仅比较而不转换

在转换不同的数据类型时，相等和不相等操作遵循下列基本规则：
```markdown
[ ] 如果有一个操作数是布尔值，则在比较相等性之前先将其转化为数值。 false为0，true为1.
[ ] 如果一个操作数是字符串，另一个操作数是数值，在比较相等性之前先将字符串转化为数值。
[ ] 如果操作数是对象，另一个操作数不是，则调用对象的valueOf()方法，用得到的基本类型值按照前面规则进行比较。
null 和 undefined比较遵循下列原则：
[ ] null 和 undefined 是相等的。
[ ] 要比较相等性之前，不能将null和undefined转化成其他任何值。
[ ] 有一个操作数是NaN, 则相等操作符返回false,而不相等操作符返回true.重要提示：及时两个操作数都是NaN,也返回false，因为按照规则NaN不等于NaN.
[ ] 两个都是对象，则比较他们是不是同一个对象，两个操作数都指向同一个对象，则相等操作符返回true，否则返回false。
```

下列列出一些特殊的情况集结果：
```markdown

       表达式        |    值     
------------------- | --------  
null == undefined   |   true    
     "NaN" == NaN   |   false  
         5 == NaN   |   false  
       NaN == NaN   |   false  
       NaN != NaN   |   true 
       false == 0   |   true 
        true == 1   |   true 
        true == 2   |   false 
   undefined == 0   |   false 
        null == 0   |   false 
         "5" == 5   |   true
```
> 全等和全不等在比较之前不转换操作数

null === undefined ; // false,不同类型 


### 条件操作

- expression ? v1 : v2;
### 赋值操作

- "="  "*="  "/="  "%="  "+="  "-+"  "<<="  ">>="  ">>>=" 
### 逗号操作

- var num, num2, num3;

## 语句

### if 语句

- if else if else 推荐添加{}

### do-while 语句

- 后测试循环语句。先执行循环体一次后才会测试出口

### while 语句

- 前测试循环语句。

### for 语句

- 前测试循环语句。
使用while循环做不到的，使用for循环也做不到。
```js
for (var i = 0; i < 10; i++) {
  alert(i);
}
alert(i);  // 结果为 10；因为不存在块级作用域，循环内部定义的变量外部也可以访问

for (;;) {
  // 无限循环
}
```
### for in 语句

- 一般用来枚举对象属性 如window
> 如果要迭代的对象变量值为`null`或`undefined`, for-in 语句会抛出错误，ECMAScript 5 更正了，不抛出错误，
> 而只是不执行循环体，为了兼容性，在使用之前，先检测确认该对象的值不是`null`或`undefined`。

### label 语句
### break 和 continue 语句
### with 语句
- 严格模式下不允许使用with语句，
> with是将代码的作用域设置到一个特定的对象中。
```js
var qs = location.search.substring(1);
var hostName = location.hostname;
var url = location.url;

// 等价于

with(location) {
  var qs = search.substring(1);
  var hostName = hostname;
  var url = url;
}
```
### switch 语句

## 函数
- 对任何语言来说函数都是核心，使用`function`声明

### 理解参数

ECMAScript 函数不介意传递进来多少个参数, 也不在乎传进来什么数据类型,`即便定义的函数只接收两个参数，
在调用函数的也未必一定要传递两个参数，可以是一个、三个甚至不传递`。
`因为ECMAScript中的参数在内部是用一个数组来表示的，函数接收到的始终是一个数组，而不关心包含哪些参数`。
函数内部通过`arguments对象`来访问这个参数数组，从而获取传递给函数的每一个参数。
在`严格模式`下对如何使用arguments对象做出了限制，

### 没有重载
不存在函数签名（参数的类型和数量）的特性, ECMAScript函数不能重载。
- 未指定返回值的函数返回的是一个特殊的undefined值。

## 参考文档

- [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)

-----------------------

> 文章若有纰漏，请大家补充指正，谢谢！
> 如有疑问可留言，我会在第一时间回复。
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
