title: "读《JavaScript高级程序设计》总结6"
date: 2015-12-26 20:47:31
description: "读《JavaScript高级程序设计 -- 第6章 面向对象的程序设计》总结,详细介绍对象属性特征、创建对象以及继承等"
tags:
- "js.Property"
- "js.window"
- "随笔"
---

## 理解对象

面向对象(Object-Oriented, OO)

### 属性类型

ECMAScript第五版定义了只有内部才用的特性(attribute),描述属性的各种特征.
分为两种:`数据属性`和`访问器属性`

- 数据属性有四个值

> Configurable: 是否可以通过`delete`删除属性从而重新定义,能否修改属性的特性,能否把属性修改为访问器属性,默认值为true.
> Enumerable: 能否通过for-in循环返回属性,默认为true.
> Writable: 能否修改属性的值,默认为true.
> Value: 包含这个属性的数据值.去读属性从这个位置读,写属性值的时候把新值保存在这个位置,默认为undefined.

要修改属性默认的特性值,通过ECMAScript 5的Object.defineProperty(属性所在的对象, 属性的名字, 描述符对象)方法.描述符对象的属性必须是configurable,enumerable,writable,value中的一个或多个值.

以上四个属性设置为false,在非严格模式下修改将忽略,严格模式下会抛出错误.

```js
var person = {};  // var p = { name: 'Jack'};  Value特性将被设置为'Jack'
Object.defineProperty(person, 'name', {
  writable: false,
  value: 'Jack'
});

console.log(person.name); // 'Jack'
person.name = 'Chan';
console.log(person.name); // 'Jack'
```

`一旦将configurable设置为false,就不能在把它变回可配置,此时再调用Object.defineProperty()修改除writable之外的特性,都会导致错误.`
在调用Object.defineProperty()时不设置除Value之外的三个值,默认都为false.


- 访问器属性

访问器属性不包含数据值,它们是一对getter和setter函数,也包含四个特性:

`Configurable`: 同上,默认值true
`Enumerable`: 同上,默认为itrue
'Get': 在读取属性时调用,默认值undefined
'set': 在写入属性时调用,默认值undefined

同样通过Object.defineProperty(属性所在的对象, 属性的名字, 描述符对象)方法.

```js
var book = {
  _year: 2004,  // _year前面的下划线是一种记号,只能通过对象方法访问的属性
  edition: 1
};

Object.defineProperty(book, 'year', {
  get: function() {
    reruen this._year;
  },
  set: function(newValue) {
    if (newValue > 2004) {
      this._year = newValue;
      this.edition += newValue - 2004;
    }
  }
});

book.year = 2005;
console.log(book.edition); // 2
```

get和set只设置一个,访问另一个会忽略或抛错. 在defineProperty之前使用非标准方法
`__defineGetter__()`和`__defineSetter__()`,由Firefox引入,其他浏览器也给出相应实现.

```js
var book = {
  _year: 2004,  // _year前面的下划线是一种记号,只能通过对象方法访问的属性
  edition: 1
};
// 定义访问器的旧有方法
book.__defineSetter__('year', function() {
  return this._year;
});
book.__defineGetter__('year', function(newValue) {
  if (newValue > 2004) {
    this._year = newValue;
    this.edition += newValue - 2004;
  }
});

book.year = 2005;
console.log(book.edition); // 2
```

在不支持Object.defineProperty()的浏览器中不能修改`Configurable`和`Enumerable`.

### 定义多个属性

```js
var book = {};

Object.defineProperty(book, {
  _year: {
    value: 2004
  },

  edition: {
    value: 1
  },

  year: {
    get: function() {
      return this._year;
    },

    set: function(newValue) {
      if (newValue > 2004) {
        this._year = newValue;
        this.edition += newValue - 2004;
      }
    }
  }
});
```

以上代码定义了两个属性(_year, edition)和一个访问器(year)

### 读取属性的特性

ECMAScript 5 的Object.getOwnPropertyDescription(属性所在对象, 描述符的属性名称),返回值是一个对象.

