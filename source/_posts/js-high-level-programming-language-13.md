title: "读《JavaScript高级程序设计》总结13"
date: 2015-12-31 7:27:59
description: "读《JavaScript高级程序设计 -- 第12章 事件》主要介绍事件流,事件处理程序,事件对象,事件类型,内存和性能以及模拟事件"
tags:
- "JavaScript"
- "随笔"
---

## 事件流

JavaScript与HTML之间交互是通过事件流.

问题: 可以想象画在一张纸上的一组同心圆.如果你把手指放在圆心,那么你的手指指向的不是一个圆,而是纸上的所有圆.

### 事件冒泡流

IE事件流叫做`事件冒泡(event bubbling)`.即事件开始时由最具体的元素(文档中嵌套层次最深的那个节点)接收.然后逐级向上传播到较为不具体的接节点(文档).

```plain
<!DOCTYPE html>
<html>
  <head>
    <title></title>
  </head>
  <body>
    <div id="myDiv">Click Me!</div>
  </body>
</html>
```

单击div元素,clic事件会按照如下顺序传播:div -> body -> html -> document.

### 事件捕获

Netscape 团队提出另一种事件流叫做`事件捕获`.思想是:不太具体的节点应该更早接收到事件,而最具体的节点应该最后接收到事件.事件捕获的用意在于`事件达到预定目标之前捕获它`.以前面例子为例: document > html -> body -> div.目前IE9,Safari,Chrome,Opera和Firefox都支持这种事件流模型.`DOM2级事件`规范要求事件应该从document对象开始,但这些浏览器都是从window对象开始捕获事件的.由于老版本浏览器不支持,建议使用`事件冒泡`,在特殊时候在使用`事件捕捉`.

### DOM事件流

`DOM2级事件`规定事件流包含三个阶段:
> 1. 事件捕捉
> 2. 处于目标阶段
> 3. 事件冒泡阶段

首先发生的是事件捕获,为截获事件提供机会.然后是实际的目标接收到事件.最后一个阶段是冒泡阶段,可以在这个阶段对事件做出响应.`DOM2级事件`规定明确要求在捕获阶段不会涉及事件目标.但IE9+,Sa,CH,Ff,Op9.5+更高版本都会在捕获阶段触发事件对象上的事件,结果就有两个机会在目标是对象上操作事件.

## 事件处理程序

### DOM0级事件处理程序

```js
var btn = document.getElementById('myBtn');
btn.onclick = function() {
  alert('Clicked');
  alert(this.id); //'myBtn' 可以在事件处理程序中通过`this`访问元素的任何属性和方法.
};

btn.onclick = null; // 删除事件处理程序.
```

响应某个事件的函数就叫做`事件处理程序(事件侦听器)`.事件处理程序的名字以`on`开头.

### DOM2级事件处理程序

- addEventListener()
- removeEventListener()

第三个参数是boolean值,如果为true,表示在捕获阶段调用事件处理程序. false表示在事件冒泡阶段调用事件处理程序.

```js
var btn = document.getElementById('myBtn');
btn.onaddEventListener('click', function(){
  alert('Clicked');
}, false);

btn.onaddEventListener('click', function(){
  alert(this.id); //'myBtn'
}, false);
```

removeEventListener不能移除匿名函数.

### IE事件处理程序

- attachEvent()
- detachEvent()

此时的this就是window,因为事件处理程序会在全局作用域中运行.

### 跨浏览器事件处理

EventUtil.js `源码还有很多兼容扩展方法`.

```js
var EventUtil = {
  addHandler: function(element, type, handler){
    if (element.addEventListener){  // DOM2级方法
      element.addEventListener(type, handler, false);
    } else if (element.attachEvent){ // IE
      element.attachEvent("on" + type, handler);
    } else {
      element["on" + type] = handler;  // DOM0级方法
    }
  },

  removeHandler: function(element, type, handler){
    if (element.removeEventListener){
      element.removeEventListener(type, handler, false);
    } else if (element.detachEvent){
      element.detachEvent("on" + type, handler);
    } else {
      element["on" + type] = null;
    }
  },
}
```

