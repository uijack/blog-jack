title: "AngularJS Directives详解"
date: 2016-01-12 21:35:06
description: "总结AngularJS Directives的使用和注意事项"
tags:
- "angular"
---

## 创建自定义指令

此文解释了在你的AngularJS应用里，何时该创建自定义指令以及如何去实现它们.

### "指令(directive)"是什么?

简单点说，指令就是一些附加在HTML元素上的自定义标记（例如：属性，元素，或css类）,它告诉AngularJS的HTML编译器 ($compile) 在元素上附加某些指定的行为，甚至操作DOM、改变DOM元素，以及它的各级子节点.

Angular内置了一整套指令，如ngBind, ngModel, 和ngView.就像你可以创建控制器和服务那样.你也可以创建自己的指令来让Angular使用.当Angular 启动器引导你的应用程序时, HTML编译器就会遍历整个DOM，以匹配DOM元素里的指令.

> 对于HTML模板来说，"编译"意味着什么?对于AngularJS来说，`编译`意味着把监听事件绑定在HTML元素上，使其可以交互.我们使用`编译`这个术语的原因就在于，把指令关联到DOM上的这种递归操作非常类似于 编译式语言编译源代码的过程.

### 指令匹配

- angular的`HTML编译器($compile)`是怎样决定该在什么时候调用一个指令?

以元素input标签匹配了`ngModel`的指令为例:

```plain
<input ng-model="foo">
// 等价于下面
<input data-ng:model="foo">
```

Angular把一个元素的标签和属性名字进行规范化，来决定哪个元素匹配哪个指令.我们通常用区分大小写的规范化命名方式(比如ngModel)来识别指令.然而,`HTML是区分大小`的，所以我们在DOM中使用的指令只能用小写的方式命名， 通常使用破折号间隔的形式(比如：ng-model).

规范化的过程如下所示：
1.从元素或属性的名字前面去掉`x-`和`data-`
2.从`:`,`-`, 或`_`分隔的形式转换成`小驼峰命名法`(camelCase).

> 最佳实践:建议使用破折号分隔符的方式(比如ng-bind for ngBind). 如果你想`支持HTML验证工具`，你可以加前缀data.(比如把ngBind写成data-ng-bind). 其它的形式虽然也是合法的,都是历史遗留问题而支持.

`$compile(编译)` 可以基于`元素名字`、`属性`、`类名`和`注释`来匹配指令的.

```plain
<my-dir></my-dir>
<span my-dir="exp"></span>
<!-- directive: my-dir exp -->
<span class="my-dir: exp;"></span>
```

> 最佳实践: 最好通过`标签名`和``属性`来使用指令而不要通过注释和类名。这样做可以更容易地看出一个元素是跟哪个指令匹配的.

通常注释式命名式指令使用在如下情景：某些指令需要跨越多个元素，但是受DOM API的限制，无法跨越多个元素(如:table标签元素)。 AngularJS 1.2 引入了ng-repeat-start和ng-repeat-end指令，作为更好的解决方案。 建议开发者使用这种方式，而不要用“自定义注释”形式的指令。

### 文本和属性绑定

在编译的过程中编译器会使用 $interpolate服务去匹配文本和属性,查看是否含有内嵌表达式.这些表达式会作为``watches`的值来注册,并作为`digest`循环的一部分来进行实时更新。如下:

```plain
<a ng-href="img/{{username}}.jpg">Hello {{username}}!</a>
```

### ngAttr属性绑定

Web浏览器有时候对于属性的合法性检查简直是吹毛求疵.

```plain
// 控制台中报错Error: Invalid value for attribute cx="".
// 这是由于SVG DOM API的限制,不能简单的写为cx="".使用ng-attr-cx
<svg>
  <circle cx="{{cx}}"></circle>
</svg>

// 正确的写法
<svg>
  <circle ng-attr-cx="{{cx}}"></circle>
</svg>
```

## 创建指令

- 谈一下注册指令API的API

