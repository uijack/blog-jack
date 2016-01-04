title: "读《JavaScript高级程序设计》总结20"
date: 2016-01-04 20:10:02
description: "读《JavaScript高级程序设计 -- 第19章 JSON》介绍JSON语法，解析与序列化"
tags:
- "JavaScript"
- "随笔"
---


## 语法

JSON的语法可以表示以下三种类型的值：

1. 简单值：与JS相同语法，可以在json中表示字符串、数值、布尔值和null,但`不支持`特殊值`undefined`。
2. 对象： 复杂数据类型，有一组有序的键值对儿，每个值可以是简单值，也可以是复杂数据类型的值。
3. 数组： 数组也是一种复杂数据类型值，表示一组有序的值的列表。可以通过数值索引访问其中的值。值可以使任意类型，--简单值，对象或数组

### 简单值

JS字符串与JSON字符串的最大区别在于：JSON字符串必须使用`双引号`（单引号会导致语法错误）。
布尔值和null也是有效的JSON形式，但实际运用中，JSON更多的用来表示复杂的数据结构，而简单值只是一部分。

### 对象

JSON中的对象与JS字面量稍微有一些不同，`JSON中的对象要求给属性加双引号（不能是单引号）`，
`没有变量声明`，`末尾没有分号`，`同一对象中绝对不应该出现两个相同属性`。

### 数组

JS中的数组字面量：var values = [25, "hi", true];
JSON中：[25, "hi", true]
同样注意：`JSON`数组也没有变量和末尾分号。

## 解析与序列化

JSON流行原因：

1. 拥有与JS类似的语法。
2. 可以把JSON数据结构解析为有用的JavaScript对象。（最重要原因）

### JSON对象

早期的JSON解析器基本上使用JS的`eval()`函数，解析返回JS对象和数组。

JSON对象的两个方法：

> stringify()
> parse()

```js
var book = {
  title: "Professional JavaScript",
  authors: [
    "Nicholas C. Zakas"
  ],
  edition: 3,
  year: 2011
};

var jsonText = JSON.stringify(book);
alert(jsonText);
```
这个例子使用JSON.stringify()把一个JavaScript对象序列化成一个JSON字符串。然后将它保存在变量jsonText中，
默认情况下，JSON.stringify()输出的JSON字符串不包含任何空格字符和缩进。

利用JSON.parse()可以将JSON字符串转化为相应的JavaScript值。解析不是有效的JSON对象，会抛出异常。

### 序列化选项

- 过滤结果

如果参数是数组。那么JSON.stringify()的结果将只包含数组中列出的属性：

```js
var book = {
  title: "Professional JavaScript",
  authors: [
    "Nicholas C. Zakas"
  ],
  edition: 3,
  year: 2011
};

var jsonText = JSON.stringify(book, ["title", "edition"]);
alert(jsonText);
```

结果：{"title":"Professional JavaScript","edition":3}

> 如果第二个参数是函数，行为会稍有不同，传入的函数接受两个参数，属性(键)名和属性值。
> 属性名只能是字符串，而在值并非键值对结构的值时，键名可以是空字符串。

为了改变序列化对象的结果，函数返回的值就是相应键的值。如果返回`undefined`,那么相应的属性会被忽略。

```js
var book = {
  title: "Professional JavaScript",
  authors: [
      "Nicholas C. Zakas"
  ],
  edition: 3,
  year: 2011
};

var jsonText = JSON.stringify(book, function(key, value){
  switch(key){
    case "authors":
      return value.join(",")
     
    case "year":
      return 5000;
        
    case "edition":
      return undefined;
        
    default:
      return value;
  }
});
alert(jsonText);
```

结果为：{"title":"Professional JavaScript","authors":"Nicholas C. Zakas","year":5000}

- 字符串缩进

JSON.stringify()方法的`第三个参数`用于控制结果中的缩进和空白符.参数是一个`数值`,表示每个级别缩进的空格数
JSON.stringify()同时也在结果字符串中插入了换行符以提高可读性.第三个参数最大数值为`10`.超过会转化为10.
如果是字符串,则这个字符串将在JSON字符串中被用作缩进字符(不要使用空格),最好是制表符或者两个段划线之类的任意字符.

- toJSON()方法

JSON.stringify()不能满足对对象的自定义序列化需求,可以调用对象上的`toJSON()`方法,返回其自身的JSON数据格式.

```js
var book = {
  title: "Professional JavaScript",
  authors: [
      "Nicholas C. Zakas"
  ],
  edition: 3,
  year: 2011,
  toJSON: function(){
    return this.title;
  }
};
var jsonText = JSON.stringify(book); // "Professional JavaScript"
```

`toJSON()`方法可以作为函数过滤器的补充,
`序列化内部顺序:`
1. 如果存在toJSON()方法而且能通过它取的有效值,则调用该方法.否则,按默认顺序执行序列化.
2. 如果提供了第二个参数,应用这个函数过滤器.传入函数过滤器的值是第(1)步返回的值.
3. 对第(2)步返回的每个值经行相应的序列化.
4. 如果提供第三个参数,执行相应的格式化.

### 解析选项

JSON.parse()方法也可以接受另一个参数,该参数是函数.将在每个键值对儿上调用.这个函数被称为还原函数(reviver).
接受两个参数,一个键和一个值.而且都需要返回一个值.如果还原函数返回`undefined`,则表示要从结果中删除相应的键.
如果返回其他值,则将该值插入到结果中.

在将日期字符串转化为Date对象时,经常要用到还原函数.如:

```js
var book = {
  "title": "Professional JavaScript",
  "authors": [
    "Nicholas C. Zakas"
  ],
  edition: 3,
  year: 2011,
  releaseDate: new Date(2011, 11, 1)
};

var jsonText = JSON.stringify(book);
alert(jsonText);

var bookCopy = JSON.parse(jsonText, function(key, value){
  if (key == "releaseDate"){
    return new Date(value); // return undefined;
  } else {
    return value;
  }
});

// alert("releaseDate" in bookCopy);
alert(bookCopy.releaseDate.getFullYear());
```

以上代码先是为book对象新增releaseDate属性,该属性保存着一个Date对象.经过序列化之后变成有效JSON字符串,经过解析还原成Date对象, 才能基于这个对象调用getFullYear()方法.


-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
