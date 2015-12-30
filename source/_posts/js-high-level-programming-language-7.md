title: "读《JavaScript高级程序设计》总结7"
date: 2015-12-27 17:18:07
description: "读《JavaScript高级程序设计 -- 第7章 函数表达式》总结,介绍递归, 闭包,模仿块级作用域,私有变量等"
tags:
- "js.闭包"
- "随笔"
---

## 函数表达式

函数定义方式有两种: 函数声明, 函数表达式

- 函数声明的重要特性就是函数声明提升,即在执行代码之前会先读取函数声明(区别于函数表达式).
- 函数表达式即创建一个匿名函数并赋值给变量.

## 递归

经典递归的改写:

```js
var factorial = (function f(num) {
  if (num <= 1) {
    return 1;
  } else {
    return num * f(num - 1);
  }
});
```

优点:可以避免严格模式下使用哪个arguments.callee代替函数名的不支持.防止factorial被篡改导致函数出错.
严格模式和非严格模式都行得通.
## 闭包

闭包是指: 有权访问另一个函数作用域中的变量的函数.创建闭包的常见方式,就是在一个函数内部创建另一个函数.

```js
function createComparisonFunction( propertyName ) {
  return fucntion( obj1, obj2 ) {
    var v1 = obj1[propertyName];  // 1
    var v2 = obj2[propertyName];  // 2

    if ( v1 < v2 ) {
      return -1;
    } else if ( v1 > v2 ) {
      return 1;
    } else {
      return 0;
    }
  };
}
```
1 和 2是内部函数(一个匿名函数)中的代码, 访问外部函数变量中的propertyName, 即使这个内部函数被返回了,而且是在其他地方被调用了,它仍可以访问变量`propertyName`.因为内部函数的作用域链中包含`pcreateComparisonFunction()`的作用域.

`作用域链`:当某个函数第一次被调用时, 会创建一个执行环境(execution context)及相应的作用域链,并把作用域链赋值给一个特殊的内部属性(即`[[Scope]]`). 然后,使用`this`, `arguments`和`其他命名参数的值`来初始化函数的活动对象(activation object),`函数外部的活动对象`始终处于第二位,外部函数的外部函数的活动对象处于第三位,......直到作用域链终点的全局执行环境.

```js
fucntion compare( value1, value2 ) {
  if ( value1 < value2 ) {
    return -1;
  } else if ( value1 > value2 ) {
    return 1;
  } else {
    return 0;
  }
}

var result = compare(5, 10);
```

以上代码在全局作用域中调用它.

![函数作用域图](/img/js-high-level-7-1.png)

由上图可知: `作用域链本质上是一个指向变量对象的指针列表,它只引入但不包含实际变量对象`.

### 闭包与变量

闭包只取得包含函数中任何变量的最后一个值.闭包保存的是整个变量对象,而不是某个特殊的变量.

```js
function createFunctions() {
  var result = new Array();
  for (var i=0;i < 10; i++) {
    result[i] = function() {
      return i;
    };
  }

  return resutl;
}
```

结果,result每个变量的值都是10. 可以通过匿名函数强制让闭包的行为符合预期.

```js
function createFunctions() {
  var result = new Array();
  for (var i=0;i < 10; i++) {
    result[i] = function(num) {
      return function() {
        return num;
      };
    }(i);
  }

  return resutl;
}
```

因为函数参数是按值传递的.

让闭包访问该对象,而不是外部对象.

```js
var name = 'window';

var object = {
  name: 'object',
  getName: funciton() {
    var that = this; // that指代object,很重要,能够避免this值的意外改变
    return function() {
      return that.name;
    }
  }
};

object.getName()(); // 'object'
```

几种特殊情况,this的值可能会意外改变.
```js
var name = 'window';

var object = {
  name: 'object',
  getName: funciton() {
    return this.name;
  }
};

object.getName(); // 'object'
(object.getName)(); // 'object'  等价于上面种
(object.getName = object.getName)(); // 'window' 非严格模式下
```

### 内存泄漏

合理使用闭包,删除使用完的的外部引用

## 模仿块级作用域
```js
function outp(count) {
  for (var i=0; i < count; i++) {
    console.log(i);
  }

  alert(i); // count的值
}
```

在Java,C++中, i只会定义在循环的语句块中,循环结束,变量就被销毁.但是在JS中, i是被定义在outp()的活动对象中的.

用块级作用域(私有作用域)的匿名函数
(function() {
  // 这里是块级作用域
})()

```js
function outp(count) {
  (fucntion() {
    for (var i=0; i < count; i++) {
      console.log(i);
    }
  })();
  alert(i); // 导致一个错误
}
```

## 私有变量

把有权限访问私有变量和私有函数的公有方法称为`特权方法(privileged method)`.
两种在对象上创建特权的方法:
1, 在构造函数中定义特权方法.
```js
function MyObj() {
  var a = 10;
  funciton aaa() {}

  // 特权方法
  this.publicMethod = function() {
    a++;
    return aaa();
  };
}
```

多查找作用域链中的一个层次,就会在一定程度上影响查找速度.这正是使用闭包和私有变量的一个明显的不足之处.

### 模块模式

模块模式(module pattern)则是为单例创建私有变量和特权方法. 所谓单例即只有一个实例的对象.

```js
var singleton = function() {
  // 私有变量和私有函数
  var privateVar = 10;

  function privateFun() {
    return false;
  }

  // 特权/公有方法和属性
  return {
    publicProperty: true,

    publicMethod: function() {
      privateVar++;
      return privateFun();
    }
  };
}();
```

WEB应用程序中,需要一个单例来管理应用程序级的信息.创建一个管理组件的application对象

### 增强的模块模式

```js
var singleton = function() {
  // 私有变量和私有函数
  var privateVar = 10;

  function privateFun() {
    return false;
  }

  // 创建对象

  var obj = new CustomerType();

  // 特权/公有方法和属性
  obj.publicProperty = true;
  obj.publicMethod = function() {
    privateVar++;
    return privateFun();
  };
  
  return obj;
}();
```
```js
var application = function() {
  // 私有变量核函数
  var components = new Array();

  // 初始化
  components.push(new BaseComponent());

  // 创建application的一个局部副本
  var app = new BaseComponent();

  // 公共接口
  app.getComponentCount = function() {
    return components.length;
  };

  app.registerComponent = function(component) {
    if (typeof component == 'object') {
      components.push(component);
    }
  };

  // 返回这个副本
  return app;
}();
```



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