和`controller`一样,`directive`也是注册在模块上的.要注册一个指令,可以使用 module.directive API.
module.directive 接受`规范化normalized(前面提到的指令匹配)`的指令名字和工厂方法.此工厂方法应该返回一个带有不同选项的对象来告诉`编译器$compile`此指令被匹配上该做些什么.

工厂函数仅在编译器`第一次匹配`到指令的时候`调用一次`. 你可以在这里进行初始化的工作。该工厂函数使用`$injector.invoke`调用，所以它可以像控制器一样进行依赖注入。

> 尽量返回一个对象，而不要只返回一个函数.

----------------------------------

> 为了防止与未来的标准冲突，最好是`前缀化`自定义指令名字.不要与未来HTML元素产生冲突,不能使用ng或者与angular未来版本起冲突的前缀.推荐使用`两三个单词的前缀`(如btfCarousel).

下面的例子我们将会使用作为`my前缀`,如:myCustomer

### 模板指令扩展

当你有大量代表客户信息的模板。这个模板在你的代码中重复了很多次,当你改变一个地方的时候,你不得不在其他地方同时改动,这时候,你就要使用指令来简化你的模板。

创建一个指令，简单的使用静态模板来替换它的内容.

```plain
<div ng-controller="Ctrl">
  <div my-customer></div>
</div>

最终呈现结果:
Name: Naomi Address: 1600 Amphitheatre
```

```js
angular.module('docsSimpleDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.customer = {
    name: 'Naomi',
    address: '1600 Amphitheatre'
  };
}])
.directive('myCustomer', function() {
  return {
    template: 'Name: {{customer.name}} Address: {{customer.address}}'
  };
});
```

> 除非你的模板非常小,否则最好分割成单独的hmtl文件,然后使用templateUrl选项来加载.

如果你熟悉`ngInclude`,那么会发现`templateUrl`的作用与之类似.
下面是用templateUrl选项的同一个例子:

```js
// script.js
angular.module('docsTemplateUrlDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.customer = {
    name: 'Naomi',
    address: '1600 Amphitheatre'
  };
}])
.directive('myCustomer', function() {
  return {
    templateUrl: 'my-customer.html'
  };
});
```

```plain
// index.html
<div ng-controller="Controller">
  <div my-customer></div>
</div>
```

```plain
// my-customer.html
Name: {{customer.name}} Address: {{customer.address}}
```

> `templateUrl`返回被用于directive加载和使用HTML template的URL,templateUrl有两个参数:
> directive需要的元素和与这个元素相关的属性对象。

注意: 

> 你目前不能从templateurl函数访问`scope变量`,`因为模板在scope之前初始化(重要重要重要)`

组件扩展例子:

```js
// script.js
angular.module('docsTemplateUrlDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.customer = {
    name: 'Naomi',
    address: '1600 Amphitheatre'
  };
}])
.directive('myCustomer', function() {
  return {
    templateUrl: function(elem, attr){
      return 'customer-'+attr.type+'.html';
    }
  };
});
```

```plain
// index.html
<div ng-controller="Controller">
  <div my-customer type="name"></div>
  <div my-customer type="address"></div>
</div>

最终结果:
Name: Naomi
Address: 1600 Amphitheatre
```

```plain
// customer-name.html
Name: {{customer.name}}
```

```plain
// customer-address.html
Address: {{customer.address}}
```

当你创建一个指令,默认为属性或者元素,为了使创建的directive是以类名触发,你需要使用限制参数:
限制参数通常有以下几种设置:

> `A`: 仅匹配属性名
> `E`: 仅匹元素性名
> `C`: 仅匹配类名

这些限制参数还可以被组合:

> `ACE`: 即匹配属性名,又匹配元素或者类名.

如下使用restrict: `E`来匹配directive指令

```js
// script.js
angular.module('docsRestrictDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.customer = {
    name: 'Naomi',
    address: '1600 Amphitheatre'
  };
}])
.directive('myCustomer', function() {
  return {
    restrict: 'E',
    templateUrl: 'my-customer.html'
  };
});
```

