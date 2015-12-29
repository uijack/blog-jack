title: "读《JavaScript高级程序设计》总结8"
date: 2015-12-28 06:35:38
description: "读《JavaScript高级程序设计 -- 第8章 BOM》介绍Window,location,navigator,screen,history"
tags:
- "JavaScript"
- "随笔"
---

## Window对象

### 窗口关系及框架

与frame相关.

- window
- top
- frames
- parent
- self

### 窗口位置

跨浏览器取得窗口左边和上边的位置

```js
// window.screenLeft  IE Safari Opera Chrome
// window.screenX  Firefox
var leftPos = (typeof window.screenLeft == 'number') ? window.screenLeft : window.screenX;
var topPos  = (typeof window.screenTop == 'number') ? window.screenTop : window.screenY;
```

window.moveTo(x, y); //将窗口移到(x,y)
window.moveBy(x, y); //将窗口移动(x,y)

### 窗口大小

取得页面视口的大小

```js
var pageWidth  = window.innerWidth,
    pageHeight = window.innerHeight;

if (typeof pageWidth != 'number') {
  if (document.compatMode == 'CSS1Compat') {  // 是否处于标准模式
    pageWidth  = document.documentElement.clientWidth;
    pageHeight = document.documentElement.clientHeight;
  } else {
    pageWidth  = document.body.clientWidth;
    pageHeight = document.body.clientHeight;
  }
}
```

- wondow.resizeTo(100, 100); // 移到
- window.resizeBy(100, 100); // x和y各移动多少

### 导航和打开窗口

- window.open();

可以接收4个参数: 要加载的url, 窗口目标, 一个特性字符创, 一个新页面是否取代浏览器历史记录当中当前加载页面的布尔值.
该方法返回一个指向新窗口的引用,和window相似

var worxWin = window.open(...);
worxWin.close(); // 关闭打开的窗口

### 间歇调用()和超时调用(setTimeout())

```js
var timeoutId = setTimeout(function() {}, 1000);
clearTimeout(timeoutId);

var stId = stInterval(function() {}, 1000);
clearInterval(stId);
```

### 系统对话框

- alert();
- confirm();
- prompt();

## location 对象

既是window对象的属性,也是document对象的属性.`window.location` 和 `document.location`引用的是同一个对象.

属性名:

1, hash: 返回URL中#号后跟的零或多个字符, 不包含则返回null
2, host: 返回服务器名称和端口
3, href: 返回当前加载页面完整的url
4, pathname: 返回URL中的目录或文件名
5, port: 端口
6, protocol: 协议http: 或https:
7, search: 返回URL的查询字符串,这个字符串以?号开头

- replace()

### 查询字符串参数

```js
function getQueryStringArgs() {
  // 取得查询字符串并去掉开头的问号
  var qs = (location.search.length > 0 ? location.search.substring(1) : '');

  // 保存数据的对象
  var args = {};

  // 取得每一项
  var items = qs.length ? qs.split('&') : [];
  var item  = null,
      name  = null,
      value = null,
      i     = 0,
      len   = items.length;

  // 逐个将每一项添加到args对象中
  for (i = 0; i < length; i++) {
    item  = items[i].split('=');
    name  = decodeURIComponent(item[0]);
    value = decodeURIComponent(item[1]);

    if (name.length) {
      args[name] = value;
    }
  }

  return args;
}
```

### 位置操作

location.assign('http://www.baidu.com'); // 打开百度

> 以下两句代码都会调用location.assign()方法.

window.location = 'http://www.baidu.com';
location.href = 'http://www.baidu.com';

- location.reload(); // 重新加载,有可能从缓存中读取
- location.reload(); // 重新加载,从服务器重新加载

## navigator 对象

常用属性:
appCodeName, appName, appVersion, language, onLine, mimeTypes, plugins, platform, `userAgent`, vendor.

### 检测插件

- 检测插件 IE中无效

```js
function hasPlugin( name ) {
  name = name.toLowerCase();
  var plugins = navigator.plugins.length || 0;
  for (var i = 0; i < plugins.length; i++) {
    if (plugins[i].name.toLowerCase().indexOf(name) > -1) {
      return true;
    }
  }
  return false;
}


// 检测Flash
alert(hasPlugin('Flash'));

// 检测QuickTime
alert(hasPlugin('QuickTime'));

```

- 检测IE中的插件

```js
function hasIEPlugin( name ) {
  try {
    new ActiveXObject(name);
    return true;
  } catch( ex ) {
    return false;
  }
}

// 检测Flash
alert(hasIEPlugin('ShockwaveFlash.ShockwaveFlash'));

// 检测QuickTime
alert(hasIEPlugin('QuickTime.QuickTime'));
```

- 检测所有浏览器的中的Flash

```js
function hasFlash() {
  var result = hasPlugin('Flash');
  if (!result) {
    result = hasIEPlugin('ShockwaveFlash.ShockwaveFlash');
  }

  return result;
}
```

plugins集合有个名叫做`refresh()`方法,用于刷新 plugins 以反映最新安装的插件.

### 注册程序

- registerContentHandler()
- registerProtocolHandler()

## screen 对象

详情请看PDF

## history 对象

保存用户上网的历史, 方法:

- history.go(-1); 后退一页
- history.go(1); 前进一页,穿那个数字就进几页 ,没有什么也不做.

可以用`history.back()`和`history.forward()`.

## 参考文档

- [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)

-----------------------

> 文章若有纰漏，请大家补充指正，谢谢！
> 如有疑问可留言，我会在第一时间回复。
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