根据以上代码:
var description = Object.getOwnPropertyDescription(book, '_year');
alert(description.value); //2004
alert(description.configurable); //false 原因看上面声明.
alert(typeof description.get);  // 'undefined'

var description = Object.getOwnPropertyDescription(book, 'year');
alert(description.value); //undefined
alert(description.configurable); //false 原因看上面声明.
alert(typeof description.get);  // 'function'

## 创建对象

`Object构造函数`或者`对象字面量`都可以创建单个对象. `缺点:同一接口重复创建对象,会产生大量重复代码`.
为了解决问题,使用工厂模式的一种变体.

### 工厂模式

一种函数, 用拉封装以特定接口创建函数对象的细节.

```js
function createPerson(name, age, job) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;

  o.sayName = function() {
    alert(this.name);
  };

  return o;
}

var p1 = new createPerson('Jack', 24, 'JS');
var p2 = new createPerson('Tonny', 29, 'Java');
```

`工厂模式`虽然解决了创建多个相似对象的问题, 但是没有解决`对象识别(即怎样知道一个对象的类型)`.随着发展,又出现了新模式.

### 构造函数模式

```js
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;

  this.sayName = function() {
    alert(this.name);
  };
}

var p1 = new Person('Jack', 24, 'JS');
var p2 = new Person('Tonny', 29, 'Java');

p1.constructor == Person // true
p2.constructor == Person // true
```

要创建Person对象, 必须使用new操作符,经历一下几个步骤:
1, 创建新对象
2, 将构造函数的作用域赋值给新对象(因此this就指向这个新对象)
3, 执行构造函数中的代码(为新属性添加)
4, 返回新对象

对象的constructor属性最初用来标识对象类型的,检测对象类型还是用typeof操作符更可靠.
p1 instanceof == Person // true
p2 instanceof == Person // true
p1 instanceof == Object // true
p2 instanceof == Object // true

- 将构造函数当做函数

```js
// 当做构造函数使用
var person = new Person('Jack', 24, 'JS');
person.sayName(); // 'Jack'

// 作为普通函数调用
Person('AAA', 23, 'Doctor'); // 添加到window上
window.sayName(); // 'AAA'

// 在另一个对象的作用域下使用
var o = new Object();
Person.call(o, 'Chan', 20, 'Java');
o.sayName(); // 'Chan'
```

- 构造函数的问题

`缺点: 就是每个方法都要在每个实例上重新创建一遍.`

与`构造函数模式`等价代码
```js
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;

  this.sayName = new Function('alert(this.name'); // 与声明函数的逻辑上是等价的.
}
```

这种方式创建函数,会导致不同作用域链和标识符解析.但创建Function新实例的机制仍然是相同的,
因此,不同实例上的同名函数实不相等的
p1.sayName == p2.sayName; // false

缺点: 创建两个完成同样任务的Function实例的确没必要.况且有this对象在, 根本不用再执行代码前就把函数绑定到特定对象上.将sayName函数定义转移到构造函数外.
因此引出`原型模式property`

### 原型模式property

每个创建函数都有一个property属性,是一个指针,指向一个对象.

`原型对象好处:`: 可以让`所有对象实例共享`它所包含的`属性`和`方法`.即不必在构造函数中定义对象实例的信息,而是直接添加到原型对象上.

```js
function Person() {};
Person.property.name = 'Jack';
Person.property.age = 24;
Person.property.job = 'JS;
Person.property.sayName = function() {
  console.log(this.name);
};

var p1 = new Peroson();
p1.sayName(); // 'Jack'
var p2 = new Person();
p2.sayName(); 'Jack'

p1.sayName == p2.sayName; // true
```

- 理解原型对象

![理解原型对象1](/img/js-high-level-6-1.png)
![理解原型对象2](/img/js-high-level-6-2.png)

> isPropertyOf()
> Object.getPropertyOf()
> hasOwnProperty()
> Object.getOwnPropertyDescription()
> hasPrototypeProperty()
> Object.keys()
> 

