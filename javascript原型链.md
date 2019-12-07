---
title: "JavaScript原型链"
date: 2019-07-20T16:22:42+08:00
lastmod: 2019-08-25T16:22:42+08:00
draft: false
description: "JavaScript原型链"
show_in_homepage: true
show_description: false
license: ''

tags: ['JavaScript']
categories: ['前端']

featured_image: ''
featured_image_preview: ''

comment: true
toc: true
autoCollapseToc: true
math: true
---

`Brendan Eich(布兰登·艾奇)` 作为`JavaScript`的作者曾说过 **“它是`C`语言和`Self`语言一夜情的产物。”**


<!--more-->


是因为它借鉴了`C`语言的基本语法和`Self`语言的使用基于原型`(prototype)`的继承机制。


所以我们也经常可以看到`JavaScript`被描述为一种基于原型的语言，每个对象都有一个原型对象。


对象以它的原型作为模版、从原型上可以继承属性和方法。原型对象也是对象，也有它自己的原型对象，并从中继承属性和方法。



一层一层，以此类推……


如果你愿意一层一层的拨开我的心~

![不愿意](/images/js-prototype0.jpg)

上面说的这种关系就是


**原型链 `prototype chain`**



本文力争用通俗易懂的语言(人话)，来带大家搞懂原型对象、原型链。

## 函数和对象

首先我们先来看一下`JavaScript`中函数和对象的关系，这对我们理解原型链有很大的帮助。

你可能也会看到“`JavaScript`中万物皆对象”，这显然是错误的。

实际上在`JavaScript`中，有许多特殊的对象子类型，可以叫做复杂基本类型。

**函数就是对象的一个子类型。**

**函数的本质就是对象。**

但是为什么使用`typeof`进行类型检查的时候会有下面的结果呢？

```javascript
var o = {};                 // 普通对象
var f = function () {};     // 函数对象
console.log(typeof o);      //object
console.log(typeof f);      //function
```
JavaScript: 嘿，兄弟，我是一夜情的产物，不要要求太多！

