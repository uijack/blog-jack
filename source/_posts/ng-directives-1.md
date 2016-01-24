title: "AngularJS Directives详解(1)"
date: 2016-01-13 07:12:06
description: "总结AngularJS Directives的使用和注意事项"
tags:
- "angular1.x"
---

## 创建自定义指令

此文解释了在你的AngularJS应用里，何时该创建自定义指令以及如何去实现它们.

### "指令(directive)"是什么?

简单点说，指令就是一些附加在HTML元素上的自定义标记（例如：属性，元素，或css类）,它告诉AngularJS的HTML编译器 ($compile) 在元素上附加某些指定的行为，甚至操作DOM、改变DOM元素，以及它的各级子节点.

Angular内置了一整套指令，如ngBind, ngModel, 和ngView.就像你可以创建控制器和服务那样.你也可以创建自己的指令来让Angular使用.当Angular 启动器引导你的应用程序时, HTML编译器就会遍历整个DOM，以匹配DOM元素里的指令.

> 对于HTML模板来说,"编译"意味着什么?对于AngularJS来说,`编译`意味着把监听事件绑定在HTML元素上,使其可以交互.我们使用`编译`这个术语的原因就在于,把指令关联到DOM上的这种递归操作非常类似于编译式语言编译源代码的过程.

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

> `最佳实践`: 最好通过`标签名`和`属性`来使用指令而不要通过注释和类名。这样做可以更容易地看出一个元素是跟哪个指令匹配的.

通常注释式命名式指令使用在如下情景：某些指令需要跨越多个元素，但是受DOM API的限制，无法跨越多个元素(如:table标签元素)。 AngularJS 1.2 引入了ng-repeat-start和ng-repeat-end指令，作为更好的解决方案。 建议开发者使用这种方式，而不要用“自定义注释”形式的指令。

### 文本和属性绑定

