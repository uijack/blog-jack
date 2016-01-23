title: "读《JavaScript高级程序设计》总结21"
date: 2016-01-04 21:51:35
description: "读《JavaScript高级程序设计 -- 第19章 Ajax 与 Comet》介绍XMLHttpRequest,进度事件,跨源资源共享,其他跨域技术"
tags:
- "JavaScript"
---


## XMLHttpRequest对象

Ajax(Asynchronous JavaScript and XML)技术的核心是XMLHttpRequest对象(简称XHR).
IE中的三个不同版本XHR对象,`MSXML2.XMLHttp`,`MSXML2.XMLHttp.3.0`,`MSXML2.XMLHttp.6.0`

适用于IE7之前的版本:

```js
function createXHR() {
  if (typeof arguments.callee.activeXString != 'string') {
    var versions = ['MSXML2.XMLHttp.6.0', 'MSXML2.XMLHttp.3.0', 'MSXML2.XMLHttp'],
      i, len;
    for (i=0, len=versions.length; i < len; i++) {
      try {
        new ActiveXObject(versions[i]);
        arguments.callee.activeXString = versions[i];
        break;
      } catch (ex) {
        // 跳过
      }
    }
  }
  return new ActiveXObject(arguments.callee.activeXString);
}
```

此函数会更具IE中可用的MSXML库的情况创建最新版本的XHR对象.
IE7+,Firefox,Opera,Chrome,Safari都支持原生的XHR对象,使用XMLHttpRequest构造函数

var xhr = new XMLHttpRequest();

兼容模式:

```js
function createXHR(){
  if (typeof XMLHttpRequest != "undefined"){
    return new XMLHttpRequest();
  } else if (typeof ActiveXObject != "undefined"){
    if (typeof arguments.callee.activeXString != "string"){
      var versions = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0",
                      "MSXML2.XMLHttp"],
          i, len;

      for (i=0,len=versions.length; i < len; i++){
        try {
          new ActiveXObject(versions[i]);
          arguments.callee.activeXString = versions[i];
          break;
        } catch (ex){
          //skip
        }
      }
    }

    return new ActiveXObject(arguments.callee.activeXString);
  } else {
    throw new Error("No XHR object available.");
  }
}
        
var xhr = createXHR();        
xhr.open("get", "example.txt", false);
xhr.send(null);

if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
    alert(xhr.statusText);
    alert(xhr.responseText);
} else {
    alert("Request was unsuccessful: " + xhr.status);
}
```

### XHR用法

第一个要调用的方法就是`open('get', 'example.txt', false)`,接收3个参数
1,请求方式,
2,URL相当于执行代码的当前页面(相对或者绝对路径).
3.调用open()并不会真正发送请求,只是启动一个请求状态以备发送.
4.第三个参数表示同步还是异步.默认是异步false.

xhr.send(null);

send()接收一个参数,没有发送数据则为null,必须要.

服务器执行响应后会自动填充xhr对象的属性:

> responseText: 返回为本
> responseXML: 响应类型是'text/xml'或'application/xml'.
> status: 响应的HTTP状态
> statusText: HTTP状态说明

在接收到响应后,第一步检查status属性,一般来说为200.304表示从缓存里面取数据.
也可以检测XHR对象的readyState属性:

0:未初始化.尚未调用open().
1:启动,已调用open(),未调用send()
2.发送,已调用send(),但未接到响应
3.接收,已接受部分响应数据
4.完成,已接收到全部数据,而且可以在客户端使用了.

只要readyState属性的值由一个变成另一个,都会触发一次readystatechange事件.可以利用这个时间来检测状态变化后的readyStaete的值.

```js
var xhr = createXHR();        
xhr.onreadystatechange = function(event){
  if (xhr.readyState == 4){
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
      alert(xhr.responseText);
    } else {
      alert("Request was unsuccessful: " + xhr.status);
    }
  }
};
xhr.open("get", "example.txt", true);
xhr.send(null);
```

在接收到响应之前还可以调用`abort()`方法来取消异步请求

xhr.abort();

调用之后应该对XHR对象进行解引用操作,由于内存原因,不建议重用XHR对象.

## XMLHttpRequest2级

定义了FormData类型:

var data = new FormData();
data.append('name', 'Jack');

可以将data直接传递给XHR对象的send()方法.

### 超时设置

IE8为XHR对象添加了一个timeout属性,表示请求在等待响应时间多少毫秒之后就停止.
规定时间没响应就调用`ontimeout`事件处理程序.

### overrideMimeType()方法

调用overrideMimeType()方法的时候,必须在send()方法之前.

## 进度事件

## 跨源资源共享

附加一个额外的Origin头部,其中包含请求页面的源信息(协议,域名,端口)
`Origin: http://www.uijack.com`
如果服务器认为这个请求可以接收,就在`Access-Control-Allow-Origin`头部中回发相同的源信息
`Access-Control-Allow-Origin: http://www.uijack.com`.

## 其他跨域技术

大部分采用绝对地址;

## 安全

### JSONP

JSON with padding:和JSON差不多,只不过是被包含在函数调用的中的JSON,如:
callback({"name": "Jack"});

JSONP由两部分组: 回调函数和数据:

http://www.uijack.com/json/?callback=handleResponse



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
