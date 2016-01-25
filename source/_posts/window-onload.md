title: "window.onload与jQuery $(documen).ready()"
date: 2016-01-25 19:53:31
description: "由ready引出的rem问题"
tags:
- JavaScript
- Keng
---
## 前言

网页中的`javascript脚本代码`往往需要在文档加载完成后才能够去执行,否则,可能导致无法获取对象的情况,为了避免这种情况的发生,可以使用以下两种方式:

> 1. 将脚本代码放在网页的底端,这样可以保证操作对象已经加载完成.
> 1. 通过window.onload来执行脚本代码.


### jQuery绑定ready()方法判定

```js
function completed() {
  // readyState === "complete" 在老版本IE上适用
  if ( document.addEventListener || event.type === "load" || document.readyState === "complete" ) {
    detach();
    jQuery.ready();
  }
}

if ( document.readyState === "complete" ) {
  // Handle it asynchronously to allow scripts the opportunity to delay ready
  setTimeout( jQuery.ready );

  // 标准浏览器支持DOMContentLoaded事件
} else if ( document.addEventListener ) {
  // 绑定DOMContentLoaded事件和响应函数，响应函数会解决延时
  document.addEventListener( "DOMContentLoaded", completed, false );

  // 回退到window.onload事件绑定，所有的浏览器都支持
  window.addEventListener( "load", completed, false );

// 如果是IE事件模型
} else {
  // 确保在onload之前执行延时，可能时间比较迟，但是对于iframes来说比较安全
  document.attachEvent( "onreadystatechange", completed );

  // 回退到window.onload事件绑定，所有的浏览器都支持
  window.attachEvent( "onload", completed );

  // 如果IE并且不是一个frame,不断地检查，看是否该文件已准备就绪
  var top = false;

  try {
    top = window.frameElement == null && document.documentElement;
  } catch(e) {}

  if ( top && top.doScroll ) {
    (function doScrollCheck() {
      if ( !jQuery.isReady ) {

        try {
          // Use the trick by Diego Perini
          // http://javascript.nwbox.com/IEContentLoaded/
          top.doScroll("left");
        } catch(e) {
          return setTimeout( doScrollCheck, 50 );
        }

        // 移除之前绑定的事件
        detach();

        // 执行延迟
        jQuery.ready();
      }
    })();
  }
}
```
一旦监听到文档准备完成，则调用jQuery.ready执行延时对象的成功回调列表：即所有通过jQuery.ready(fn)[或jQuery(fn)]方式添加的函数fn。


### 文档加载状态

document.readyState用来判断文档加载状态,是一个只读属性,可能的值有:

```plain
0-uninitialized：XML 对象被产生，但没有任何文件被加载。
1-loading：加载程序进行中，但文件尚未开始解析。
2-loaded：部分的文件已经加载且进行解析，但对象模型尚未生效。
3-interactive：仅对已加载的部分文件有效，在此情况下，对象模型是有效但只读的。
4-complete：文件已完全加载，代表加载成功。
```

但是,老版本的Firefox并不支持document.readyState[最新的Firefox已经支持了].所以想要兼容所有浏览器监听文档准备完成分两种情况来处理:
　　
- 标准浏览器使用addEventListener添加DOMContentLoaded和load监听,任何一个事件被触发即可
- 老版本IE浏览器使用attachEvent添加onreadystatechange和onload来监听,任何一个被触发,并且onreadystatechange时document.readyState === "complete"即可.

window.onload是一个事件,当文档加载完成之后就会触发该事件.可以为其注册事件处理函数.将要执行的代码放在事件处理函数中

## 对比window.onload与$(documen).ready()

### 执行时间

1. `window.onload`必须等到页面(包括图片)内所有元素加载完毕后才执行.
1. `$(document).ready()`是DOM结构绘制完毕后就执行,`不必等到加载完毕`.

以上说明ready要比onload先执行.

### 编写数量

1. `window.onload`不能同时编写多个,如果给window.onload()绑定多个函数,只会执行最后一个.
1. `$(document).ready()`可以编写多个,并且都可以执行.

### 简写方式

`window.onload`没有简化写法,$(scument).ready(function(){})可以简写成`$(function(){});

## AngularJS遇到的Keng

采用AngularJS + inoci 做H5 app,如果采用rem来布局,当计算设备clientWidth的时候,动态给html标签设置font-size,不要把该逻辑放在angular的run()中,否则会导致所有页面内容挤在一起,因为在执行run()的时候页面已经差不多渲染完了.此时才设置正确的font-size参考值.等浏览器下一次重排的时候颜面才渲染正常.为了正常显示,可以在首页的script标签里面`单独设置`.

```js
<script type="text/javascript">
  var deviceWidth = document.documentElement.clientWidth;
  document.documentElement.style.fontSize = deviceWidth / 6.4 + 'px';
</script>
```

这样就不会导致所有渲染的页面因为font-size参考错误而挤在一起.
