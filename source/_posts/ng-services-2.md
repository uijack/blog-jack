title: "AngularJS Services详解(2)"
date: 2016-01-19 07:10:14
description: "介绍AngularJS Services创建组件服务"
tags:
- "angular1.x"
---


## 创建服务组件

在AngularJS中创建一个服务组件很简单,只需要定义一个具有`$get`方法的构造函数,然后使用模块的provider方法进行登记.例如:下面是一个支持四则运算的服务.

```js
function doCalc(){
  var injector = angular.injector(["ezstuff"]),
    mycalculator = injector.get("ezCalculator"),
    ret = mycalculator.add(3,4);

  document.querySelector("#result").textContent = ret;
}

angular.module("ezstuff",[])
  .provider("ezCalculator",function(){
    this.$get = function(){
      return {
        add : function(a,b){return a+b;},
        subtract : function(a,b){return a-b;},
        multiply : function(a,b){return a*b;},
        divide: function(a,b){return a/b;}
      }
    };
  })
```

```plain
<body>
  <button onclick="doCalc();">3+4=?</button>
  <div id="result"></div>
</body>
```

## 可配置的服务

有时我们希望服务在不同的场景下可以有不同的行为，这意味着服务可以进行`配置`,AngularJS使用模块的`config()`方法对服务进行配置，需要将`实例化的服务提供者`(而不是服务实例)注入到`配置函数`中.

```js
angular.module("myModule",[])
.config(["myServiceProvider",function(myServiceProvider){
    //do some configuration.
}]);
```

> 服务提供者provider对象在注入器中的登记名称是:`服务名+Provider`.例如:`$http`的服务提供者实例名称是`$httpProvider`。

一下改写上一个四则运算:

```js
angular.module("ezstuff",[])
  .provider("ezCalculator",function(){
    var currency = "$";
    this.setLocal = function(l){
      var repo = {
        "CN":"¥",
        "US":"$",
        "JP":"¥",
        "EN":"€"
      };
      if(repo[l]) currency = repo[l];
    };
    this.$get = function(){
      return {
        add : function(a,b){return currency + (a+b);},
        subtract : function(a,b){return currency + (a-b);},
        multiply : function(a,b){return currency + (a*b);},
        divide: function(a,b){return currency + (a/b);}
      }
    };
  })
  // 上面注册了一个ezCalculator服务组件,使用config配置该服务的时候要使用:`服务名+Provider`.
  .config(function(ezCalculatorProvider){
    ezCalculatorProvider.setLocal("CN");
  });
```

## 服务定义语法糖(重要重要重要)

使用模块的provider方法定义服务组件,在有些场景下显得有些笨重.AngularJS友好地提供了一些简化的定义方法,这些方法`通常`只是对`provider`方法的封装,分别适用于不同的应用场景:

- factory

使用一个`对象工厂函数`定义服务,调用该工厂函数将返回服务实例。

- service

使用一个`类构造函数`定义服务,通过`new操作符`将创建服务实例。

- value

使用一个`值`定义服务,这个值就是服务实例.

- constant

使用一个`常量`定义服务，这个常量就是服务实例。

### factory()

factory方法要求提供一个对象工厂,调用该类工厂将返回服务实例.

```js
var myServiceFactory = function(){
    return ...
};
angular.module("myModule",[])
.factory("myService",myServiceFactory);
```

> INSIDE(在内部)：AngularJS会将factory方法封装为provider，上面的示例 等同于:

```js
var myServiceFactory = function(){
    return ...
};
angular.module("myModule",[])
.provider("myService",function(){
    this.$get = myServiceFactory;
});
```

改写的ezCalculator实例:

```js
angular.module("ezstuff",[])
.factory("ezCalculator",function(){
  return {
    add : function(a,b){return a+b;},
    subtract : function(a,b){return a-b;},
    multiply : function(a,b){return a*b;},
    divide: function(a,b){return a/b;}
  };
});
```

### service()

service方法要求提供一个`构造函数(this)`,AngularJS使用这个构造函数创建服务实例:

```js
var myServiceClass = function(){
    this.method1 = function(){...}
};
angular.module("myModule",[])
.service("myService",myServiceClass);
```

> INSIDE：AngularJS会将service方法封装为provider,上面的示例等同于:

```js
var myServiceClass = function(){
    //class definition.
};
angular.module("myModule",[])
.provider("myService",function(){
    this.$get = function(){
        return new myServiceClass();
    };
});
```
改写的ezCalculator实例:

```js
var ezCalculatorClass = function(){
  this.add = function(a,b){return a+b;};
  this.subtract = function(a,b){return a-b;};
  this.multiply = function(a,b){return a*b;};
  this.divide = function(a,b){return a/b;};
};

angular.module("ezstuff",[])
.service("ezCalculator",ezCalculatorClass);
```

### value()

有时我们需要在不同的组件之间共享一个变量,可以将这种情况视为一种服务:provider返回的总是变量的值。

```js
angular.module("myModule",[])
.value("myValueService","cx129800123");
```

> INSIDE：AngularJS会将value方法封装为provider,上面的示例等同于:

```js
angular.module("myModule",[])
.provider("myService",function(){
    this.$get = function(){
        return "cx129800123";
    };
});
```

### constant()

有时我们需要在不同的组件之间共享一个常量,可以将这种情况视为一种服务:provider返回的总是常量的值。constant方法提供了对这种情况的简化封装:

```js
angular.module("myModule",[])
.constant("myConstantService","Great Wall");
```

和`value()`不同,AngularJS并没有将`constant()`封装成一个provider，而仅仅 是在内部登记这个值。这使得常量在AngularJS的启动配置阶段就可以使用(创建任何 服务之前):你可以将常量注入到模块的`config()`中.

```js
angular.module("ezstuff",[])
  .constant("ezCurrency","CN")
  .provider("ezCalculator",function(){
    var currency = "$";
    this.setLocal = function(l){
      var repo = {
        "CN":"¥",
        "US":"$",
        "JP":"¥",
        "EN":"€"
      };
      if(repo[l]) currency = repo[l];
    };
    this.$get = function(){
      return {
        add : function(a,b){return currency + (a+b);},
        subtract : function(a,b){return currency + (a-b);},
        multiply : function(a,b){return currency + (a*b);},
        divide: function(a,b){return currency + (a/b);}
      }
    };
  })
  // 在config()中将常量ezCurrency注入到模块中
  .config(function(ezCurrency,ezCalculatorProvider){
    ezCalculatorProvider.setLocal(ezCurrency);
  });
```
