title: "读《JavaScript高级程序设计》总结5-2"
date: 2015-12-24 9:35:56
description: "读《JavaScript高级程序设计 -- 第5章 引用类型》总结, 详细介绍Date、RegExp、Function、window、基本包装类型、单体内置对象等"
tags:
- "JavaScript"
---

## Date 类型

Date类型使用自UTC(Coordinated Universial Time, 国际协调时间)1970年1月1日午夜(0时)开始经过的毫秒数来保存
Date类型保存的日期能够精确到1970年1月1日之前或之后285 616年.

```js
var now = new Date();       // 创建当前时间和日期
```
ECMAScript5新增Date.now()方法.返回日期毫秒数,可以用于分析代码运行时间.

```js
var start = Date.now(); // 不支持它的浏览器中使用+操作符把Date对象转化成字符串,可以达到同样目的
//  dosomething();      // var start = +Date.now();
var stop = Date.now();  // var stop  = +Date.now();

var rs = stop - start;
```
### 继承方法

Date重写了toString(),toLocaleString(),valueOf()方法
toString() 和 toLocaleString() 差别仅在调试代码时比较有用
valueOf()方法不返回字符串,而是`返回日期的毫秒数`.可用于比较日期大小
```js
var da1 = new Date(2007, 0, 1);
var da2 = new Date(2007, 1, 1);
da1 < da2 // true    比较毫秒数;
da1 > da2 // false 
```

### 日期格式化和方法

![日期格式化](/img/js-high-level-5-4.png)

### 日期和时间组件方法
- getTime()  返回日期毫秒数
- getFullYear() 得到四位数年份 2007
- getMonth()  返回日期月份 0~11 11表示12月份
- getDate()   返回日期的天数 1~31
- getDay()    返回日期属于星期几 0~6 0表示星期日
- getHours()  返回日期小时数 0~23
- getMinutes()  返回日期分钟数 0~59
- getSeconds()  返回日期毫秒数 0~59
- getMilliseeconds() 返回日期中的毫秒数

## RegExp 类型

var expression = / pattern / flags;

> pattern 简单或复杂正则表达式,包含字符类型,限定符,分组,向前查找或者反向引用
> 每个正则可带一个或多个标志(flags), 标明正则表达式的行为

flags支持三种:
- g 表示全局模式,用于所有字符串,而非第一个匹配时立即结束
- i 表示不区分大小写,忽略大小写情况
- m 表示多行模式,一行文本结束继续查找下一行

```js
var re = /\w+/;  // 表达式字面量
var re = new RegExp('\\w+');  // 构造函数创建
```

```js
var re = /(\w+)\s(\w+)/;
var str = 'John Smith';
var newstr = str.replace(re, '$2, $1');
console.log(newstr); // "Smith John"
```
> 正则表达式字面量始终会共享同一个RegExp实例
> 使用构造函数创建的每一个新RegExp实例都是新的实例

```js
var re = null,
    i, j;
for (i = 0; i < 10; i++) {
  re = /cat/g;
  re.test('catastrophe');
}

for (j = 0; j < 10; j++) {
  re = /cat/g;
  re.test('catastrophe');
}

```

### RegExp 实例属性

![RegExp 属性](/img/js-high-level-5-5.png)

两种模式的source属性是相同的.

### RegExp 实例方法

- exec() (index, input)
- test()
- valueOf() // 返回正则表达式本身

### RegExp 构造函数属性

省略...

### 模式的局限性

省略...

## Functon 类型

每一个函数都是Function类型的实例, 函数名实际实际上也是一个指向函数对象的指针.
声明方式:
```js
function sum(num1, num2) {
  return num1 + num2;
}

var sum = function(num1, num2) {
  return num1 + num2;
}

var sum1 = new Functon('num1', 'num2', "return num1 + num2"); // 不推荐,
// 会导致两次解析,第一次是解析常规ECMAScript代码,第二次是解析传入构造函数中的字符串.
```
### 函数声明与函数表达式