`原型模式缺点`: 当p1修改name值,p2的name值也会修着修改`.属性是被很多实例共享,这种共享对于函数非常适合,但是对于实例私有属性不适合.

### 组合使用构造函数模式和原型模式

创建自定义类型最常见的模式. `构造函数模式用于定义实例属性, 原型模式用于定义方法和共享属性.`
```js
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.friends = ['AAA', 'BBB'];
}

Person.prototype = {
  constructor: Person,
  sayName: function() {
    alert(this.name);
  }
};

var p1 = new Person('Jack', 24, 'JS');
var p2 = new Person('Tonny', 29, 'Java');

p1.friends.push('CCC');
alert(p1.friends); // 'AAA', 'BBB', 'CCC'
alert(p2.friends); // 'AAA', 'BBB'

alert(p1.sayName == P2.sayName); // true
```

### 动态模型加载
```js
function Person() {
  this.name = name;
  if (typeof this.sayName != 'function') {
    Person.prototype.sayName = function() {
      alert(this.name);
    }
  }
}

var p1 = new Person('Jack', 24, 'JS');
p1.sayName(); // 'Jack'
```
sayName不存在的情况下才会添加原型中.这段代码只会在初次调用构造函数时才会执行,要记住对原型的修改会立即反馈到所有实例中.

> 使用动态模型时,不能使用对象字面量重写原型,如果已经创建实例的情况下重写原型,那么会切断现有实例与原型之间的联系.


### 寄生构造函数模式

上述模式都不使用情况下,可以使用此法.即和工厂模式一样.在构造函数内部创建一个零时变量.
特殊情况下:假设创建一个具有额外方法的特殊数组

```js
function SpecialArray() {
  var values = new Array(); //创建数组
  values.push.apply(values, arguments); // 添加值
  values.toPipedString = function() {
    return this.join('|');  // 添加方法
  };

  return values; //返回值
}

var colors = new SpecialArray('red', 'blue', 'green');
colors.toPipedString();  //'red|blue|green'
```

特殊说明: 返回对象与构造函数或者构造函数的原型属性之间没有关系,不能依赖instanceof来确定对象类型,由此不建议使用

### 稳妥构造函数模式

指:没有公共属性,方法也不引用this对象,最实用在一些安全环境中,防止数据被其他引用程序改动使用.
`稳妥构造函数模式遵循`与`寄生构造函数类似模式`,但有不同:
1, 新创建对象的实例方法不引用this
2, 不使用new操作符调用构造函数

// 重写前面实例
```js
function Person(name, age, job) {
  var o = new Object(); //创建要返回对象
  // 可以在此定义私有变量和函数
  o.sayName = function() {  // 添加方法
    alert(name);
  }

  return o; //返回对象
}
```

运用到ADsafe, Caja 缺点和寄生构造函数模式一样

## 继承

原型链:作为实现继承的主要方法, 基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法.

`构造函数`,`原型`和`实例`的关系: (important)

> 每个构造函数都有一个原型对象,
> 原型对象都包含一个指向构造函数的指针
> 实例都包含一个指向原型对象的内部指针

如果让原型对象等于另一个类型的实例(SubType.prototype = new SuperType();), 此时原型对象将包含一个指向另一个原型的指针,相应地,另一个原型中也包含着一个指向另一个构造函数的指针.加入另一个原型又是另一个类型的实例, 层层递进,就构成了实例与原型的链条.这就是所谓的原型链的基本概念.

```js
function SuperType() {
  this.property = false;
}

SuperType.prototype.getSuperValue = function() {
  retun this.property;
}

function SubType() {
  this.subproperty = true;
}

// 继承SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function() {
  return this.property;
}

