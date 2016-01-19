title: "AngularJS Directives详解(2)"
date: 2016-01-19 19:20:14
description: "简单介绍AngularJS Directives实例详解"
tags:
- "angular"
---

### 创建指令

指令也是一种`服务`,只是这种服务的定义有几个特殊要求:

1. 必须使用模块的`directive()`方法注册服务
2. 必须以`对象工厂/factory()`方法定义服务实现
3. 对象工厂必须返回一个`指令定义对象`

```js
//定义指令的类工厂
var directiveFactory = function(injectables){
    //指令定义对象
    var directiveDefinationObject = {
        ...
    };
    return directiveDefinationObject;
};
//在模块上注册指令
angular.module("someModule",[])
.directive("directiveName",directiveFactory);
```

> INSIDE:指令在注入器中的登记名称是:指令名+Directive.例如,ng-app指令的服务名称是："ngAppDirective"。

示例定义一个简单的指令ez-hoverable,这个指令被限制只能出现在属性的位置,每个具有这个指令的HTML元素,将在鼠标移入时以虚线边框突出显示。

```js
var ezHoverableFactory = function(){
  return {
    restrict : "A",
    link : function(scope,element,attrs){
      element.on("mouseover",function(){
        element.css({outline:"#ff0000 dotted thick"});
      })
      .on("mouseout",function(){
        element.css({outline:"none"});
      })
    }
  };
};
angular.module("ezstuff",[])
.directive("ezHoverable",ezHoverableFactory);
```
```plain
<span ez-hoverable>我是SPAN</span>
<button ez-hoverable>我是BUTTON</button>
<div ez-hoverable>我是DIV</div>
```js

## 指令定义对象

每个指令定义的工厂函数,需要返回一个`指令定义对象`.指令定义对象就是一个具有`约定属性`的JavaScript对象,编译器($compile)在编译时就根据这个定义对象对指令进行展开。
指令定义对象的常用`约定属性`如下:

- template : string

使用template指定的HTML标记替换指令内容（或指令自身）

- restrict : sring

用来限定指令在HTML模板中出现的位置。

- replace : `true`|`false`

使用这个属性指明template的替换方式。

- scope :  true|false|{...}

scope属性为指令创建私有的作用域，这在创建可复用的Widget(组件)时非常有用

- link : function(..){...}

link属性是一个函数，用来在指令中操作DOM树、实现数据绑定。

- transclude : true|false|'element'

允许指令包含其他HTML元素，这通常用于实现一个容器类型的Widget。

### template:定义替换模板

最简单的指令只需要使用template属性进行模板替换就可以实现,template指明一个HTML片段，可以用来:
1. 替换指令的内容。这是默认的行为，可以使用replace属性更改.
2. 如果replace = true，那么用HTML片段替换指令本身.
3. 包裹指令的内容，如果transclue属性为true.

```js
angular.module("ezstuff",[])
.controller("ezCtrl", ["$scope", function($scope) {
  $scope.customer = {
    name: "Naomi",
    address: "1600 Amphitheatre"
  };
}])
.directive("ezCustomer", function() {
  return {
    template: "Name: {{customer.name}} Address: {{customer.address}}"
  };
});
```

```plain
<div ez-customer></div>
经过$compile编译后生成的结构:
<div ez-customer>
  Name: {{customer.name}} Address: {{customer.address}}
</div>
将数值填充就行
```

### restrict: 限制指令的出现位置

restict属性可以是EACM这四个字母的`任意组合`,用来限定指令的应用场景。如果不指定这个属性,默认情况下,指令将仅允许被用作`元素名`和`属性名`:
```plain
E - 指令可以作为HTML元素使用
A - 指令可以作为HTML属性使用
C - 指令可以作为CSS类使用
M - 指令可以在HTML注释中使用
```

我们对之前的示例，增加一个restrict属性，限制这个只能作为元素名使用。代码已经预置到右边，你可以看到，现在唯一合法的方式是使用如下方式应用指令：

```plain
<ez-customer></ez-customer>

经过$compile之后变成:
<ez-customer>
  Name: {{customer.name}} Address: {{customer.address}}
</ez-customer>
```

### replace 模板的使用方式(重要)

我们希望使用template完整地替换原始的DOM对象,而不是填充其内容,`replace`属性负责这件事。replace属性指明使用template时,如何替换指令元素:
1. `true` - 编译时,将使用template替换指令元素
2. `false` - 编译时，将使用template替换指令元素的内容

