title: "AngularJS Controller详解"
date: 2016-01-12 07:08:06
description: "总结AngularJS Controller的使用和注意事项"
tags:
- "angular"
---
## 理解控制器(Controller)

1. ng-controller是一个`指令(directive)`.
2. 在Angular中,控制器就像JavaScript中的构造函数一般,用来增强`Angular作用域(scope)`的.
3. 控制器(Controller)通过`ng-controller`指令被添加到DOM中,ng会调用该控制器的构造函数来生成一个控制器对象,这样就创建了一个新的`子级作用域(scope)`,在构造函数中,作用域`scope`会作为`$scope`参数注入其中,并允许用户代码访问它.

## Controller的用途

### 初始化`$scope`对象

当我们创建应用程序时，我们通常要为Angular的$scope对象设置初始状态，这是通过在$scope对象上添加属性实现的。这些属性就是供在视图中展示用的视图模型（view model）它们在与此控制器相关的模板中均可以访问到.

> 虽然Angular允许我们在全局作用域下(window)定义控制器函数，但建议不要用这种方式。在一个实际的应用程序中，推荐在 Angular模块下通过`.controller`为你的应用创建控制器.

```js
var myApp = angular.module('myApp',[]);

myApp.controller('GreetingCtrl', ['$scope', function($scope) {
  $scope.greeting = 'Hola!';
}]);
```

```plain
<html ng-app="myApp">
<div ng-controller="GreetingCtrl">
{{greeting}}
</div>
</html>
```

### 为$scope添加行为

为了对事件作出响应，或是在视图中执行计算，我们需要为scope提供相关的行为操作的逻辑.这些方法就可以在模板/视图中被调用了.

```js
var myApp = angular.module('myApp',[]);

myApp.controller('DoubleController', ['$scope', function($scope) {
  $scope.double = function(value) { return value * 2; };
}]);
```

```plain
<div ng-controller="DoubleController">
  Two times <input ng-model="num"> equals {{ double(num) }}
</div>
```

## 正确使用Controller

通常情况下，控制器不应做很多处理，它只需负责一个单一视图所需的业务逻辑.

### 以下场景不要使用Controller

1. 操作DOM: 控制器应该只包含`业务逻辑`,DOM操作属于`应用程序的表现层逻辑`,把任何表现逻辑放到控制器中会大大`增加业务逻辑的测试难度`,需要手动操作DOM,最好将表现层的逻辑封装在`指令(directive)`中.(很重要)
2. 格式化输入: 使用angular form controls代替
3. 过滤输出: 使用angular filter代替
4. 在控制器间复用有状态或无状态的代码: 使用angular service代替
5. 管理其他部件的生命周期(如手动创建service实例)

> 由于`controller`存在着很多级,在HTML中controller一层一层嵌套,子级controller的`$scope`可以访问父级,如果子级有则只能访问自己的属性. 类似`prototype`一样.

## controller显式注入service

```js
// 第一种:通过数组形式注入(推荐推荐推荐)
myModule.controller('MyController', ['$location', function($location) { ... }]);

// 第二种:通过$inject
var MyController = function($location) { ... };
MyController.$inject = ['$location'];
myModule.controller('MyController', MyController);
```

简单的例子: [参考官网例子example](https://code.angularjs.org/1.4.8/docs/guide/controller)
