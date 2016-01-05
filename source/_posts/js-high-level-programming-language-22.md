title: "读《JavaScript高级程序设计》总结22"
date: 2016-01-05 07:26:35
description: "读《JavaScript高级程序设计 -- 第22章 高级技巧》介绍高级函数,防篡改对象,高级定时器,自定义事件和拖放"
tags:
- "JavaScript"
- "随笔"
---

## 高级函数

### 安全的类型检测

typeof操作符,有一些无法预知行为,直到Safari第4版在对正则表达式应用typeof操作符时会返回"function",因此很难确定某个值到底是不是函数.
instanceof操作符在存在多个全局作用域(像一个页面包含多个框架)的情况下,也是问题很多

### 作用域安全的构造函数

首先确认`this`对象是正确类型的实例,如果不是,那么创建新的实例返回.

以前:

```js
function Person(name, age, job){
  this.name = name;
  this.age = age;
  this.job = job;
}
        
var person1 = new Person("Nicholas", 29, "Software Engineer"); // 使用new操作符调用函数,this就会指向新创建的对象的实例

// 因为this对象是在运行时绑定.
alert(person1.name);     //"Nicholas"
alert(person1.age);      //29
alert(person1.job);      //"Software Engineer"

var person2 = Person("Nicholas", 29, "Software Engineer"); // 直接调用,此时的this是window
alert(person2);         //undefined
alert(window.name);     //"Nicholas"
alert(window.age);      //29
alert(window.job);      //"Software Engineer"
```

安全构造:
```js
function Person(name, age, job){
  if (this instanceof Person){
    this.name = name;
    this.age = age;
    this.job = job;
  } else {
    return new Person(name, age, job);
  }
}
        
var person1 = Person("Nicholas", 29, "Software Engineer");
alert(window.name);   //""
alert(person1.name);  //"Nicholas"

var person2 = new Person("Shelby", 34, "Ergonomist");
alert(person2.name);  //"Shelby"
```

可以避免全局对象上以外设置属性,但是`实现这个模式后,你就锁定了可以调用构造函数的环境`.
如果你使用构造函数窃取模式的继承且`不使用原型链`,那么这个继承很可能被破坏.

```js
function Polygon(sides){
  if (this instanceof Polygon) {
      this.sides = sides;
      this.getArea = function(){
          return 0;
      };
  } else {
      return new Polygon(sides);
  }
}

function Rectangle(width, height){
  Polygon.call(this, 2);
  this.width = width;
  this.height = height;
  this.getArea = function(){
      return this.width * this.height;
  };
}

var rect = new Rectangle(5, 10);
alert(rect.sides);   //undefined
```

因为此时Rectangle的this对象并非Polygon的实例,所以会返回一个新的Polygon对象,
Rectangle构造函数中的this对象并没有得到`增长`.同时Polygon.call()返回的值也没用到,
所以Rectangle实例就中不会有sides属性

如果构造函数窃取结合原型链或者寄生组合则可以解决这个问题.

```js
function Polygon(sides){
    if (this instanceof Polygon) {
        this.sides = sides;
        this.getArea = function(){
            return 0;
        };
    } else {
        return new Polygon(sides);
    }
}

function Rectangle(width, height){
    Polygon.call(this, 2);
    this.width = width;
    this.height = height;
    this.getArea = function(){
        return this.width * this.height;
    };
}

Rectangle.prototype = new Polygon(); // 使用原型链

var rect = new Rectangle(5, 10);
alert(rect.sides);   //2
```

此时,一个Rectangle实例也同时是一个Polygon实例,所以Polygon.call()会照原意执行,最终Rectangle添加了sides属性.

推荐使用

### 惰性载入函数(重要)

惰性载入表示函数执行的分支仅会发生一次,`2种实现惰性载入方式`.

- 第1种:在函数被调用时再处理函数.

> 在第一次调用的过程中,该函数会被覆盖为另外一个按合适方式执行的函数.这样任何对原函数的调用都不用在经过执行的分支了.
> 例如用惰性载入重写createXHR()

