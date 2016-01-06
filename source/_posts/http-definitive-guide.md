title: "《HTTP权威指南》随笔1"
date: 2016-01-06 19:27:53
description: "读《HTTP权威指南 -- 第1章 HTTP概述》主要有MIME"
tags:
- "HTTP"
- "随笔"
---


## 媒体类型

因特网上有数千种不同的类型数据,HTTP会给每种要通过Web传输的对象都打上名为`MIME类型(MIME type)`的数据标签格式.
最初设计MIME(Multipurpose Internet Mail Extension, 多用途因特网邮件扩展)是为了解决不同的电子邮件系统之间`搬移报文`是存在的问题.后来HTTP也采纳用来`描述并标记多媒体内容`.

MIME类型是一种文本标记,表示`一种主要的对象类型`和`一个特定的子类型`.中间由`一条斜杠`来分割.

```js
HTML格式文本: text/html
普通的ASCII文本文档: text/plain
JPEG版本图片: image/jpeg
GIFg格式图片: image/gif
Apple的QuickTime电影: video/quicktime
微软PowerPoint文件: application/vnd.ms-powerpoint
```

大多数浏览器都可以处理数百种常见的对象类型.
HTTP客户端和HTTP服务器共同构成了万维网的基本组件.`Web浏览器`是最常见的`HTTP客户端`