```plain
// index.html
<div ng-controller="Controller">
  <my-customer></my-customer>
</div>

最终结果:
Name: Naomi Address: 1600 Amphitheatre
```

```plain
// my-customer.html
Name: {{customer.name}} Address: {{customer.address}}
```
> 什么情况下该用元素名，什么情况下该用属性名？
> 当创建一个含有自己模板的组件的时候,建议使用元素名,常见情况是,当你想为你的模板创建一个DSL（特定领域语言）的时候。如果仅仅想为已有的元素添加功能，建议使用属性名.

使用元素名做为myCustomer指令是非常正确的决定，因为你不是用一些'customer'行为来装饰这个元素，而是定义一个具有自定义行为的元素作为customer组件

### 给指令一个独立作用域(isolate scope)

上面我们的myCustomer指令已经非常好了,但是它有个致命的缺陷,我们在给定的作用域内仅能使用一次。如果要重用,则每次都要给该指令创一个控制器.如下:

```js
// script.js
angular.module('docsScopeProblemExample', [])
.controller('NaomiController', ['$scope', function($scope) {
  $scope.customer = {
    name: 'Naomi',
    address: '1600 Amphitheatre'
  };
}])
.controller('IgorController', ['$scope', function($scope) {
  $scope.customer = {
    name: 'Igor',
    address: '123 Somewhere'
  };
}])
.directive('myCustomer', function() {
  return {
    restrict: 'E',
    templateUrl: 'my-customer.html'
  };
});

```

```plain
// index.html
<div ng-controller="NaomiController">
  <my-customer></my-customer>
</div>
<hr>
<div ng-controller="IgorController">
  <my-customer></my-customer>
</div>

最终结果:
Name: Naomi Address: 1600 Amphitheatre
---------------------------------------
Name: Igor Address: 123 Somewhere
```

```plain
// my-customer.html
Name: {{customer.name}} Address: {{customer.address}}
```

> 以上不是一个好的解决方案.我们想要做的是能够把指令的作用域与外部的作用域隔离开来,然后映射外部的作用域到指令内部的作用域.可以通过创建独立作用域(isolate scope)来达到这个目的.我们可以`使用指令的scope选项`:


```js
// script.js
angular.module('docsIsolateScopeDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.naomi = { name: 'Naomi', address: '1600 Amphitheatre' };
  $scope.igor = { name: 'Igor', address: '123 Somewhere' };
}])
.directive('myCustomer', function() {
  return {
    restrict: 'E',
    scope: {
      customerInfo: '=info'
    },
    templateUrl: 'my-customer-iso.html'
  };
});
```

```plain
// index.html
<div ng-controller="Controller">
  <my-customer info="naomi"></my-customer>
  <hr>
  <my-customer info="igor"></my-customer>
</div>

最终结果:
Name: Naomi Address: 1600 Amphitheatre
---------------------------------------
Name: Igor Address: 123 Somewhere
```

```plain
// my-customer.html
Name: {{customer.name}} Address: {{customer.address}}
```

首先看index.html页面,第一个`<my-customer>`元素使用naomi值(在controller的scope作用域上暴露出来)来绑定info属性.第二个绑定的是igor.

```js
//...
scope: {
  customerInfo: '=info'
},
//...
```

scope是一个对象,包含了为每个独立作用域绑定的属性组合,在这个例子中他是一个属性.
1. 它的名字(customerInfo) 对应于指令里的独立作用域的customerInfo属性
2. 它的值 (=info) 告诉`$compile`绑定在info属性上。

> `重要重要重要`,指令scope选项中的`=attr`属性名是被`规范化`过后的名字.如要绑定`<div bind-to-this="thing">`,scope选项要使用'=bindToThis'进行绑定。
> `重要重要重要`,当scope对象的属性名字和directive指令绑定的属性名一样的时候,可以快捷语法:

```js
...
scope: {
  // same as '=customer'
  customer: '='
},
...
```

使用独立作用域(isolate scope)还有另外一个用处，那就是可以绑定不同的数据到指令内部的作用域.我们可以添加另外一个属性vojta到作用域，然后在我们的指令模板中访问它.
