```js
function createXHR(){
  if (typeof XMLHttpRequest != "undefined"){
    createXHR = function(){ // 将原来的createXHR替换掉.相当于重写createXHR方法.
      return new XMLHttpRequest();
    };
  } else if (typeof ActiveXObject != "undefined"){
    createXHR = function(){                    
      if (typeof arguments.callee.activeXString != "string"){
        var versions = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0",
                        "MSXML2.XMLHttp"],
            i, len;

        for (i=0,len=versions.length; i < len; i++){
          try {
            new ActiveXObject(versions[i]);
            arguments.callee.activeXString = versions[i];
          } catch (ex){
              //skip
          }
        }
      }
  
      return new ActiveXObject(arguments.callee.activeXString);
    };
  } else {
      createXHR = function(){
        throw new Error("No XHR object available.");
      };
  }
  
  return createXHR();
}
        
var xhr1 = createXHR();
var xhr2 = createXHR();
```

在这个惰性载入的`createXHR()`中,if语句的每一个分支都会为`createXHR`变量赋值,有效的覆盖了原有的函数.
最后一步便是调用新的赋值的函数.下一次调用createXHR()的时候,就会直接调用被分配的函数.这样就不用再次执行if语句了.

- 第2种:声明函数时就指定适当的函数.

> 第二种实现惰性载入的方式是在声明函数时就指定适当的函数.这样第一次调用是就不会损失性能了,而在代码首次加载时会损失一点性能.

```js
var createXHR = (function(){
  if (typeof XMLHttpRequest != "undefined"){
    return function(){
        return new XMLHttpRequest();
    };
  } else if (typeof ActiveXObject != "undefined"){
    return function(){                    
      if (typeof arguments.callee.activeXString != "string"){
        var versions = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0",
                        "MSXML2.XMLHttp"],
            i, len;

        for (i=0,len=versions.length; i < len; i++){
          try {
              new ActiveXObject(versions[i]);
              arguments.callee.activeXString = versions[i];
              break;
          } catch (ex){
              //skip
          }
        }
      }
  
      return new ActiveXObject(arguments.callee.activeXString);
    };
  } else {
    return function(){
      throw new Error("No XHR object available.");
    };
  }
})();
        
var xhr1 = createXHR();
var xhr2 = createXHR();
```

这个例子中使用的技巧是创建一个匿名,自执行的函数.用已确认应该使用哪一种函数实现.

两种方式任选其一.看具体需求而定.不过两种方式都能避免执行不必要的代码.

### 函数绑定

`建议看PDF`:

一个简单的`bind()`函数接受`一个函数`和`一个环境`.并返回一个在给定环境中调用给定函数的函数,并且将所有参数原封不动传递过去.

```js
function bind(fn, context){
  return function(){
    return fn.apply(context, arguments);
  };
}
    
var handler = {
    message: "Event handled",

    handleClick: function(event){
        alert(this.message + ":" + event.type);
    }
};

var btn = document.getElementById("my-btn");
EventUtil.addHandler(btn, "click", bind(handler.handleClick, handler));
```
原生都已经能支持bind()函数.支持浏览器有IE9+,Firefox4+和Chrome

EventUtil.addHandler(btn, "click", handler.handleClick.bind(handler));

缺点: 有更多的开销,需要更多的内存,同时多重函数调用稍微慢一点,所以最好只在必要时使用.

### 函数柯里化

与`函数绑定`紧密相关的主题是函数柯里化(function currying), 他用于`创建`已经设置好了一个或多个参数的函数.
函数柯里化的基本方法和函数绑定是一样的,`使用一个闭包返回一个函数`.
`两者区别`在于,当函数被调用时,返回的函数还需要`设置一些传入的参数`.

```js
function add(num1, num2) {
  return num1 + num2;
}

function curriedAdd(num2) {
  return add(5, num2);
}

alert(add(2,3)); //5
alert(curriedAdd(3)); //8
```

柯里化函数通常由以下步骤动态创建: 调用另一个函数并为它传入要柯里化的函数和必要参数.

