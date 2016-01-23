title: "AngularJS 依赖注入详解"
date: 2016-01-13 19:47:26
description: "总结AngularJS Dependency Injection使用和注意事项"
tags:
- "angular1.x"
---

依赖注入(DI)是一种让代码管理其依赖关系的设计模式。

## 使用依赖注入

DI在angular是随处可见的,定义组件的时候可以使用,或当为一个模块提供运行(run)和配置(config)的时候,

1. 组件: services, directives, filters和被一个可注射工厂方法或构造函数的动画.这些组件可以被注入`service`,`value`组件作为依赖关系.
2. 由构造函数定义的Controller,可以被注可以入`service`,`value`的组件依赖.
3. `run`方法接受一个函数.可以被可以注入`service`,`value`,`constant`的组件依赖
4. `config`方法接受一个函数,可以被可以注入`provider`,`constant`的组件依赖.注意你不能注入`service`或`value`组件到配置里面.

可以看[Modules更多的关于run和config介绍](https://code.angularjs.org/1.4.8/docs/guide/module#module-loading-dependencies)

### 工厂(factory)方法

通过工厂方法你可以定义directive, service, filter.工厂方法被注册在模块(modules)上.使用如下:

```js
angular.module('myModule', [])
.factory('serviceId', ['depService', function(depService) {
  // ...
}])
.directive('directiveName', ['depService', function(depService) {
  // ...
}])
.filter('filterName', ['depService', function(depService) {
  // ...
}])
```

### 模块(module)方法

我们可以通过调用配置在module上的run和config的函数来运行.这些函数和依赖一样被注入.

```js
angular.module('myModule', [])
.config(['depProvider', function(depProvider) {
  // ...
}])
.run(['depService', function(depService) {
  // ...
}]);
```

### Controllers

可以使用controller数组形式注入

```js
someModule.controller('MyController', ['$scope', 'dep1', 'dep2', function($scope, dep1, dep2) {
  ...
  $scope.aMethod = function() {
    ...
  }
  ...
}]);
```

$scope,dep1,dep2将被注册到controller中使用.

1. `$scope`: 控制器和DOM元素交互,其他组件(service)只能通过$rootScope服务访问.
2. `resolves`: 待理解

### 依赖注解(annotation)

angular调用某个方功能(service factories和controllers)通过注射器,你需要注解这个方法,让注射器中知道什么服务被入射到这个方法.有2种方式在你的代码里面可以注注解.

1. 使用内联数组注释(推荐),上面Controllers的形式

```js
someModule.controller('MyController', ['$scope', 'greeter', function($scope, greeter) {
  // ...
}]);
```

2. 使用`$inject`属性注解

```js
var MyController = function($scope, greeter) {
  // ...
}
MyController.$inject = ['$scope', 'greeter'];
someModule.controller('MyController', MyController);
```

在这种场合下，$inject 数组中的服务名顺序必须和函数参数名顺序一致.

3. 隐式注解
```js
someModule.controller('MyController', function($scope, greeter) {
  // ...
});
```

以上在运行的生活需要之用gulp或者grunt第三方工具(ng-annotate),帮助转化成第一种.

## 使用严格的依赖注入

```plain
<!doctype html>
<html ng-app="myApp" ng-strict-di>
<body>
  I can add: {{ 1 + 2 }}.
  <script src="angular.js"></script>
</body>
</html>
```
在ng-app后面添加`ng-strict-di`,当service试图使用隐式注解时，严格模式会引发一个错误.

```plain
angular.module('myApp', [])
.factory('willBreak', function($rootScope) {
  // $rootScope is implicitly injected
})
.run(['willBreak', function(willBreak) {
  // Angular will throw when this runs
}]);
```

当willBreak service被实例化的时候,angular会抛出错误因为在严格模式下,使用`ng-annotate`工具可以确保所有应用程序都有注解.

如果你是自己启动angular,你也可以使用strict 依赖注入通过提供参数配置:

```js
angular.bootstrap(document, ['myApp'], {
  strictDi: true
});
```