var inst = new SubType();
inst.getSuperValue(); // true
```

以上代码外加默认的原型如下图所示:

![原型链图](/img/js-high-level-6-3.png)

注意: 执行一个方法,1)先搜索实例, 2)搜索SubType.prototype 3)搜索SuperType.prototype,一直到Object.prototype顶端.

- 确定原型和实例关系
1, 使用instanceof操作符
inst instanceof Object; // true
inst instanceof SuperType; // true
inst instanceof SubType; // true

2, isPropertyOf()方法
Object.prototype.isPropertyOf(inst); // true
SuperType.prototype.isPropertyOf(inst); // true
SubType.prototype.isPropertyOf(inst); // true

- 谨慎定义方法
子类重写超类中的方法将会屏蔽掉原来超累方法(`根据先后搜索顺序可知道`);

注意: 通过原型链实现继承时,不能使用对象字面量创建原型方法,因为会重写原型链.如下:

```js
// 继承SuperType
SubType.prototype = new SuperType();

// 适用对象字面量添加新方法会导致上一行代码无效被覆盖
SubType.prototype = {
  getSubValue: function() {
    return this.subproperty;
  },
  someOtherMethod: function() {
    return false;
  }
};

// 这样会切断原型链,此时SubType和SuperType已无关系.
```

- 原型链问题

1, 包含引用类型值的类型 (引用类型值会被所有实例共享,这也正是为什么要在构造函数中定义属性,而不是在原型对象中)
2, 在创建子类型实例时,不能向超类型的构造函数中传递参数.因此`实践中很少会单独使用原型链`

解决方法:如下继续

### 借用构造函数

```js
function Super() {
  this.colors = ['red','blue'];
}

function Sub() {
  // 继承了Super
  Super.call(this);  // 可以用apply()
}

var ins1 = new Sub();
ins1.colors.push('green');
ins1.colors; // 'red,blue,green'

var ins2 = new Sub();
ins2.colors; // 'red,blue'
```

传递参数:

```js
function Super(name) {
  this.name = name;
}

function Sub() {
  // 继承了Super,同时还传递了参数
  Super.call(this, 'Jack');  // 可以用apply(this, ['Jack'])

  this.age = 29; // 实例属性
}

var ins1 = new Sub();
ins1.name; // 'Jack'
ins1.age;  // 29

// 确保父类不会重写子类型属性, 可以先调用超类构造函数后,在添加应该在子类型中定义的属性
```

`缺点:方法都在构造函数中定义,函数复用就无从谈起,而且超类型原型中定义的方法对子类不可见`;因此不推荐

### 组合继承,也叫伪经典继承

指的是将原型链和构造函数的技术组合到一起.思路:原型链实现对原型属性和方法的继承,通过构造函数来实现对实例属性的继承.

```js
function Super(name) {
  this.name = name;
  this.colors = ['red','blue'];
}

Super.prototype.sayName = function() {
  alert(this.name);
};

function Sub(name, age) {
  // 继承Super属性
  Super.call(this, name);  // 可以用apply()

  this.age = age;
}

Sub.prototype = new Super();  // 继承Super方法

Sub.prototype.sayAge = function() {
  alert(this.age);
}

var ins1 = new Sub('Jack', 24);
ins1.colors.push('green');
ins1.colors; // 'red,blue,green'
ins1.sayName(); // 'Jack'
ins1.sayAge(); // 24

var ins2 = new Sub('Tonny', 25);
ins2.colors; // 'red,blue'
ins2.sayName(); // 'Tonny'
ins2.sayAge(); // 25
```

最常用的继承模式,instanceof 和 isPropertyOf() 能识别基于继承组合创建的对象

### 原型式继承

这种方法没有严格意义上的构造函数,只是借助`原型可以基于已有的对象创建新对象`,`同时还不必因此创建自定义类型`.

```js
function object(obj) {
  function F() {};
  F.prototype = obj;
  return new F();
}
```

在函数内部先创建一个临时性构造函数,然后将传入`对象`作为构造函数的原型,最后返回了这个临时类型的一个新实例.
`实际上object()对传入的对象执行了一次浅复制`.

```js
var person = {
  name: 'Jack',
  friends: ['A', 'B', 'C']
};