```js
function curry(fn){
  // 在arguments对象上调用slice()方法,并传入参数1表示要返回的数组包含从第二个参数开始返回所有参数
  // 然后args数组包含了来自外部函数的参数.
  // 这是第一次柯里化时传入的参数
  var args = Array.prototype.slice.call(arguments, 1);
  return function(){
    // 创建innerArgs数组用来存放所有传入的参数,有了存放来自函数外部和内部函数的参数数组后
    // 就可以调用concat()方法将他们组合成finalArgs.然后使用apply()将结果传递给该函数.
    // 这是真正调用函数再次传入的参数,然后将它们按顺序连接起来给fn函数使用.
    var innerArgs = Array.prototype.slice.call(arguments),
        finalArgs = args.concat(innerArgs);
        // 并没有考虑到执行环境,所以第一个参数为null
    return fn.apply(null, finalArgs);
  };
}
    
function add(num1, num2){
    return num1 + num2;
}

var curriedAdd = curry(add, 5);
alert(curriedAdd(3));   //8

var curriedAdd2 = curry(add, 5, 12);
alert(curriedAdd2());   //17
```

curry()工作就是将返回函数的参数进行排序.`第一个参数`是要进行`柯里化`的函数.,其他参数是要传入的值.

`还有更复杂的`bind()结合curry().

## 防篡改对象

利用第6章对象属性的问题.手工设置每个属性的[[Configurable]],[[Writale]],[[Enumerable]],[[Value]],[[Get]],[[Set]]特性
注意:一旦将对象定义为防篡改,就无法撤消了.

### 不可扩展对象

`可扩展对象`随时都可以向对象中添加属性和方法.

var peoson = {name: 'Jack'}

调用`Object.preventExtensions(peoson)`;方法

person.age =29;

alert(person.age); // undefined;

非严格模式下会静静的失败.严格模式下尝试给不可扩展的对象添加新成员会导致抛出错误.
虽然`不可添加`,但是已有的成员丝毫不受影响,`可以修改和删除以后成员`.

使用`Object.isExtensible(person)`可以检测对象是否可以扩展.返回布尔值.

### 密封对象

ECMAScript5为对象定义的第二个保护级别是`密封对象(sealed object`.密封对象不可扩展,
而已有成员的[[Configrable]]特性将被设置为`false`.这就意味着不能删除属性和方法.,因为不能使用Object.definedProperty()把数据属性修改为访问器属性.但是`属性值是可以修改的`.

```js
var person = { name: "Nicholas" };
Object.seal(person);

person.age = 29;
alert(person.age);      //undefined

delete person.name;
alert(person.name);     //"Nicholas"
```

在非严格模式下,尝试添加或删除没影响,严格模式下会抛错.使用`Object.isSealed(person)` 可以检测被密封对象不可扩展.

```js
var person = { name: "Nicholas" };
alert(Object.isExtensible(person));   //true
alert(Object.isSealed(person));       //false

Object.seal(person);
alert(Object.isExtensible(person));   //false
alert(Object.isSealed(person));       //true

person.age = 29;
alert(person.age);
```

### 冻结对象

最严格防篡改对象就是`冻结对象(forzen object)`.既不可扩展,又是密封的.而且对象的[[Writable]]特性会被设置成false.
`如果定义了[[Set]]函数,访问器属性仍然是可写的`.`Object.isFrozen()`检测冻结对象.

```js
var person = { name: "Nicholas" };
alert(Object.isExtensible(person));   //true
alert(Object.isSealed(person));       //false
alert(Object.isFrozen(person));       //false

Object.freeze(person);
alert(Object.isExtensible(person));   //false
alert(Object.isSealed(person));       //true
alert(Object.isFrozen(person));       //true

person.age = 29;
alert(person.age);
```

## 高级定时器

详细看PDF

### 重复的定时器(必看PDF)

- setInterval().
- setTimeout() 推荐使用.

### Yielding Processes

### 函数节流

## 自定义事件

## 拖放

拖放的概念很简单: 创建一个绝对定位的元素,使其可以用鼠标移动,这个技术源自一种叫做`鼠标拖尾`的经典网页技巧.
鼠标拖尾是一个或者多个图片在页面上跟着鼠标指针移动,单元鼠标拖尾的基本代码需要为文档设置一个`onmousemove`事件处理程序.
它总是将指定元素移动到鼠标指针的位置.

### 修缮拖动功能

### 添加自定义事件

- new EventTarget().



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