## 事件处理

### DOM中的事件对象

```js
var btn = document.getElementById("myBtn");
  var handler = function(event){
    switch(event.type){
      case "click":
        alert("Clicked");
        break;
          
      case "mouseover":
        event.target.style.backgroundColor = "red";
        break;
          
      case "mouseout":
        event.target.style.backgroundColor = "";
        break;                        
    }
  };
  
  btn.onclick = handler;
  btn.onmouseover = handler;
  btn.onmouseout = handler;
```
只有设置event的`cancelable`属性值为true的事件,才可以使用`preventDefault()`来取消其默认行为,
`stopPropagation()`方法用于立即停止事件在DOM层中传播.即取消进一步的事件捕获或冒泡.

- event.eventPhase: 1捕获阶段, 2目标对象, 3事件冒泡. 表明事件处于那个流

### IE中的事件对象

- cancelBubble

### 跨浏览器的事件对象

```js
var EventUtil = {

  addHandler: function(element, type, handler){
    if (element.addEventListener){
      element.addEventListener(type, handler, false);
    } else if (element.attachEvent){
      element.attachEvent("on" + type, handler);
    } else {
      element["on" + type] = handler;
    }
  },

  getButton: function(event){
    if (document.implementation.hasFeature("MouseEvents", "2.0")){
      return event.button;
    } else {
      switch(event.button){
        case 0:
        case 1:
        case 3:
        case 5:
        case 7:
          return 0;
        case 2:
        case 6:
          return 2;
        case 4: 
          return 1;
      }
    }
  },

  getCharCode: function(event){
    if (typeof event.charCode == "number"){
      return event.charCode;
    } else {
      return event.keyCode;
    }
  },

  getClipboardText: function(event){
    var clipboardData =  (event.clipboardData || window.clipboardData);
    return clipboardData.getData("text");
  },

  getEvent: function(event){
    return event ? event : window.event;
  },

  getRelatedTarget: function(event){
    if (event.relatedTarget){
      return event.relatedTarget;
    } else if (event.toElement){
      return event.toElement;
    } else if (event.fromElement){
      return event.fromElement;
    } else {
      return null;
    }
  },

  getTarget: function(event){
    return event.target || event.srcElement;
  },

  getWheelDelta: function(event){
    if (event.wheelDelta){
      return (client.engine.opera && client.engine.opera < 9.5 ? -event.wheelDelta : event.wheelDelta);
    } else {
      return -event.detail * 40;
    }
  },

  preventDefault: function(event){
    if (event.preventDefault){
      event.preventDefault();
    } else {
      event.returnValue = false;
    }
  },

  removeHandler: function(element, type, handler){
    if (element.removeEventListener){
      element.removeEventListener(type, handler, false);
    } else if (element.detachEvent){
      element.detachEvent("on" + type, handler);
    } else {
      element["on" + type] = null;
    }
  },

  setClipboardText: function(event, value){
    if (event.clipboardData){
      event.clipboardData.setData("text/plain", value);
    } else if (window.clipboardData){
      window.clipboardData.setData("text", value);
    }
  },

  stopPropagation: function(event){
    if (event.stopPropagation){
      event.stopPropagation();
    } else {
      event.cancelBubble = true;
    }
  }
};
```

## 事件类型

DOM3级事件:
1. UI(User Interface用户界面)事件.用户与页面元素交互是触发.
2. 焦点事件
3. 鼠标事件
4. 滚动事件
5. 文本事件(输入文本触发)
6. 键盘事件
7. 合成事件
8. 变动事件

### UI事件

- load: 当页面加载完成后在window上面触发.图片加载完赋值给src.(重要)
- unload: 当页面完全卸载后再window上触发
- abort:用户停止下载过程中
- error
- select
- resize
- scroll