var p1 = object(person);
p1.name = 'Tony';
p1.friends.push('D');

var p2 = object(person);
p2.name = 'Chan';
p2.friends.push('E');

person.friends; // 'A,B,C,D,E'
```

实际上相当于创建了person对象的两个副本.
`ECMAScript5`通过新增Object.create()方法规范了原型式继承. 接受两个参数:一个用作新对象原型的对象,另一个(可选)为新对象定义额外属性的对象,在传入一个参数情况下`Object.create()`与`object()`的行为是一样的.


```js
var person = {
  name: 'Jack',
  friends: ['A', 'B', 'C']
};

//传递一个参数:
var p1 = Object.create(person);
p1.name = 'Tony';
p1.friends.push('D');

var p2 = Object.create(person);
p2.name = 'Chan';
p2.friends.push('E');

person.friends; // 'A,B,C,D,E'
```

```js
var person = {
  name: 'Jack',
  friends: ['A', 'B', 'C']
};

//传递两个参数:
var p1 = Object.create(person, {
  name: {
    value: 'Tony'
  }
});
p1.name; // 'Tony'
```

### 寄生式继承

`寄生式继承`与`原型式继承`紧密关联,思路和`寄生构造函数`,'工厂模式'类似:即创建一个`仅用于封装继承过程`的函数.
该函数在内部以某种方式来增强对象,组后在返回对象

```js
function createAnother(original) {
  // 原型式继承
  var clone = object(original); // 通过调用函数创建一个息对象
  clone.sayHi = function() {  // 以某种方式来增强对象
    alert('Hi');
  };
  return clone; // 返回对象
}
```

在考虑对象不是`自定义类型`和`构造函数`情况下,`寄生式继承`也是一种有用的模式. 
`缺点:不能做到函数复用而降低效率`.

### 寄生组合式继承

`组合继承`是JS最常用的继承模式. 但是`组合继承`无论在什么情况下`都会调用两次超类型构造函数`.
> 1, 创建子类型原型的时候 Sub.prototype = new Super(); // name 和 colors都是Super实例属性
> 2, 子类型构造函数内部 Super.call(this, name); // name和colors是新对象实例属性,屏蔽Super中的两个同名属性

所谓寄生组合式继承:通过借用构造函数来继承属性,通过原型链继承方法,不必为指定子类型的原型而调用超类的构造函数,所需的无非就是超类型原型的一个副本而已.本质上,使用寄生组合式继承来继承超类型的原型,然后再将结果指定给子类型的原型.

```js
function inheritPrototype(subType, superType) { //子类构造函数和超类构造函数
  var prototype = object(superType.prototype); // 创建超类原型的一个副本
  prototype.constructor = subType; // 增强对象
  subType.prototype = prototype; // 指定对象
}
```

```js
function SuperType(name) {
  this.name = name;
  this.colors = ['red','blue'];
}

SuperType.prototype.sayName = function() {
  alert(this.name);
};

function SubType(name, age) {
  // 继承Super属性
  SubType.call(this, name);  // 可以用apply()

  this.age = age;
}

// Sub.prototype = new Super(); 被替换掉成
inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function() {
  alert(this.age);
}
```

![原型链图1](/img/js-high-level-6-4.png)

![原型链图2](/img/js-high-level-6-5.png)

这个例子的高效率体现在只调用一次SuperType构造函数,避免SuperType.prototype上面创建不必要,多余的属性.
同时,原型链还能保持不变.可以正常使用instanceof 和 isPropertyOf(). 这是引用类型最理想的继承范式.

> 终于写完了.这章真长呀.但是经典呀.受益匪浅

## 参考文档

- [JavaScript高级程序设计（第三版）](http://www.ituring.com.cn/book/946)

-----------------------

> 文章若有纰漏，请大家补充指正，谢谢！
> 如有疑问可留言，我会在第一时间回复。
> [转载请注明出处: http://blog.uijack.com/](http://blog.uijack.com/)