在编译的过程中编译器会使用 $interpolate服务去匹配文本和属性,查看是否含有内嵌表达式.这些表达式会作为`watches`的值来注册,并作为`digest`循环的一部分来进行实时更新。如下:

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
    templateUrl: function(elem, attr){  //这个很重要
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
### 指令 restrict 选项

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

使用元素名做为myCustomer指令是非常正确的决定，因为你不是用一些`customer`行为来装饰这个元素，而是定义一个具有自定义行为的元素作为customer组件

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

使用独立作用域(isolate scope)还有另外一个用处，那就是可以绑定不同的数据到指令内部的作用域.我们可以添加另外一个属性vojta到作用域，然后在我们的指令模板中访问它.如果没有绑定,则在指令模板中无法访问.

```js
angular.module('docsIsolationExample', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.naomi = { name: 'Naomi', address: '1600 Amphitheatre' };
  $scope.vojta = { name: 'Vojta', address: '3456 Somewhere Else' };
}])
.directive('myCustomer', function() {
  return {
    restrict: 'E',
    scope: {
      customerInfo: '=info'
    },
    templateUrl: 'my-customer-plus-vojta.html'
  };
});
```

```plain
// index.html
<div ng-controller="Controller">
  <my-customer info="naomi"></my-customer>
</div>

最终结果:
Name: Naomi Address: 1600 Amphitheatre
---------------------------------------
Name: Address:
```

```plain
// my-customer-plus-vojta.html
Name: {{customerInfo.name}} Address: {{customerInfo.address}}
<hr>
Name: {{vojta.name}} Address: {{vojta.address}}
```

注意,`{{vojta.name}}`和`{{vojta.address}}` 都是空的,意味着他们是`undefined`,虽然我们在控制器中定义了vojta,但是在指令内部访问不到.

指令的`独立作用域(isolate scope)`隔离除了你添加到scope:{}对象中的数据模型之外的一切东西.这对于建立一个`可复用的组件`来说是非常有用的,因为它可以阻止除你传入数据模型之外的一切东西改变你内部数据模型的状态.

> 普通的作用域都使用原型方式继承自父作用域。但是独立作用域没有这样的继承关系,参考更多资料[Directive Definition Object - scope](https://code.angularjs.org/1.4.8/docs/api/ng/service/$compile#directive-definition-object)

--------------------------------------

> `最佳实践`: 如果要使你的组件在应用范围内可重用，那么使用scope选项去创建一个独立作用域

## 创建操作DOM的指令

需求: 我们会创建一个显示当前时间的指令，每秒一次更新DOM以正确的显示当前的时间。

### 创建须知

directive修改DOM一般使用`link函数`来注册DOM监听,同时更新DOM,当模板被克隆并且指令逻辑执行后他才会被触发.
`link`选项接受一个带有如下签名的函数function link(scope, element, attrs, controller, transcludeFn) {}

1. scope: 是一个angular scope对象
2. element: 指令匹配的jqLite封装的元素(angular内部实现的类似jquery的库)
3. attrs: 一个带有`规范化normalized`属性名字和相应值的对象.
4. controller: 指令需要的controller的实例或者指令自带的controller,确切的值取决于该指令的要求属性。
5. transcludeFn: 是一个嵌入的链接功能`预绑定`到正确的嵌入包含范围

[更多link选项参考$compile API page](https://code.angularjs.org/1.4.8/docs/api/ng/service/$compile#-link-)

### 创建例子

在我们的link 函数中，我们每秒更新一次显示时间，当用户改变绑定的时间格式字符串的时候也会更新。当指令被删除的时候，我们也要移除定时器，以避免引入内存泄露。

```js
// script.js
angular.module('docsTimeDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.format = 'M/d/yy h:mm:ss a';
}])
.directive('myCurrentTime', ['$interval', 'dateFilter', function($interval, dateFilter) {

  function link(scope, element, attrs) {
    var format,
        timeoutId;

    function updateTime() {
      element.text(dateFilter(new Date(), format));
    }

    scope.$watch(attrs.myCurrentTime, function(value) {
      format = value;
      updateTime();
    });

    element.on('$destroy', function() {
      $interval.cancel(timeoutId);
    });

    // start the UI update process; save the timeoutId for canceling
    timeoutId = $interval(function() {
      updateTime(); // update DOM
    }, 1000);
  }

  return {
    link: link
  };
}]);
```

```plain
<div ng-controller="Controller">
  Date format: <input ng-model="format"> <hr/>
  Current time is: <span my-current-time="format"></span>
</div>
```

1. module.directive函数的参数和controller一样也是通过`依赖注入`获得的,因此才可以在link函数内部使用$timeout和dateFilter服务.

2. 当一个被angular编译过的DOM元素被移除的时候,它会触发一个`$destroy`事件,同样的,当一个`angular作用域`被移除的时候,它会`向下传播(broadcast)$destroy事件`到所有下级作用域。

3. 通过监听事件,你可以移除可能引起内存泄露的事件监听器,注册在元素和作用域上的监听器在它们被移除的时候,会自动会清理掉.但是假如注册一个事件在服务或者没有被删除的DOM节点上,你就必须手工清理,否则会有内存泄露的风险

> `最佳实践`: 指令应该自己管理自身分配的内存。当指令被移除时,你可以使用element.on('$destroy', ...)或 scope.$on('$destroy', ...)来执行一个清理的工作.

### 创建一个包含其他元素的指令

我们现在已经实现了使用独立作用域传递数据模型到指令里面,但是有时候我们需要能够传进去整个模板而不是字符串或者对象,让我们通过创建`dialog box`组件来演示它。这个组件应该能够包裹任意内容,要想实现这个,我们需要使用`transclude`选项。

```js
// script.js
angular.module('docsTransclusionDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.name = 'Tobias';
}])
.directive('myDialog', function() {
  return {
    restrict: 'E',
    transclude: true,
    templateUrl: 'my-dialog.html'
  };
});
```

```plain
// index.html
<div ng-controller="Controller">
  <my-dialog>Check out the contents, {{name}}!</my-dialog>
</div>

结果:
Check out the contents, Tobias!
```

```plain
// my-dialog.html
<div class="alert" ng-transclude>
</div>
```

```plain
// 最终的渲染结构:
<body ng-app="docsTransclusionDirective" class="ng-scope">
  <div ng-controller="Controller" class="ng-scope">
    <my-dialog>
      <div class="alert" ng-transclude="">
        <span class="ng-binding ng-scope">Check out the contents, Tobias!</span>
      </div>
    </my-dialog>
  </div>
</body>
```

> 这个transclude选项用来干嘛呢？带有transclude选项的指令,指令的scope`访问外部的作用域scope`而不是自己的scope。

为了证明这个,请看上面例子改写的script.js:

```js
angular.module('docsTransclusionExample', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.name = 'Tobias';
}])
.directive('myDialog', function() {
  return {
    restrict: 'E',
    transclude: true,
    scope: {},
    templateUrl: 'my-dialog.html',
    link: function (scope, element) {
      scope.name = 'Jeff';
    }
  };
});
```

输出结果:Check out the contents, Tobias!

我们会认为{{name}}会被解析为Jeff,然而例子中的{{name}}被解析成了Tobias.
`transclude`选项改变了指令scope嵌套的方式,他使指令的内容拥有任何指令外部的作用域,而不是内部的作用域.此时内部作用域scope使用`=attr`是无效的,但是可以使用`&fun`来绑定函数是可以的.

> 如果指令不创建自己的scope(就是说scope:false，或省略),然后在link函数里执行scope.name = 'Jeff'; 很明显外部的scope会受影响,因为指令是继续了外部的scope ,在输出上会看出 Jeff.

这样的行为对于包含内容的指令是非常有意义的。因为如果不这样的话,当你需要数据模型的时候,你就必须分别传入这个你需要使用的数据模型,那么你就无法做到适应各种不同内容的情况.

> `最佳实践`: 仅当你要创建一个包裹任意内容的指令的时候使用transclude: true

接下来我们增加一个按钮到'dialog box'组件里面，允许用户使用指令绑定自己定义的行为。

```js
// script.js
angular.module('docsIsoFnBindExample', [])
.controller('Controller', ['$scope', '$timeout', function($scope, $timeout) {
  $scope.name = 'Tobias';
  $scope.message = '';
  $scope.hideDialog = function (message) {
    $scope.message = message;
    $scope.dialogIsHidden = true;
    $timeout(function () {
      $scope.message = '';
      $scope.dialogIsHidden = false;
    }, 2000);
  };
}])
.directive('myDialog', function() {
  return {
    restrict: 'E',
    transclude: true,
    scope: {
      'close': '&onClose'
    },
    templateUrl: 'my-dialog-close.html'
  };
});
```

```plain
// index.html
<div ng-controller="Controller">
  {{message}}
  <my-dialog ng-hide="dialogIsHidden" on-close="hideDialog(message)">
    Check out the contents, {{name}}!
  </my-dialog>
</div>
```

```plain
// my-dialog-close.html
<div class="alert">
  <a href class="close" ng-click="close({message: 'closing for now'})">&times;</a>
  <div ng-transclude></div>
</div>
```

```plain
// 最终生成html结构:
<body ng-app="docsIsoFnBindExample" class="ng-scope">
  <div ng-controller="Controller" class="ng-scope ng-binding">
    {{message}} // 此时的{{message}}为'',不显示
    <my-dialog ng-hide="dialogIsHidden" on-close="hideDialog(message)" class="ng-isolate-scope">
      <div class="alert">
        <a href="" class="close" ng-click="close({message: 'closing for now'})">×</a>
        <div ng-transclude="">
          <span>Check out the contents, Tobias!
          </span>
        </div>
      </div>
    </my-dialog>
  </div>
</body>

```

我们想要通过在指令的作用域上调用我们传进去的函数,但是这个函数本该运行在定义时候的上下文。

先前我们看到如何在scope选项中使用=attr,但是本例子中却使用了`&attr`,`&`允许绑定了原scope的函数到独立作用域,允许独立作用于调用它.同时保留了来函数的作用域.所以当用户点击X的时候,就会运行Controller控制器的close函数.

注意:此时`<div ng-transclude="">`用来站位,填充`<my-dialog>`的内容,如果放到class='alert'这个div上面,则里面的a标签内容将被替换掉.

> `最佳实践`: 当你的指令想要开放一个API去绑定特定的行为，在scope选项中使用&attr。

### 创建一个带事件监听的指令

先前,我们使用link函数创建一个操作DOM元素的指令,基于上面的例子,我们创建一个监听元素的事件,以作出相应操作的指令。
比如说,假如我们想要创建一个让用户可拖曳的元素,该怎么做呢？

```js
// script.js
angular.module('dragModule', [])
.directive('myDraggable', ['$document', function($document) {
  return {
    link: function(scope, element, attr) {
      var startX = 0, startY = 0, x = 0, y = 0;

      element.css({
       position: 'relative',
       border: '1px solid red',
       backgroundColor: 'lightgrey',
       cursor: 'pointer'
      });

      element.on('mousedown', function(event) {
        // Prevent default dragging of selected content
        event.preventDefault();
        startX = event.pageX - x;
        startY = event.pageY - y;
        $document.on('mousemove', mousemove);
        $document.on('mouseup', mouseup);
      });

      function mousemove(event) {
        y = event.pageY - startY;
        x = event.pageX - startX;
        element.css({
          top: y + 'px',
          left:  x + 'px'
        });
      }

      function mouseup() {
        $document.off('mousemove', mousemove);
        $document.off('mouseup', mouseup);
      }
    }
  };
}]);
```

```plain
// index.html
<span my-draggable>Drag ME</span>
```

### 创建相互通信的指令

你可以通过在模板中使用指令来把任意指令组合起来.有时候你可能想要一个由一组指令组合组成的组件.假设你想要一个带有tab的容器,容器的内容对应于当前激活的tab.

```js
// script.js
angular.module('docsTabsExample', [])
.directive('myTabs', function() {
  return {
    restrict: 'E',
    transclude: true,
    scope: {},
    controller: ['$scope', function($scope) {
      var panes = $scope.panes = [];

      $scope.select = function(pane) {
        angular.forEach(panes, function(pane) {
          pane.selected = false;
        });
        pane.selected = true;
      };

      this.addPane = function(pane) {
        if (panes.length === 0) {
          $scope.select(pane);
        }
        panes.push(pane);
      };
    }],
    templateUrl: 'my-tabs.html'
  };
})
.directive('myPane', function() {
  return {
    require: '^myTabs',
    restrict: 'E',
    transclude: true,
    scope: {
      title: '@'
    },
    link: function(scope, element, attrs, tabsCtrl) {
      tabsCtrl.addPane(scope);
    },
    templateUrl: 'my-pane.html'
  };
});

```

```plain
// index.html
<my-tabs>
  <my-pane title="Hello">
    <h4>Hello</h4>
    <p>Lorem ipsum dolor sit amet</p>
  </my-pane>
  <my-pane title="World">
    <h4>World</h4>
    <em>Mauris elementum elementum enim at suscipit.</em>
    <p><a href ng-click="i = i + 1">counter: {{i || 0}}</a></p>
  </my-pane>
</my-tabs>
```

```plain
// my-tabs.html
<div class="tabbable">
  <ul class="nav nav-tabs">
    <li ng-repeat="pane in panes" ng-class="{active:pane.selected}">
      <a href="" ng-click="select(pane)">{{pane.title}}</a>
    </li>
  </ul>
  <div class="tab-content" ng-transclude></div>
</div>
```

```plain
// my-pane.html
<div class="tab-pane" ng-show="selected" ng-transclude>
</div>
```
myPane指令有一个require的选项,其值为:^myTabs.当指令使用这个选项,$compile服务会查找一个名叫myTabs的控制器,如果没有找到,就会抛出一个错误.^前缀意味着指令将会在它的父元素上面搜索控制器(如果没有^前缀,指令默认只在所属元素上搜索指定的控制器).

这里myTabs的控制器是来自何处呢？简单的通过controller选项就可以为指令定义一个控制器,比如上面例子中myTabs 就使用了这个选项.如ngController一样,`此controller选项把这个控制器绑定到了指令的模板上`。

回顾myPane的定义,你会注意到`link函数`的最后一个参数: tabsCtrl,当一个指令需要(require)一个控制器时,它会接收该指令的控制器实例作为link函数的第四个参数,通过它，myPane就可以调用myTabs的addPane函数了。

如果需要多个controller,则可以数组表示:

```js
angular.module('docsTabsExample', [])
.directive('myPane', function() {
  return {
    require: ['^myTabs', '^ngModel'],
    restrict: 'E',
    transclude: true,
    scope: {
      title: '@'
    },
    link: function(scope, element, attrs, controllers) {
      var tabsCtrl = controllers[0],
          modelCtrl = controllers[1];

      tabsCtrl.addPane(scope);
    },
    templateUrl: 'my-pane.html'
  };
});
```

聪明的读者可能想知道link 和 controller之间的区别.最基本的区别就是:

> controller可以`导出一个API`,而`子指令`的link函数可以通过`require来`与这个API交互。
> `最佳实践`: 当你想暴露一个API给其它的指令调用那就用controller(使用`this`),否则用link。

如果你想更深入的了解编译的处理过程,可以查看HTML编译器.[$compile API](http://docs.ngnice.com/api/ng.$compile)页面有directive每个选项的具体解释,可参阅.