### 焦点事件

- blur :失去焦点触发,不会冒泡,所有浏览器都支持.
- focus: 获得焦点出发,不会冒泡,所有浏览器都支持.
- focusin: 获得焦点出发,会冒泡. (支持有限)
- focusout: 失去焦点触发.会冒泡.(支持有限)

确定浏览器是否支持,可以使用如下代码检测:
var isSupported = document.implementation.hasFeature("FocusEvents", "3.0");

### 鼠标与滚轮事件

- click
- dblclick
- mousedown
- mouseenter
- mouseleave
- mousemove
- mouseout
- mouseover
- mouseup

注意先后顺序以及事件依赖; 详情看PDF

### 键盘与文本事件

- 键码 发生`keydown` 和 `keyup`事件,event对象的keyCode属性包含一个值,与键盘上的特定键位对应.

```js
var textbox = document.getElementById('myText');
EventUtil.addHandler(textbox, 'keyup', function(event) {
  event = EventUtil.getEvent(event);
  alert(event.keyCode);
});
```
![键码图](/img/js-high-level-13-1.png)

- 文本类型属性`inputMethod`: 文本输入到文本框中的方式

0:浏览器不确定输入
1: 键盘输入
2: 粘贴
3: 拖放进来
4: 使用IME输入
5: 表单选择某项输入
6: 手写输入
7: 语音输入
8: 通过几种组合方式输入
9: 通过脚本输入

### 复合事件

DOM3级事件新添加的一类事件,用于处理IME(输入法编辑器)输入序列,在物理键盘上找不到的字符.

### 变动事件

### HTML5事件

- contextmenu 事件
- beforeunload 事件
- DOMContentLoaded事件: 与load不同
- readystatechange 事件
- pageshow,pagehide
- hashchange

### 设备事件

- orientationchange 事件,IOS为Safari切屏模式
- MozOrientation
- deviceorientation 设备方向变化
- devicemotion 设备移动 

### 触摸与手势

- 触摸事件

touchstart,touchmove, touchend, touchcancel,

- 手势事件

gesturestart,gesturechange, gestureend

## 内存和性能

### 事件委托

对`事件处理程序过多`问题的解决方案就是`事件委托`.其利用了`事件冒泡`.只指定一个事件处理程序,就可以管理某一类型的所有事件.

```plain
<ul id="myLinks">
  <li id="goSomewhere">goSomewhere</li>
  <li id="doSomething">doSomething</li>
  <li id="sayHi">sayHi</li>
</ul>
```
其中包含三个被点击后会执行操作的列表项.按照传统做法,会分别给每一个li添加事件处理程序.
如果是一个复杂的Web应用程序,就会有数不清的代码用于添加时间按处理程序.
可以利用`事件委托`技术解决问题.

```js
(function(){
  var list = document.getElementById("myLinks");
    
  EventUtil.addHandler(list, "click", function(event){
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);

    switch(target.id){
      case "doSomething":
        document.title = "I changed the document's title";
        break;

      case "goSomewhere":
        location.href = "http://www.wrox.com";
        break;

      case "sayHi":
        alert("hi");
        break;
    }
  });
})();
```

将事件委托给ul元素.最适合采用事件委托的技术的事件包括:click,mousedown,mouseup,keydown,keyup,keypress.
虽然mouseover和mouseout事件也冒泡,但是处理并不太容易,需要经常计算元素位置.

### 事件移除

有些时候虽然html内容被innerHTML替换了,但是绑定的事件还留在内存中.造成性能问题.
所以设置div的innerHTMl属性之前,现移除掉不用的事件处理程序,在替换掉,可以保存内存被再次利用.

## 模拟事件

### DOM中的事件模拟

- 模拟鼠标事件
- 模拟键盘事件
- 模拟其他事件
- 自定义DOM事件

### IE中的事件模拟



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
