title: "读《JavaScript高级程序设计》总结17"
date: 2016-01-03 19:52:14
description: "读《JavaScript高级程序设计 -- 第17章 错误处理与调试》介绍浏览器报告的错误,错误处理，调试技术和常见的IE错误"
tags:
- "JavaScript"
- "随笔"
---


## 浏览器报告的错误

> IE,Tool(工具) -> Internet Options(Internet选项) -> Advanced(高级)选项卡，-> Display a notifacation about every script error(显示每个脚本错误的通知)。

> Firefox, Tool -> Error Console.流行的插件`Firebug`.

> Safari, Develop -> Edit -> Preferences > Advanced -> Show develop menu in menubar

> Chrome, Control this page -> Developer -> JavaScript console

## 错误处理

### try-catch语句

将可能导致错误的代码房子try里面，把用于处理错误的代码放在catch里面。finally字句可选。

### 错误类型

ECMA-262定义了下列7种错误类型：

1. Error
2. EvalError： 使用eval()函数发生异常抛出，或者没有把eval()当成函数调用，就会抛错。
3. RangeError： 数值超出相应范围时触发。
4. ReferenceError
5. SyntaxError
6. TypeError
7. URLError

其中Error是基类，其他错误类型都继承自该类。可以使用error instanceof 类型匹配错误。

### 抛出错误

`throw`操作符。

### 错误error事件

任何没有同通过try-catch处理的错误都会触发window对象的error事件。

### 错误处理的策略

### 常见错误类型

1. 类型转换错误： 5=='5', 5==='5'
2. 数据类型错误：通过检查类型确保安全
3. 通信错误

### 区分致命错误和非致命错误

非致命错误：
1. 不影响用户的主要任务
2. 只影响页面的一部分
3. 可以恢复
4. 重复相同操作可以消除错误

致命错误：
1. 应用程序根本无法运行
2. 错误明显影响到用户的主要操作。
3. 会导致其他连带错误

### 把错误记录到服务器

## 调试技术

### 将消息记录到控制台

通过`console`对象的方法：

```plain
error(message);
info(meaasge);
log(message);
warn(message);
```

### 将消息记录到当前页面

在页面中开辟一小块区域，可以用于显示消息。这个区域通常是元素。

```js
function log(message){                    
  var console = document.getElementById("debuginfo");
  if (console === null){
    console = document.createElement("div");
    console.id = "debuginfo";
    console.style.background = "#dedede";
    console.style.border = "1px solid silver";
    console.style.padding = "5px";
    console.style.width = "400px";
    console.style.position = "absolute";
    console.style.right = "0px";
    console.style.top = "0px";
    document.body.appendChild(console);
  }
  console.innerHTML += "<p>" + message + "</p>";
}
  
function sum(num1, num2){
  log("Entering sum(), arguments are " + num1 + "," + num2);

  log("Before calculation");
  var result = num1 + num2;
  log("After calculation");

  log("Exiting sum()");
  return result;
}

var result = sum(10, 23);
```

### 抛出错误

确定错误消息很具体，可以确定错误源。 throw.

## 常见的IE错误

### 终止操作

只能修改已经加载完毕的元素

### 无效字符

invalid character

### 未找到成员

### 位置运行错误

span标签不能包含div标签

### 语法错误

syntax error

### 系统无法找到指定资源

请求某个资源URL，长度超过IE的限制2083个字符。



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
