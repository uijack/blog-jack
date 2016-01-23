title: "读《JavaScript高级程序设计》总结9"
date: 2015-12-28 18:22:25
description: "读《JavaScript高级程序设计 -- 第8章 客户端检测》主要介绍能力测试,用户代理检测"
tags:
- "JavaScript"
---

## 能力检测

迄今为止,客户端检测仍是Web开发领域中一个饱受争议的话题.因为浏览器之间的"怪癖(quirk)",不到万不得已不要使用,应该找最通用的方法.
`能力测试`又称`特性测试`.能力测试的目标不是识别特定的浏览器,而是识别浏览器的能力.确定浏览器支持特定的能力.

### 更可靠的能力检测

确定一个对象是否支持排序,
```js
// 不要这样做,这不是能力测试,只检测了是否存在相应的方法
fucntion isSortable(obj) {
  return !!object.sort;
}

// 因为下面这个也会返回true
var result = isSortable({sort: true});

// 这样更好: 检查sort是不是函数
fucntion isSortable( obj ) {
  return typeof object.sort == 'function';
}
```

## 怪癖检测

`怪癖检测`目标是识别浏览器的特殊行为.想要知道浏览器存在什么缺陷(`bug`).

## 用户代理检测

## 用户代理字符串检测技术

- 识别呈现引擎
- 识别浏览器
- 识别平台
- 识别Windows操作系统
- 识别移动设备
- 识别游戏系统

### 完整的代码

以下是完整的用户代理字符串检测脚本,包括检测呈现`引擎`,`平台`,`Windows操作系统`,`移动设备`和`游戏系统`.

```js
var client = function() {

  //rendering engines 呈现引擎
  var engine = {            
    ie: 0,
    gecko: 0,
    webkit: 0,
    khtml: 0,
    opera: 0,

    //complete version 完整的版本号
    ver: null  
  };
  
  //browsers 浏览器
  var browser = {

    ie: 0,
    firefox: 0,
    safari: 0,
    konq: 0,
    opera: 0,
    chrome: 0,

    //specific version 特殊的版本号
    ver: null
  };

  
  //platform/device/OS 平台/设备/操作系统
  var system = {
    win: false,
    mac: false,
    x11: false,
    
    //mobile devices 移动设备
    iphone: false,
    ipod: false,
    ipad: false,
    ios: false,
    android: false,
    nokiaN: false,
    winMobile: false,
    
    //game systems 游戏系统
    wii: false,
    ps: false 
  };    

  //detect rendering engines/browsers 检测呈现引擎和浏览器
  var ua = navigator.userAgent;    
  if (window.opera){
    engine.ver = browser.ver = window.opera.version();
    engine.opera = browser.opera = parseFloat(engine.ver);
  } else if (/AppleWebKit\/(\S+)/.test(ua)){
    engine.ver = RegExp["$1"];
    engine.webkit = parseFloat(engine.ver);

    //figure out if it's Chrome or Safari 确定是Chrome还是Safari
    if (/Chrome\/(\S+)/.test(ua)){
      browser.ver = RegExp["$1"];
      browser.chrome = parseFloat(browser.ver);
    } else if (/Version\/(\S+)/.test(ua)){
      browser.ver = RegExp["$1"];
      browser.safari = parseFloat(browser.ver);
    } else {
        //approximate version 近似地确定版本号
      var safariVersion = 1;
      if (engine.webkit < 100){
          safariVersion = 1;
      } else if (engine.webkit < 312){
          safariVersion = 1.2;
      } else if (engine.webkit < 412){
          safariVersion = 1.3;
      } else {
          safariVersion = 2;
      }

      browser.safari = browser.ver = safariVersion;
    }
  } else if (/KHTML\/(\S+)/.test(ua) || /Konqueror\/([^;]+)/.test(ua)){
      engine.ver = browser.ver = RegExp["$1"];
      engine.khtml = browser.konq = parseFloat(engine.ver);
  } else if (/rv:([^\)]+)\) Gecko\/\d{8}/.test(ua)){
      engine.ver = RegExp["$1"];
      engine.gecko = parseFloat(engine.ver);

      //determine if it's Firefox 确定是不是Firefox
      if (/Firefox\/(\S+)/.test(ua)){
        browser.ver = RegExp["$1"];
        browser.firefox = parseFloat(browser.ver);
      }
  } else if (/MSIE ([^;]+)/.test(ua)){    
    engine.ver = browser.ver = RegExp["$1"];
    engine.ie = browser.ie = parseFloat(engine.ver);
  }

  //detect browsers 检测浏览器
  browser.ie = engine.ie;
  browser.opera = engine.opera;

  //detect platform 检测平台
  var p = navigator.platform;
  system.win = p.indexOf("Win") == 0;
  system.mac = p.indexOf("Mac") == 0;
  system.x11 = (p == "X11") || (p.indexOf("Linux") == 0);

  //detect windows operating systems 检测Windows操作系统
  if (system.win){
    if (/Win(?:dows )?([^do]{2})\s?(\d+\.\d+)?/.test(ua)){
      if (RegExp["$1"] == "NT"){
        switch(RegExp["$2"]){
          case "5.0":
            system.win = "2000";
            break;
          case "5.1":
            system.win = "XP";
            break;
          case "6.0":
            system.win = "Vista";
            break;
          case "6.1":
            system.win = "7";
            break;
          default:
            system.win = "NT";
            break;
        }
      } else if (RegExp["$1"] == "9x"){
        system.win = "ME";
      } else {
        system.win = RegExp["$1"];
      }
    }
  }

  //mobile devices  移动设备
  system.iphone = ua.indexOf("iPhone") > -1;
  system.ipod = ua.indexOf("iPod") > -1;
  system.ipad = ua.indexOf("iPad") > -1;
  system.nokiaN = ua.indexOf("NokiaN") > -1;

  //windows mobile 
  if (system.win == "CE"){
    system.winMobile = system.win;
  } else if (system.win == "Ph"){
    if(/Windows Phone OS (\d+.\d+)/.test(ua)){;
      system.win = "Phone";
      system.winMobile = parseFloat(RegExp["$1"]);
    }
  }

  //determine iOS version 检测IOS版本
  if (system.mac && ua.indexOf("Mobile") > -1){
    if (/CPU (?:iPhone )?OS (\d+_\d+)/.test(ua)){
      system.ios = parseFloat(RegExp.$1.replace("_", "."));
    } else {
      system.ios = 2;  //can't really detect - so guess 不能真正检测出来, 所以只能猜测
    }
  }

  //determine Android version 检测Android版本
  if (/Android (\d+\.\d+)/.test(ua)){
    system.android = parseFloat(RegExp.$1);
  }

  //gaming systems 游戏系统
  system.wii = ua.indexOf("Wii") > -1;
  system.ps = /playstation/i.test(ua);

  //return it 返回这些对象
  return {
    engine:   engine,
    browser:  browser,
    system:   system
  };
}();
```

### 使用方法

`用户代理是客户端检测的最后一个选择`,只要可用都应该优先采用`能力检测`和`怪癖检测`.



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
