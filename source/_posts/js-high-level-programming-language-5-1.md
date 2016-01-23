title: "读《JavaScript高级程序设计》总结5-1"
date: 2015-12-23 20:52:11
description: "读《JavaScript高级程序设计 -- 第5章 引用类型》总结, 详细介绍Object、Array"
tags:
- "JavaScript"
---
## 前言

1、`引用类型的值`是`引用类型`的一个实例.
2、不具备传统面向对象语言所支持的类和接口等基本结构。
3、引用类型有时候也被称为`对象定义`，描述的是一类对象所具有的属性和方法。

## Object 类型

目前大多数引用类型值都是`Object`类型的实例。
创建Object实例的两种方法：

- 使用new操作符后跟Object构造函数，var p = new Object();
- 使用`对象字面量`表示法，简写形式。var p = { age: 29, name: 'Jack'};
> 最后一个属性后面不能添加逗号`,`，在IE7及以下版本和Opear会导致错误。
> 当数字作为对象属性时，会自动转化成字符串。

![参数命名规则](/img/js-high-level-5-1.png)

访问对象属性：
1、点表示法：person.name  // 推荐使用
2、方括号法：person['name'], person['your name'] //如果属性名包含会导致语法错误，或者关键字和保留字，使用此法。

## Array 类型

> 同一个数组可以保存字符串、数值、对象、布尔值等。
> 数组大小可以动态调整

创建Array两种方法：

- 使用构造函数：
```js
var colors = new Array();
var colors = new Array(20);
var colors = new Array('red', 'green', 'blue');
var colors = Array(5);
var colors = Array('greg');
```
- 数组字面量创建：
```js
var colors = ['red', 'blue', 'green'];
var names = [];
var values = [1, 2, ] ;      // 不推荐，IE8之前，会创建一个包含3项的数组
var values = [, , , , ,] ;   // 不推荐，IE8之前，会创建一个包含6项的数组
```

读取和设置数组值：
- 采用方括号[索引]

数组最多可以包含4 294 967 295 个项。达到上限会发生异常。

### 检测数组

