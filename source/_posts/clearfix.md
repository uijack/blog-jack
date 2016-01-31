title: "CSS清除浮动解决方案"
date: 2016-01-31 21:17:20
description: "常用3中清除浮动解决方案clearfix,overflow,inline-block."
tags:
- "css"
---


## 清除子元素和清除上级元素

清除浮动包括`清除子元素的浮动`和`清除上级元素的浮动`两种,其中清除上级上级元素的浮动,只需要设置`clear:both`就可以了. 而清除子元素的浮动则可以用`空标签法`,`clearfix方法`或者`overflow方法`.因清除上级元素的浮动比较简单,而空标签发法清除子元素浮动会增加额外标签,所以主要说:clearfix方法,overflow方法,以及inline-block方法.

## 为什么要清除浮动?

一个块级元素的高度如果没有设置`height`,那么其高度就由里面子元素来撑开,由于子元素使用浮动,脱离了标准文档流,那么父元素的高度会将其忽略.用firebug查看下如果不清除浮动,父元素会出现高度不够.父元素如果设置`border`或者`background`都得不到正确的解析.

## 清除子元素浮动clearfix方法

html:
```plain
<ul class="demo1 clearfix">
    <li><img src="campaign-2.png"></li>
    <li><img src="campaign-2.png"></li>
    <li><img src="campaign-2.png"></li>
</ul>
```
CSS简洁版:
```plain
<style type="text/css" media="screen">
    .demo1 {
      padding: 10px;
      background-color: #ccc;
      border: 1px solid #bbb;
      list-style: none;
    }

    .demo1 li {
      float: left;
      display: inline;
      margin-right: 10px;
      margin-left: 0;
    }

    .demo1 li img {
      width: 100px;
      height: 80px;
    }
    .clearfix:before, .clearfix:after {
      content: "";
      display: table;
    }
    .clearfix:after {
      clear: both;
      overflow: hidden;
    }
    .clearfix {
      zoom: 1;
    }
</style>
```
> clearfix的方法主要就是在浮动元素的父元素上加上一个clearfix class,然后这个父元素的框就会包括所有的浮动子元素.


CSS经典版:待验证
```plain
.clearfix:after {
    visibility: hidden;
    display: block;
    font-size: 0;
    content: " ";
    clear: both;
    height: 0;
}
* html .clearfix             { zoom: 1; } /* IE6 */
*:first-child+html .clearfix { zoom: 1; } /* IE7 */
```

## 清除子元素浮动overflow方法

```plain
.demo2 {
    overflow: auto; //或者 overflow: hidden
    *zoom: 1;
}
```
> 这种方法主要是对父元素设置css,所以不需要加个额外class

## 清除子元素浮动inline-block方法

完美兼容浏览器,结果还是有差别,父元素的宽度由于是`inline-block`正常表现,随子元素内容宽度增加而增加.

```plain
.demo4 {
    display: inline-block;
    *display: inline;
    *zoom: 1;
}
```
