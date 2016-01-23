title: "读《JavaScript高级程序设计》总结15"
date: 2016-01-02 11:01:02
description: "读《JavaScript高级程序设计 -- 第15章 使用Canvas绘图》主要介绍Canvas基本用法,上下文和WebGL"
tags:
- "JavaScript"
---

## 基本用法

支持: IE9+, Firefox1.5+, Safari2+, Opera9+, Chrome,IOS版Safari以及Android版Webkit某种程度上都支持.
要使用`canvas`,必须设置`width`和`height`属性,也可以通过CSS样式设置.不添加任何样式或者绘制,在页面中看不到该元素.

要在这块画布(canvas)上绘制,首先要取得`绘图上下文`.

```js
var canvas = document.getElementById('canvas');

// 确定浏览器支持canvas元素
if (canvas.getContext) {
  var cxt = canvas.getContext('2d'); //取得2d上下文对象.
  // 更多操作
}
```

导出在`<canvas>元素上绘制的图像.使用`toDataURL()`方法

```js
var canvas = document.getElementById('canvas');

// 取得图像的数据URL
var imgURL = canvas.toDataURL('image/png'); // 接受一个图像的MIME类型格式参数。不是上下文环境，是canvas对象的方法

// 显示图像
var image = document.createElement('img');
image.src = imgURL;
document.body.appendChild(image);
```

## 2D上下文环境

绘制坐标开始于左上角(0,0)处.width和height表示水平和垂直的两个方向上可用的像素数目.

### 填充和描边

- 填充: 使用指定样式(颜色, 渐变或图像)填充图形. `fillStyle`
- 描边: 只在图形的边缘划线.`strokeStyle`

`fillStyle` 和 `strokeStyle`属性的值可以是字符串、渐变对象、或者模式对象，默认值都是`#000000`。可以指定css颜色值的任何格式：颜色名，16进制码，rgb，rgba，hsl，hsla。

```js
var cxt = canvas.getContext('2d'); //取得2d上下文对象.
cxt.strokeStyle = 'red';
cxt.fillStyle = '#0000ff'; // 填充颜色
```

### 绘制矩形

使用方法：

> fillRect()
> strokeStyle()
> clearRect()

都接受四个参数：x坐标, y坐标, 举行宽，举行高。

```js
var cxt = canvas.getContext('2d');

// 绘制红色矩形
cxt.fillStyle = '#ff0000';
cxt.fillRect(10, 10, 50, 50);

// 绘制半透明的蓝色矩形
cxt.fillStyle = 'rgba(0,0,255,0.5)';
cxt.fillRect(30, 30, 50, 50);

// 绘制红色描边矩形
cxt.strokeStyle = '#ff0000';
cxt.strokeRect(10, 10, 50, 50);

// 绘制半透明的蓝色描边矩形
cxt.strokeStyle = 'rgba(0,0,255,0.5)';
cxt.strokeRect(30, 30, 50, 50);
```

`clearRect()`方法用于`清除画布上的矩形区域`。本质上，这个方法可用把绘制的上下文中的某一矩形区域变透明。
在上面的代码添加：

```js
cxt.clearRect(40, 40, 10, 10);
```
在两个矩形重叠的地方清除一个小矩形。

### 绘制路径

`beginPath()`：表示要开始绘制路径。绘制方法如下：

```plain
1. arc(x, y, radius, startAngle, endAngle, countercolorwise): 以(x,y)为圆心，radius为半径，其实和结束角度（用弧度表示），绘制一条弧度，contercolorwise表示是否按逆时针方向计算，为false表示`按顺时针`方向计算。

2. arcTo(x1, y1, x2, y2, radius): 从上一点开始绘制一条弧线，到（x2, y2）为止。并以给定的半径穿过（x1,y1）.

3. bezierCurveTo(clx, cly, c2x, c2t, x, y); 从上一点开始绘制一条曲线，到（x,y）为止。并且以(c1x, c1y)和(c2x, c2y)为控制点。

4. lineTo(x, y); 从上一点绘制一条直线到（x,y）为止.
5. moveTo(x,y); 将绘图游标移动到（x,y）,不划线。
6. quadrationCurveTo(cx, cy, x, y): 从上一点开始绘制一条曲线，到（x,y),并且以（cx,cy）作为控制点。
7. rect(x, y, width, hright): 从点（x, y）绘制一个width,height的矩形。绘制的是矩形路径，而不是strokeRect()和fillRect()所绘制的独立的形状。
```

> closePath()
> fill()
> stroke()
> clip(): 在路径上创建一个剪切区域

### 绘制文本

> fillText()
> strokeText()

上面两个方法都有以下3个基础属性： 

> font: '10px Arial'.
> textAlign: start， end, left， right, center。 推荐用（start和end,而不是left,right）
> textBaseline: top, hanging, middle, alphabetic, ideographic, bottom.

### 变换

> rotate(angle) : 围绕圆点旋转图像angle弧度。
> scale(scaleX, scaleY): 缩放图像，在x轴方向乘以scaleX,y轴乘以scaleY,默认为1.0
> translate(x,y): 将坐标圆点移动到(x, y).执行这个变换过后，坐标(0,0)会变成(x,y).
> transform(m1_1, m1_2, m2_1, m2_2, dx, dy);
> setTransform(m1_1, m1_2, m2_1, m2_2, dx, dy);

### 绘制图像

将一副图像绘制到画布上。

> drawImage()

第一种：
var img = document.images[0];
cxt.drawImage(img, 10, 10); //  起点坐标(10,10), 大小为图片原始大小

第二种：
cxt.drawImage(img, 10, 10, 20, 30); // 图片大小为20*30像素

第三种：9个参数：
要绘制的图像，源图像x坐标，源图像y坐标，源图像width，源图像height，目标图像x坐标，目标图像y坐标，目标图像width，目标图像height.

```js
cxt.drawImage(img, 0, 10, 50, 50, 1, 100, 40, 60);
```
将原始图像的一部分绘制到画布上，这部分起点为(0,10)，宽度高度都为50px, 最终绘制到图像起点(0, 100),大小为40*60px.

第一个参数也可以接收另一个`<canvas>`元素作为第一个参数。将另外一个画布内容绘制到当前画布上。

### 阴影

2D上下文会根据以下几个属性，自动为形状和路径绘制出阴影。

> shadowColoe:
> shadowOffsetX:
> shadowOffsetY:
> shadowBlur: 模糊的像素数，默认0，既不模糊。

### 渐变

> createLinearGradient(起点x,起点y, 终点x, 终点y);// 返回一个CanvasGradient对象的实例

对象的方法：

> addColorStop(色标位置, css颜色值)

### 模式

模式就是重复的图像，可以用来填充和描边图形，调用：

> createPattern(图像对象， background-repeat属性值形同)；

background-repeat： repaet, repeat-x, repeat-y, no-repeat.

### 使用图像数据

> getImageData()

### 合成

`globalAlpha`和`globalComposition-Operation`

## WebGL

针对Canvas的3D上下文，不是W3C指定的标准

### 类型化数组

### WebGL上下文

```js
var canvas = document.getElementById('canvas');

var gl = canvas.getContext('experimental-webgl');
```
具体内容略。。。

### 支持

Firefox4+ 和Chrome都实现了WebGL API. Safari5.1也实现了，但默认是禁用。
计算机并须满足一点显卡要求。还在开发，不适合真正开发和应用。



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