- 单一的全局执行环境，使用 `instanceof操作符` if (v instanceof Array) { //执行操作 }、
- ECMAScript5新增Array.isArray() 方法。这个方法的目的是确定value是不是数组，而不管它是那个全局环境创建
- if (Array.isArray( value )) { // 执行操作 } 支持浏览器：IE9+,Ff4+, Saf5+, Op10.5+ Chrome

### 转换方法 join(),toString(),toLocaleString(),valueOf()

所有对象都具有 `toLocaleString()` `toString()` `valueOf()`方法，

`toString()` 会返回每一项的字符串形式与`逗号（,）`拼接成的字符串,
`valueOf()` 返回的还是数组字符串, 
`toLocaleString()`调用每一项的toLocaleString(),而不是toString().

```js
var p1 = {
  toLocaleString: function() {
    return 'Jack';
  },
  toString: function() {
    return: 'Jack';
  }
};

var p2 = {
  toLocaleString: function() {
    return 'Chan';
  },
  toString: function() {
    return: 'CHAN';
  }
};

var p = [p1, p2];
alert(p);  // Jack,CHAN
alert(p.toString());        // Jack,CHAN
alert(p.toLocaleString());  // Jack,Chan
```
> 可以使用join() 方法传递分隔符
```js
var colors = ['red', 'blue', 'yellow'];
alert(colors.join(','));       //red,blue,yellow
alert(colors.join('||'));      //red||blue||yellow
```
> 如果不给join()方法传值,或者传入`undefined`,则默认使用逗号分隔符.
> 如果数组中的某一项是`null`和`undefined`,该值在使用`join()` `toLocaleString()` `toString()` `valueOf()`方法返回的结果以`空字符串`表示.

### 栈方法 推入push(), 弹出pop()

栈: LIFO 后进先出
- push() 推入数组尾部 返回数组长度
- pop() 取得最后一项,并删除数组最后一项

```js
var col = [];
alert( col.push('a') ); // 1
alert( col.pop() ); // 'a'
alert( col.lenght ); // 0
```

### 队列方法 推入push(), 移除首相shift(), unshift()

队列: FIFO 先进先出
- push()   推入数组尾部 返回数组长度.
- shift()   移除数组中的第一项并返回该项, 同时将数组长度减1.
- unshift() 他能在数组前端添加任意个项并返回新数组长度.

> 使用unshift() 和 pop() 可以从相反方向模拟队列,即在数组前端添加,数组末端移除.
> IE7及以前调用unshify()总返回`undefined`而不是数组新长度,IE8在非兼容模式下返回正确新长度.

### 重新排序 reverse() 和 sort()

- sort() 按升序,小到大,调用每项的toString()转型方法,得到字符串在排序,即使是`数值`也一样.
- reverse() 按降序, 大到小.
> sort()接收一个比较函数(有两个参数arg1 和 arg2)作为参数, 以便我们指定那个值位于前面

比较函数(arg1, arg2)参数规则:
1、arg1 应该在 arg2 之前 则返回一个负数(一般为-1);
2、相等返回0;
3、arg1 应该位于 arg2 之后,则返回一个正数(一般为1);

> 对于数值类型或者valueOf()方法会返回数值类型的对象类型.可以使用更简单的比较函数

```js
function compare(value1, value2) {
  return value1 - value2;  //降序,如果想要升序则 return value2-value1;
}
```

### 操作方法 concat() slice() splice()

- concat() 基于当前数组创建新的数组,将接收到的参数(单项或者数组(将被一项一项拆分添加))添加到这个新数组,最后返回新的数组.
```js
var v1 = ['a'];
var v2 = v1.concat('b', ['c', 'd']);
console.log(v2);  // ['a', 'b', 'c', 'd'];
```

- slice() 能够基于当前数组中的一或多个项创建一个新的数组.
1、接收一个参数,返回该参数指定位置开始到数组末尾的所有项
2、两个参数, 返回之间的数据项,但不包括结束位置项
3、如果有一个参数是负数, 则用数组长度加上该负数确定位置, 如果结束为止小于起始位置,返回`空数组`.
```js
var v1 = ['a', 'b', 'c', 'd'];
var v2 = v1.slice(2);  // ['c', 'd'] 
var v3 = v1.slice(1, 3);  // [b', 'c'];
```
- splice() 最强大,主要用途是向数组的中部插入项,始终返回数组(返回删除的项,没有删除返回空数组).三种使用规则:
1、`删除`: 删除任一数量的项,`只需指定2个参数`,要删除的第一项的位置和删除的项数,如splice(0, 2)删除数组中的前两项.
2、`插入`: 可以向指定位置插入任意数量的项,`只需指定3个参数`,(起始位置、0(要删除的项数)、要插入的项数),如splice(2,0,'red','green');会从当前数组的位置2开始插入字符串'red'和'green'.
3、`替换`: 向指定位置插入任意数量项,且同事删除任意数量向项,`只需指定3个参数`,(起始位置、0(要删除的项数)、要插入的项数).

### 位置方法 indexOf() lastIndexOf()
ECMAScript 5 添加
- indexOf() 从头0开始查找
- lastIndexOf() 数组的尾开始查找
- 接受两个参数(查找项, 可选的查找起点位置的索引),如果第二个参数为负数,将转化为数组长度的整数倍加上改负数,变成最小的正数为止.

```js
var array = [2, 5, 9];
array.indexOf(2);     // 0
array.indexOf(7);     // -1
array.indexOf(9, 2);  // 2
array.indexOf(2, -1); // -1
array.indexOf(2, -3); // 0
```
```js
var array = [2, 5, 9, 2];
array.lastIndexOf(2);     // 3
array.lastIndexOf(7);     // -1
array.lastIndexOf(2, 3);  // 3
array.lastIndexOf(2, 2);  // 0
array.lastIndexOf(2, -2); // 0
array.lastIndexOf(2, -1); // 3
```
> 支持浏览器包括IE9+、Ff3+、Saf3+、Op9.5+、Chrome

### 迭代方法 every() filter() forEach() map() some()

![方法使用](/img/js-high-level-5-2.png)

- every()

```js
function isBigEnough(element, index, array) {
  return element >= 10;
}
[12, 5, 8, 130, 44].every(isBigEnough);   // false
[12, 54, 18, 130, 44].every(isBigEnough); // true
```

- filter()

```js
function isBigEnough(value) {
  return value >= 10;
}
var filtered = [12, 5, 8, 130, 44].filter(isBigEnough);
// filtered is [12, 130, 44]
```

- forEach()

```js
function logArrayElements(element, index, array) {
  console.log('a[' + index + '] = ' + element);
}

// Note elision, there is no member at 2 so it isn't visited
[2, 5, , 9].forEach(logArrayElements);
// logs:
// a[0] = 2
// a[1] = 5
// a[3] = 9
```

- map()

```js
var numbers = [1, 4, 9];
var roots = numbers.map(Math.sqrt);
// roots is now [1, 2, 3], numbers is still [1, 4, 9]

var kvArray = [{key:1, value:10}, {key:2, value:20}, {key:3, value: 30}];
var reformattedArray = kvArray.map(function(obj){ 
   var rObj = {};
   rObj[obj.key] = obj.value;
   return rObj;
});
// reformattedArray is now [{1:10}, {2:20}, {3:30}], 
// kvArray is still [{key:1, value:10}, {key:2, value:20}, {key:3, value: 30}]
```

- some()

```js
function isBiggerThan10(element, index, array) {
  return element > 10;
}
[2, 5, 8, 1, 4].some(isBiggerThan10);  // false
[12, 5, 8, 1, 4].some(isBiggerThan10); // true
```

### 缩小方法 reduce() reduceRight()

ECMAScript5新增,都会迭代每个数组项,然后构建一个最终返回值(求和)

```js
[0, 1, 2, 3, 4].reduce(function(previousValue, currentValue, currentIndex, array) {
  return previousValue + currentValue;
});  // 10
// 也可以去掉后面两个参数
```
![reduce call](/img/js-high-level-5-3.png)
> reduce和reduceRight支持浏览器包括IE9+、Ff3+、Saf4+、Op10.5、Chrome


> 未完待续....



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
