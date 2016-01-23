title: "读《JavaScript高级程序设计》总结14"
date: 2016-01-01 07:25:35
description: "读《JavaScript高级程序设计 -- 第14章 表单脚本》主要介绍文本框表单,选择框表单,表单序列化,富文本编辑器"
tags:
- "JavaScript"
---

## 表单的基础知识

在JS中,表单对应的是HTMLFormElement类型,继承HTMLElement类型,独有属性和方法:

1. acceptChart: 服务器能够处理的字符串,等价于HTML中的accept-charset特性
2. action: 接收请求的URL .action
3. elements: 表单的所有控件.
4. enctype: 请求编码类型. enctype
5. length: 表单控件数量.
6. method: get or post. method
7. name: 表单的民称. name
8. reset(): 重置所有表单域为默认值
9. submit(): 提交表单.
10. target: 用于发送请求和接收响应的窗口名称.

访问方式:

```js
var form = document.getElementById('form1');
var firstForm = document.forms[0];
var myForm =  document.forms['form2']; // 根据页面中的表单名称name访问
```

> 提交: form.submit();

解决表单重复提交:
1. 第一次提交表单后禁用提交按钮.
2. 利用onsubmit事件处理程序取消后连续表单操作

> 重置: form.reset(); 不推荐

### 表单字段

- 共有的表单字段属性

除`<fieldset>`元素之外,

1. disabled: 布尔值.表示当前字段是否被禁用
2. form.
3. name: 当前字段名称.
4. readOnly: 布尔值.字段只读.
5. tabIndex: 当前字段切换序号.
6. type: 字段类型, checkbox, radio
7. value: 字段将被提交给服务器的值.

- 共有方法

`focus()`获得焦点; `blur()`移走焦点; 
`change()`对于input和textarea元素,在他们失去焦点且value值改变时触发.对于select元素,在于选项改变时触发.

## 文本框脚本

`input`与`textarea`.

- input: type='text', size="25", value='initial value', maxlength='50'

显示25个字符,但输入不能超过50个字符.

- textarea: rows='25', cols='5'

不能在HTML中给textarea中指定最大字符串.

### 选择文本

上述两种文本框都支持`select()`方法.用于选择文本框中的所有文本.

### 过滤输入

1. 屏蔽字符. 调用keypress检测对应的键盘值.不符合调用preventDefault().从而屏蔽按键事件.
2. 操作剪切板
3. beforecopy,copt, beforecut, cut, paste.

### 自动切换焦点

```plain
<input type="text" name="tel1" id="txtTel1" maxlength="3">
<input type="text" name="tel2" id="txtTel2" maxlength="3">
<input type="text" name="tel3" id="txtTel3" maxlength="4">
```

> 为了增强易用性,同时加快数据输入,可以在前一个文本框中的字符串达到最大数量后,自动将焦点切换到下一个文本框中.
> 自动切换焦点.(`淘宝的支付宝密码就是这样`)


```js
(function(){
       
  function tabForward(event){            
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);
    
    if (target.value.length == target.maxLength){
      var form = target.form;
      
      for (var i=0, len=form.elements.length; i < len; i++) {
        if (form.elements[i] == target) {
          if (form.elements[i+1]){
            form.elements[i+1].focus();
          }
          return;
        }
      }
    }
  }
              
  var textbox1 = document.getElementById("txtTel1"),
      textbox2 = document.getElementById("txtTel2"),
      textbox3 = document.getElementById("txtTel3");
  
  EventUtil.addHandler(textbox1, "keyup", tabForward);        
  EventUtil.addHandler(textbox2, "keyup", tabForward);        
  EventUtil.addHandler(textbox3, "keyup", tabForward);        
                
})();
```

### HTML5约束验证API

1. 必填字段: required
2. 其他输入类型: email, url, 不支持的浏览器会转化为text.
3. 数值范围:  number, range,datetime, datetime-local,date,month, week,time.对于以上字段可以设置: min, max, step,name属性
4.正则表达式属性: pattern,开头和结尾不能加^和$
5. 检测有效性`checkValidity()`. 
6. 禁用验证,通过在form标签上设置`novalidate`属性.告诉表单不用进行验证.js通过`noValidate`访问.

### 选择框脚本

略...

### 表单序列化

需要自己写控件的必看.

### 富文本编辑

WYSIWYG(What You  See Is What You Get, 所见即所得). 
1. 借助`iframe`.
2. `contentEditable`属性

### 富文本与表单

将iframe中的表单字段利用在`外设置隐藏的表单字段`对应.通过提交onsubmit事件处理程序绑定数据.模拟提交外部表单.



-----------------------

> 参考文档: [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)
> 文章若有纰漏，请大家补充指正.
> 仅作为学习交流所用,不与商用.违者必究.
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