```js
angular.module("ezstuff",[])
.controller("ezCtrl", ["$scope", function($scope) {
  $scope.customer = {
    name: "Naomi",
    address: "1600 Amphitheatre"
  };
}])
.directive("ezCustomer", function() {
  return {
    restrict:"E",
    replace:true,
    template: "<div>Name: {{customer.name}} Address: {{customer.address}}</div>"
  };
});
```

```plain
<ez-customer></ez-customer>

经过编译后成为:

<div>Name: {{customer.name}} Address: {{customer.address}}</div>
```

### 作用域问题

默认情况下，指令没有自己的scope对象，换句话说，它使用所在DOM对象对应的scope对象。那么问题来了，如果一个指令在同一个scope内出现多次，会怎样？

```plain
<div ng-controller="ezCtrl">
    <ez-customer></ez-customer>
    <ez-customer></ez-customer>
</div>
```

没错，由于两个ez-customer指令都处在ezCtrl开辟的作用域内，所以两个指令绑定到了同样的 数据模型上，得到的是重复的结果。

显然，我们可以将每个ez-customer指令置于不同的作用域下，这意味着我们给每个ez-customer 一个不同的控制器：

```plain
<div ng-controller="ezCtrl1">
    <ez-customer></ez-customer>
</div>
<div ng-controller="ezCtrl2">
    <ez-customer></ez-customer>
</div>
```
看起来很怪异，对吗？

### scope:使用隔离的作用域

通过设置scope属性,指令的每个实例都将获得一个隔离的本地作用域:

```plain
<ez-customer name="Emmy.name" address="Emmy.address">
</ez-customer>
```

```js
angular.module("ezstuff",[])
.controller("ezCtrl", ["$scope", function($scope) {
  $scope.Emmy = {
    name: "Emmy",
    address: "1600 Amphitheatre"
  };
])
.directive("ezCustomer", function() {
  return {
    restrict:"E",
    replace:true,
    scope:{
      name : "@name", 
      address : "=address"
    },
    template: "<div>name:{{name}} address:{{address}}</div>"
  };
});
```

在上面的例子中，我们在本地scope上定义了两个属性：name和address，这样在 模板中就可以使用name和address了。

> 你应该已经注意到,name属性的值之前有一个`@attr`,这是一个约定好的标记,它告诉编译器,本地scope上的name值需要从应用这个指令的DOM元素的name属性值读取,如果DOM元素的name属性值变了,那么本地scope上的name值也会变化。

> 同样,address属性之前的`=attr`也是一个约定好的标记，它告诉编译器，本地scope上的address属性值和DOM元素的address属性值指定的外部scope对象上的模型需要建立双向连接：外部scope上模型的变化会改变本地scope上的address属性，本地scope上address属性的变化也会改变外部scope上模型的变化。

从图中可以看出：
1. 指令的template绑定的是本地scope上的name和address。
2. 本地scope的name属性的值始终是ez-customer对象上name属性的值
3. 本地scope的address属性值始终和ez-customer对应的scope对象上的Emmy.address保持同步。

### link:在指令中操作DOM

如果需要在指令中操作DOM,我们需要在对象中定义link属性,link函数的定义如下:

```js
function link(scope, iElement, iAttrs, controller, transcludeFn) { ... }
```

注意link函数的参数,AngularJS在`编译`时负责传入正确的值:

- scope

指令对应的scope对象.如果指令没有定义自己的本地作用域,那么传入的就是外部的作用域对象。

- iElement

指令所在DOM对象的jqLite封装.如果使用了template属性,那么iElement对应变换后的DOM对象的jqLite封装。

- iAttrs

指令所在DOM对象的属性集.这是一个Hash对象,每个键是驼峰规范化后的属性名。

> 例子参考[AngularJS Directives详解(1)时钟](http://blog.uijack.com/2016/01/13/ng-directives/)

### transclude:包含其他元素

有些指令需要能够包含其他未知的元素.比如我们定义一个指令ez-dialog,用来封装对话框的样式和行为,它应当允许在使用期(也就是在界面模板文件里)才指定其内容:

```plain
<ez-dialog>
  <p>对话框的内容在我们开发ez-dialog指令的时候是无法预计的。这部分内容需要转移到展开的DOM树中适当的位置。</p>
</ez-dialog>
```

`transclude`属性可以告诉编译器,利用所在DOM元素的内容,替换template中包含`ng-transclude`指令的元素的内容:
![$compile解析图](/img/ng-directive.png)

从上图中可以看到，使用transclude有两个要点：
1. 需要首先声明transclude属性值为true,这将告诉编译器,使用我们这个指令的DOM元素,其内容需要被`复制`并`插入`到编译后的DOM树的某个点。
2. 需要在template属性值中使用ng-transclude指明`插入点`。