> 解析器在向执行环境中加载数据时,解析器会率先读取函数声明,并使其在执行任何代码之前可用(可用访问);
> 至于函数表达式,则必须等到解析器执行到它所在的代码行才会真正被解释执行.

```js
/* 函数声明 末尾不用加分号 */
alert(sum(10, 10));  // 20
function sum( num1, num2 ) {
  return num1 + num2;
}
```
```js
/* 函数表达式 末尾最好加分号 */
alert(sum(10, 10));  // 意外标识符错误
var sum = function( num1, num2 ) {
  return num1 + num2;
};
```
var sum = function sum(){} 其他可以,但是Saf会导致错误.

### 作为值的函数

函数名本身就是变量,函数也可以当做参数来使用.`也可以从函数中返回一个函数`,这是一种极为有用的技术.

有一个对象数组,要根据某个对象属性对数组进行排序

```js
function createComparisonFunction( propertyName ) {
  return fucntion( obj1, obj2 ) {
    var v1 = obj1[propertyName];
    var v2 = obj2[propertyName];

    if ( v1 < v2 ) {
      return -1;
    } else if ( v1 > v2 ) {
      return 1;
    } else {
      return 0;
    }
  };
}

var data = [{name: 'Zachary', age: 28}, {name: 'Nicholas', age: 29}];
data.sort(createComparisonFunction('name'));
alert(data[0].name);  // Nicholas

data.sort(createComparisonFunction('age'));
alert(data[0].name);  // Zachary
```

### 函数内部属性

函数内部有两个特殊属性: `this`和`arguments`.

- arguments 主要用途是保存函数数组,但是这个对象还有一个叫callee的属性,是一个指针,
- 指向拥有这个arguments对象的函数.

经典的阶乘:
```js
fucntion factorial( num ) {
  if (num <= 1) {
    return 1;
  } else {
    return num * factorial(num - 1);
  }
}
```

```js
fucntion factorial( num ) {
  if (num <= 1) {
    return 1;
  } else {
    return num * arguments.callee(num - 1);
  }
}
```
> 内部没引用函数名factorial,不管函数使用什么名字,都可以保证正常完成递归.
> 这种方式可以解除函数体的代码和函数名的耦合状态.

- this引用的是函数执行的环境对象, 当网页的全局作用域中调用函数时,this对象引用window

- caller ECMAScript5中的,这个属性保存着调用当前函数的函数的引用,在全局作用域中调用当前函数,caller的值为`null`

```js
function outer() { inner(); }
function inner() { alert(inner.caller);}
outer(); // 会显示outer()函数的源代码.
```
为了实现更散松的耦合,也可以通过改写

```js
function inner() { 
  alert(arguments.callee.caller);
}
```
IE,Ff,Chrome,Saf所有版本都支持,Opera9.6都支持`caller`属性
在函数`严格模式(use strict)`下访问`arguents.callee`会导致错误,
ECMAScript5还定义了arguments.caller属性,严格模式下访问也会出错.非严格模式下这个属性始终是`undefined`.严格模式不能为函数的caller属性赋值.但是可以访问`arguments`.`这很重要`.

### 属性和方法 length, property, apply,call,bind

函数也是对象,因此有属性和方法,每个函数都包含两个属性:
- length : 表示函数希望接收的命名参数的个数
- property : 保存所有实例方法.toString和valueOf等方法都保存在property名下.

```js
function sayName(name) {};
console.log(sayName,length); // 1
```
property属性是不可枚举的,因此for-in无法发现.

每个函数都包含两个非继承而来的方法`apply()`和`call()`;
这个两个方法的用途都是在特定的作用域中调用函数.
实际上等于设置函数体的`this`对象值.
- apply(运行函数作用域, [参数数组]), 接收两个参数,this,[];
- call(运行函数作用域, arg1, arg2,...);
- 可以不传递参数.只传递作用域.(没参数用谁都可以)