[在JavaScript中，检测对象类型时，强烈建议使用Object.prototype.toString方法。typeof的一些返回值在标准文档中并未定义，因此不同的引擎实现可能不同。](http://bonsaiden.github.io/JavaScript-Garden/zh/#types.typeo)

回到正题，实际上，函数和对象没有本质的区别，函数是特殊的对象。


函数对象天生带有`prototype`属性，也就是每个函数在创建之后会天生拥有一个与之相关联的原型对象，这个原型对象中拥有一个`constructor`属性，该属性指向这个函数。

**注意：**

很多人认为新创建的函数对象身上除了`prototype`属性外，还有`constructor`这个属性。


**但是这里使用的`constructor`属性实际上是从原型链中获取的**

**即`Function.prototype.constructor`**

备注：在[ECMAScript标准](https://www.ecma-international.org/ecma-262/5.1/#sec-13.2)中函数创建相关章节有这样一句话：`NOTE A prototype property is automatically created for every function, to allow for the possibility that the function will be used as a constructor.`

解释了给新创建函数添加`prototype`属性的意义在于便于该函数作为构造函数使用。

搞清楚了`JavaScript`中函数和对象的关系后，我们接下来看一看什么是原型对象。

## 原型对象

当构造函数被创建出来的时候，会默认关联一个`Ojbect`类型的新对象，这个对象就是当前构造函数的原型对象，构造函数的原型对象默认是一个空对象。



当然，构造函数创建出来的对象可以访问该构造函数原型对象的属性和方法。

```javascript
// 1 声明构造函数Man
function Man(name) {
    this.name = name;
}

// 2 打印Man的原型对象看看
console.log(Man.prototype); // Object类型的空对象

// 3 在Man的原型对象上添加getName方法
Man.prototype.getName = function () {
    console.log(this.name);
}
// 4 使用构造函数创建对象实例
var d = new Man("笛巴哥");
d.getName();  // 笛巴哥
console.log(d);
```
![关系图](/images/js-prototype1.png)

通过上面的代码和结果进行分析，我们可以得出构造函数、实例、原型对象三者之间的关系。

![分析图](/images/js-prototype2.png)

**我们可以得出以下结论**

1.构造函数`Man`可以通过`prototype`属性访问到它的原型对象。

2.通过构造函数`Man`实例化出来的`d`可以通过`__proto__`属性访问到`Man`的原型对象。

3.`Man`的原型对象可以通过`constructor`(构造器)属性访问其关联的构造函数。

**我们可以通过三种方式来访问原型对象**

1.构造函数.`prototype`

2.实例对象.`__proto__`

3.`object.getPrototypeOf`(实例对象)

### prototype

`prototype` 函数对象拥有的属性，指向它的原型对象。

### __proto__

`__proto__` 所有的对象都拥有`__proto__`属性，指向实例的原型。

### constructor

`construtor` 构造器，原型对象可以通过`constructor`来访问其所关联的构造函数。当然，每个实例对象也从原型中继承了该属性。

**注意：**

`__proto__`属性并不在`ECMAScript`标准中，只为了开发和调试而生，不具备通用性，不能出现在正式的代码中。


搞清楚了原型对象，接下来我们来看原型链。

## 原型链

```javascript
function Man () {};
function Woman () {};

var m1 = new Man();
var w1 = new Woman();
```

![分析图](/images/js-prototype3.png)

这一夜，原型链没少折腾...


```javascript
// 1.让我们看一看食物链(原型链)的顶端 null
console.log(Object.prototype.__proto__);                      //null

// 2.Function.prototype的原型对象为Object.prototype而不是它自己
console.log(Function.prototype.__proto__ == Object.prototype);//true

// 3.Function和Object的构造函数都是Function
console.log(Function.constructor == Function);                //true
console.log(Object.constructor == Function);                  //true

// 4.Function.prototype的构造函数是Function
console.log(Function.prototype.constructor == Function);      //true

// 5.m1的原型对象为Man.prototype
console.log(m1.__proto__ == Man.prototype);  //true

// 6.Man.prototyepe|Woman.prototype的constructor指向Object
// Man.prototyepe|Woman.prototype的原型对象为Object.prototype
// 先删除实例成员，通过原型成员访问
delete  Man.prototype.constructor;
delete  Woman.prototype.constructor;
console.log(Man.prototype.constructor == Object);    //true
console.log(Woman.prototype.constructor == Object);    //true
console.log(Man.prototype.__proto__ == Object.prototype);    //true
console.log(Woman.prototype.__proto__ == Object.prototype);    //true

// 7.Man和Woman的构造函数为Function
// Man和Woman的构造函数的原型对象为空函数
console.log(Man.constructor == Function);                //true
console.log(Woman.constructor == Function);                //true
console.log(Man.__proto__ == Function.prototype);        //true
console.log(Woman.__proto__ == Function.prototype);        //true
```

## 原型链的访问规则

**就近原则**



对象在访问属性或方法时，先检查自己的实例，如果存在就直接使用。如果不存在那么就去原型对象上去找，存在就直接使用，如果没有就顺着原型链一直往上查找，找到即使用，找不到就重复该过程直到原型链的顶端，如果还没有找到相应的属性或方法，就返回`undefined`，报错。

## 三种检验方法

`Object.getPrototypeOf`方法用于获取指定实例对象的原型对象。

```javascript
function Demo(){};
var demo = new Demo();

console.log(Object.getPrototypeOf(demo));  // 打印结果为Demo相关联的原型对象
console.log(Object.getPrototypeOf(demo === Demo.prototype));  // true
console.log(Object.getPrototypeOf(demo === Demo.__proto__));  // true

```

`isPrototypeOf`方法用于检查某对象是否在指定对象的原型链中。

```javascript
function Demo(){}
var demo = new Demo();

console.log(Demo.prototype.isPrototypeOf(demo)); // true
console.log(Object.prototype.isPrototypeOf(demo)); //true
```

`instanceof`运算符的作用跟`isPrototypeOf`方法类似，左操作数是待检测的实例对象，右操作数是用于检测的构造函数。如果右操作数指定构造函数的原型对象在左操作数实例对象的原型链上面，则返回结果`true`，否则返回结果`false`。

```javascript
// 1 声明构造函数Demo
function Demo(){};
// 2 获取实例化对象demo
var demo = new Demo();
// 3 使用isPrototypeOf
console.log(demo instanceof Demo);     //true
console.log(demo instanceof Object);   //true

// 4 Object构造函数的原型对象在Function这个实例对象的原型链中
console.log(Function instanceof Object); //true
// 5 Function构造函数的原型对象在Object这个实例对象的原型链中
console.log(Object instanceof Function); //true
```

**注意**：不要错误的认为`instanceof`检查的是该实例对象是否从当前构造函数实例化创建的，其实它检查的是实例对象是否从当前指定构造函数的原型对象继承属性。

## 最佳的组合继承方案

**思路**


1.使用原型链实现对原型属性和方法的继承

2.通过伪造(冒充)构造函数来实现对实例成员的继承，并且解决了父构造函数传参问题

```javascript
// 1 提供超类型|父类型
function SuperClass(name) {
    this.name = name;
    this.showName = function () {
        console.log(this.name);
    }
}

// 2 设置父类型的原型属性和原型方法
SuperClass.prototype.info = 'SuperClass的信息';
SuperClass.prototype.showInfo = function () {
    console.log(this.info);
};

// 3 提供子类型
function SubClass(name) {
    SuperClass.call(this,name);
}

// 获取父构造函数的实例成员  Person.call(this,name);
// 获取父构造函数的原型成员  SubClass.prototype = SuperClass.prototype;
SubClass.prototype = SuperClass.prototype;
SubClass.prototype.constructor = SubClass;

var sub_one = new SubClass("zhangsan");
var sub_two = new SubClass("lisi");
console.log(sub_one);
console.log(sub_two);
```

参考
[文顶顶的博客](http://wendingding.com/2018/04/15/javaScript系列%20[04]-javaScript的原型链/)
