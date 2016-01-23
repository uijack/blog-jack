title: "读《JavaScript高级程序设计》总结23"
date: 2015-12-29 19:47:27
description: "读《JavaScript高级程序设计 -- 第23章 离线应用及客户端存储》主要讲述离线检测,应用缓存,以及数据存储(Cookie,Web,IndexedDB)"
tags:
- "JavaScript"
---

## 离线检测

支持离线Web应用是H5的另一重点,一般步骤:
1, 首先确保应用知道设备是否能上网.
2, 应用还必须能访问一定的资源(CSS, JS, 图片).
3, 必须有一块本地空间用于保存数据.无论能否上网都不妨碍读写.

- 识别设备在线还是离线

> navigator.onLine 属性, true表示设备能上网,false表示设备离线.因浏览器存在差异.

1, IE 6+和Safari 5+能正确检测网络断开,并返回false.
2, Firefox 3+和Opera 10.6+ 支持`navigator.onLine`属性,但是必须手工选中菜单项'文件->Web开发人员设置->脱机工作'才能工作.
3, Chrome 11及之前版本始终将`navigator.onLine`设置为true,这是待修复bug(2011年10月修复).

由于存在上述兼容性问题,单独使用上面方法不能确定是否连通,即便如此,在请求发生错误的情况下,检测这个属性任然管用.

> H5还提供了online和offline事件.在window对象上触发.

```js
  <p>You are currently: <span id="status"><script>document.write(navigator.onLine ? "Online" : "Offline");</script></span></p>
  <p>To change your status, click File -&gt; Work Offline. In supporting browsers ,the change is detected via events and the status above should change.</p>
  <script>
    EventUtil.addHandler(window, "online", function(){
        document.getElementById("status").innerHTML = "Online";
    });
    EventUtil.addHandler(window, "offline", function(){
        document.getElementById("status").innerHTML = "Offline";
    });
  </script>
```
为了检测应用是否离线,在页面加载后,最好先通过`navigator.onLine`取得初始状态,
然后通过两个事件来确定网络的连接状态是否变化.当上述事件触发时,navigator.onLine属性值也会改变.

## 应用缓存

HTML5的应用缓存(application cache),专为开发离线Web应用设计,从浏览器的缓存中分出一块缓存区,
可以使用一个描述文件,列出要下载和缓存的资源

```plain
CACHE MANIFEST
#Comment

file.js
file.css
```

要将描述文件与页面关联起来,可以再`<html>`中的manifest属性中指定这个文件的路径如:
`<html manifest="/offline.manifest">`这句代码告诉页面,`/offline.manifest`中包含着描述文件.
这个文件的MIME类型必须是`text/cache-manifest`.现在推荐`text/cache-appcache`.

> applicationCache对象,有一个属性status属性,值下面常量.表示当前缓存状态

![应用缓存](/img/js-high-level-23-1.png)

## 数据缓存

### Cookie

- 限制:

1. IE6以及低版本限制每个域名50个cookie
2. IE7之后最多50个
3. Firefox限制每个域最多50个
4. Opera限制30个
5. Safari和Chrome对cookie数量没有限制.

超出以后会清除掉原来的cookie.cookie长度最好限制在4095B以内.

- cookie的构成

1. 名称: 一个唯一确定的cookie名称, 不区分大小写.但是写的时候最好区分.cookie的名称必须是经过URL编码的.
2. 值: 存储在cookie中的字符串值.值必须呗URL编码.
3. 域: cookie对那个域有效. domain
4. 路径: 
5. 失效数据: 被删除的时间戳.
6. 安全标志: 指定后,cookie只有在使用SSL链接的时候才发送到服务器.

- JavaScript设置cookie

CookieUtil.js

```js
var CookieUtil = {

  get: function (name){
    var cookieName = encodeURIComponent(name) + "=",
        cookieStart = document.cookie.indexOf(cookieName),
        cookieValue = null,
        cookieEnd;
        
    if (cookieStart > -1){
        cookieEnd = document.cookie.indexOf(";", cookieStart);
        if (cookieEnd == -1){
            cookieEnd = document.cookie.length;
        }
        cookieValue = decodeURIComponent(document.cookie.substring(cookieStart + cookieName.length, cookieEnd));
    } 

    return cookieValue;
  },
    
  set: function (name, value, expires, path, domain, secure) {
    var cookieText = encodeURIComponent(name) + "=" + encodeURIComponent(value);

    if (expires instanceof Date) {
        cookieText += "; expires=" + expires.toGMTString();
    }

    if (path) {
        cookieText += "; path=" + path;
    }

    if (domain) {
        cookieText += "; domain=" + domain;
    }

    if (secure) {
        cookieText += "; secure";
    }

    document.cookie = cookieText;
  },
  
  unset: function (name, path, domain, secure){
    this.set(name, "", new Date(0), path, domain, secure);
  }
};
```

- 子cookie

详情参考PDF

### IE用户数据

并不安全,不能存放敏感信息

### Web存储机制

- sessionStroage
- globalStroage
- locationStroage

### IndexedDB

数据缓存不会加密.不要存储敏感信息.



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