![注意事项](/img/js-high-level-5-6.png)

> 实际上,`apply()`和`call()`强大的地方是能够`扩充函数懒以运行的作用域`.`改变函数的执行环境`.
> 使用`apply()`和`call()`来扩充作用域的最大好处就是:`对象不需要与方法有任何耦合关系`.
> 即不需要将方法(函数)绑定在对象上,在通过对象来调用它.

ECMAScript5还定义了一个方法 `bind()`.这个方法会创建一个函数的实例,其`this`值会绑定到传给bind()
函数的值.
```js
window.color = 'red';
var o = { color: 'blue'};

function sayColor() {
  alert(this.color);
}

var objSayColor = sayColor.bind(o); // sayColor函数的this会绑定o对象上.
objSayColor(); // blue
```
> 这种技巧的优点请参考`第22章`

## 基本包装类

为了便于操作`基本类型值`,ECMAScript还提供了3个特殊的引用类型Boolean,Number,String.
`每当读取一个基本类型值的时候,后台就会创建一个对应的基本包装类型的对象`.从而让我们能够调用对应方法来操作这些数据.

![注意事项](/img/js-high-level-5-7.png)

### Boolean 对象

声明调用Boolean构造函数并传入true或flase

不推荐使用,
```js
var falseObj = new Boolean(false); // 此时falseObj是对象
var rs = falseObj1 && true; // 此时rs是true, 因为布尔表达式所有对象都会被转化为true

var falseValue = false;
var rs = falseValue && true; // rs为 false

alert(typeof falseObj); // object
alert(typeof falseValue); // boolean

alert(falseObj instanceof Boolean); // ture;
alert(falseValue instanceof Boolean); // false;
```

### Number 对象

var numberObj = new Number(10);

- toString()可以传递一个表示基数参数,告诉它返回几进制数值的字符串.

var num = 10;
alert(num.toFixed(2)); // "10.00" 按照指定位的小数位数返回数值的字符串 

- toExponential()
- toPrecision()

### String类型

var stringObj = new String('hello world');

- String类型的实例都包含一个length属性,表示字符串中有多少个字符.
- 即使字符串中包含`双字节字符`(不是占一个字节的ASCII字符),每个字符也仍算`一个字符`.

1. 字符方法, `charAt()`,`charCodeAt()`, 都接受一个基于0的字符位置. 
```js
var stringV = "hello world";
alert( stringV.charAt(1)); // 单个字符串"e"
alert( stringV.charCodeAt(1)); // 输出字符编码"101"
// 个别浏览器还支持alert(stringV[1]) "e"
```

2. 字符串操作方法. concat(),接收任意多个字符串,返回拼接的新字符串.实践中使用`+`操作符拼接.
- slice()
- substr()
- substring()

第一个参数:表示指定子字符串的开始位置.
第二个参数(在指定情况下),slice()和substring()第二个参数指定的是子字符串最后一个字符后面的位置.
而substr()指的是返回的字符个数.

- indexOf()
- lastIndexOf()
- trim() 创建字符串副本,删除前置和后缀所有空格,然后返回结果.部分浏览器支持trimLeft(),trimRight()

3. 字符串大小写转化
- toLowerCase()
- toUpperCase()

4. 字符串匹配(值得看)
- `match()` 参数要么是正则表达式,要么是RegExp对象.
- search()
- `replace()`
- `split()` 字符串分割.

## 单体内置对象

### Global 对象
isNaN(), parseInt(),
`encodeURL`, `decodeURL`,
`encodeURLComponet()`, `decodeURLComponet()`,

![URL编码](/img/js-high-level-5-7.png)

`eval()`, `window`

### Math 对象

> min(), max()
> Math.ceil() 向上取整
> Math.floor() 向下取整
> Math.round() 执行标准四舍五入
> random() 返回(0, 1)之间的数.
> selectFrom()

`5 完结`



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
