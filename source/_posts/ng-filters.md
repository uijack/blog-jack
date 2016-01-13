title: "AngularJS Filters详解"
date: 2016-01-13 21:01:44
description: "总结AngularJS Filters的使用和注意事项"
tags:
- "angular"
---

过滤器用来格式化表达式中的值,经常用在视图模板(templates)、控制器(controllers)或者服务(services)中.
Filter API查看[$filterProvider](https://code.angularjs.org/1.4.8/docs/api/ng/provider/$filterProvider).

## 在模板中使用filters

1. 使用格式(syntax);

```plain
// 单个filter
{{ expression | filter }}

// 链式filter
{{ expression | filter1 | filter2 | ... }}

// filter拥有(多个)参数
{{ expression | filter:argument1:argument2:... }}
```

## 在controllers, services,directives中使用filters

```js
angular.module('FilterInControllerModule', []).
controller('FilterController', ['filterFilter', function(filterFilter) {
  this.array = [
    {name: 'Tobias'},
    {name: 'Jeff'},
    {name: 'Brian'},
    {name: 'Igor'},
    {name: 'James'},
    {name: 'Brad'}
  ];
  this.filteredArray = filterFilter(this.array, 'a');
}]);
```

```plain
<div ng-controller="FilterController as ctrl">
  <div>
    All entries:
    <span ng-repeat="entry in ctrl.array">{{entry.name}} </span>
  </div>
  <div>
    Entries that contain an "a":
    <span ng-repeat="entry in ctrl.filteredArray">{{entry.name}} </span>
  </div>
</div>

结果: 
All entries: Tobias Jeff Brian Igor James Brad
Entries that contain an "a": Tobias Brian James Brad
```

## 用户自定义filters

创建自定义过滤器的过程很简单:仅仅需要在模块中注册一个新的过滤器工厂方法.过滤器中的参数都会作为附加参数传递给它.
Filters推荐命名方法: myappSubsectionFilterx 或者 myapp_subsection_filterx

下面的示例过滤器反转显示一个字符串。另外，它可以根据参数把字母全部大写:

```js
angular.module('myReverseFilterApp', [])
.filter('reverse', function() {
  return function(input, uppercase) {
    input = input || '';
    var out = "";
    for (var i = 0; i < input.length; i++) {
      out = input.charAt(i) + out;
    }
    // conditional based on optional argument
    if (uppercase) {
      out = out.toUpperCase();
    }
    return out;
  };
})
.controller('MyController', ['$scope', function($scope) {
  $scope.greeting = 'hello';
}]);
```

```plain
<div ng-controller="MyController">
  <input ng-model="greeting" type="text"><br>
  No filter: {{greeting}}<br>
  Reverse: {{greeting|reverse}}<br>
  Reverse + uppercase: {{greeting|reverse:true}}<br>
</div>

结果: 
No filter: hello
Reverse: olleh
Reverse + uppercase: OLLEH
```

## Stateful(动态) filters

如果你需要动态的filter,那么你必须设置`$stateful`,那么他将会呗执行一次或多次,在每一个`$digest`周期内.

```js
angular.module('myStatefulFilterApp', [])
.filter('decorate', ['decoration', function(decoration) {

  function decorateFilter(input) {
    return decoration.symbol + input + decoration.symbol;
  }
  decorateFilter.$stateful = true; // 注意此处$stateful

  return decorateFilter;
}])
.controller('MyController', ['$scope', 'decoration', function($scope, decoration) {
  $scope.greeting = 'hello';
  $scope.decoration = decoration;
}])
.value('decoration', {symbol: '*'});
```

```plain
<div ng-controller="MyController">
  Input: <input ng-model="greeting" type="text"><br>
  Decoration: <input ng-model="decoration.symbol" type="text"><br>
  No filter: {{greeting}}<br>
  Decorated: {{greeting | decorate}}<br>
</div>

结果:
Input: hello

Decoration: *

No filter: hello
Decorated: *hello*
```

当Input和Decoration变化的时候,就会触发filter,格式化结果.

## $filterProvider

$filter: provider in module ng

过滤器的功能只是将将输入变换后给输出.但是过滤器需要依赖注入.要实现用户自定义过滤器,需要包括一个工厂方法.

```js
// Filter registration
function MyModule($provide, $filterProvider) {
  // create a service to demonstrate injection (not always needed)
  $provide.value('greet', function(name){
    return 'Hello ' + name + '!';
  });

  // register a filter factory which uses the
  // greet service to demonstrate DI.
  $filterProvider.register('greet', function(greet){
    // return the filter function which uses the greet service
    // to generate salutation
    return function(text) {
      // filters need to be forgiving so check input validity
      return text && greet(text) || text;
    };
  });
}
```

详情参考register(name, factory) method.
