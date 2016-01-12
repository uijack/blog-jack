title: "AngularJS Services详解"
date: 2016-01-12 19:35:06
description: "总结AngularJS Services的使用和注意事项"
tags:
- "angular"
---

## Services

angular service是可替换的,通过`依赖注入(DI)`和对象连接在一起.在你应用中可以使用services组织和分享代码.

### service用途

1. 懒加载实例化: 当应用程序依赖它的时候,Angular才实例化.
2. 单例: 每个组件依赖的服务由service factory实例一个单例.

angular提供了很多有用的service(如:$http).

## 如何使用service

如果你的组件(controller,service,filter,directive)依赖service,则通过注入,剩下的实例交给angular的依赖注入子系统完成.例子如下:

```js
angular.module('myServiceModule', [])
.factory('notify', ['$window', function(win) {
  var msgs = [];
  return function(msg) {
   msgs.push(msg);
   if (msgs.length == 3) {
     win.alert(msgs.join("\n"));
     msgs = [];
   }
  };
}])
.controller('MyController', ['$scope','notify', function ($scope, notify) {
  // 注意function函数的参数和前面数组的参数是一一对应的,命名推荐和前面对应的名字一样.不一样也可以.
  $scope.callNotify = function(msg) {
    notify(msg);
  };
 }]);
```

> 先注册service,filter,directive,在写controller.

```plain
<div id="simple" ng-controller="MyController">
  <p>Let's try this simple notify service, injected into the controller...</p>
  <input ng-init="message='test'" ng-model="message" >
  <button ng-click="callNotify(message);">NOTIFY</button>
  <p>(you have to click 3 times to see an alert)</p>
</div>
```

## 如何创建services

### 注册services

servies一般通过module factory API注册.

```js
var myModule = angular.module('myModule', []);
myModule.factory('serviceId', function() {
  var shinyNewServiceInstance;
  // factory function body that constructs shinyNewServiceInstance
  return shinyNewServiceInstance;
});
```
注意,此时你并没有注册一个service实例,而是一个`factory方法`.当被访问时才会被创建.

### 依赖

Services也可以有自己的依赖,就像controller申明一样,在service的factory方法声明.如下:

```js
var batchModule = angular.module('batchModule', []);

/**
 * The `batchLog` service allows for messages to be queued in memory and flushed
 * to the console.log every 50 seconds.
 *
 * @param {*} message Message to be logged.
 */
batchModule.factory('batchLog', ['$interval', '$log', function($interval, $log) {
  var messageQueue = [];

  function log() {
    if (messageQueue.length) {
      $log.log('batchLog messages: ', messageQueue);
      messageQueue = [];
    }
  }

  // start periodic checking
  $interval(log, 50000);

  return function(message) {
    messageQueue.push(message);
  }
}]);

/**
 * `routeTemplateMonitor` monitors(监视) each `$route` change and logs the
 * current template via(通过) the `batchLog` service.
 */
batchModule.factory('routeTemplateMonitor', ['$route', 'batchLog', '$rootScope',
  function($route, batchLog, $rootScope) {
    $rootScope.$on('$routeChangeSuccess', function() {
      batchLog($route.current ? $route.current.template : null);
    });
  }]);
```

### 通过$provide注册一个Service

```js
angular.module('myModule', [])
.config(['$provide', function($provide) {
  $provide.factory('serviceId', function() {
    var shinyNewServiceInstance;
    // factory function body that constructs shinyNewServiceInstance
    return shinyNewServiceInstance;
  });
}]);
```



























