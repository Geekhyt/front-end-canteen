
# JavaScript设计模式(二)


有人问我坚持写博客的原因是什么？

借用著名小说家`斯蒂芬·金`的一句话：“开始写吧！年轻人。”

当你把`消费级兴趣`升级为`生产型兴趣`时，你才会渐渐发现以前没有窥见的门道和妙处。


你品，你细品！

咳咳，天凉了请喝鸡汤～

可能本系列文章中所讲的设计模式你在工作中经常应用它们，但是并不知道它们的名字。

一旦我们将这些设计模式整理学习并融会贯通后，便可以大大增强我们的编程功底，在遇到实际业务需求时，给我们提供更好的解决问题的思路。

毕竟我们的口号是：不加班！

书接上文，本文给大家介绍的是`结构型设计模式`，包括`外观模式`、`适配器模式`、`代理模式`、`装饰者模式`、`桥接模式`、`组合模式`以及`享元模式`。


## 外观模式 Facade
`概念：可以对复杂的子系统接口提供一个更高级的统一接口，对底层结构兼容性做统一封装来简化用户使用。`

这种模式比较简单也比较容易理解，在日常的开发中你一定遇到过以下的场景。

那些年我们对ie浏览器做过的兼容。。


IE：我太难了！

这些常见的兼容方案属于外观模式。

```javascript
var getEvent = function (event) {
  return event || window.event;
}

var getTarget = function (event) {
  var event = getEvent(event);
  return event.target || event.srcElement;
}

var preventDefault = function (event) {
  var event = getEvent(event);
  if (event.preventDefault) {
    event.preventDefault();
  } else {
    event.returnValue = false;
  }
}

var stopBubble = function (event) {
  var event = getEvent(event);
  if (event.stopPropagation) {
    event.stopPropagation();     
  } else {
    event.cancelBubble = true;
  }
}
```

在团队开发中，为了避免添加点击事件时出现的重名覆盖问题，我们会使用DOM2级事件处理程序绑定事件，并添加对浏览器的兼容。

```javascript
function addEvent (dom, type, fn) {
  if (dom.addEventListener) {
    dom.addEventListener (type, fn, false);
  } else if (dom.attachEvent) {
    // 兼容IE(低于9)
    dom.attachEvent('on' + type, fn);
  } else {
    dom['on' + type] = fn;
  }
}
```
你当然也可以通过外观模式，来封装多个功能来简化底层操作方法。

```javascript
var T = {
  g : function (id) {
    return document.getElementById(id);
  },
  css : function (id, key, value) {
    document.getElementById(id).style[key] = value;
  },
  attr : function (id, key, value) {
    document.getElementById(id)[key] = value;
  },
  html : function (id, html) {
    document.getElementById(id).innerHTML = html;
  },
  on : function (id, type, fn) {
    document.getElementById(id)['on' + type] = fn;
  }
};

T.css('box', 'background', 'red');
T.attr('box', 'className', 'box');
T.html('box', '这是新添加的内容');
T.on('box', 'click', function() {
  T.css('box', 'width', '500px');
})
```
外观模式对接口进行了包装，我们使用时无须知道接口实现的具体细节，按照规则使用即可。

## 适配器模式 Adapter
`概念:将一个类(对象)的接口(方法或者属性)转化成另外一个接口，以满足用户需求，使类(对象)之间接口的不兼容问题通过适配器得以解决。`

适配器在我们的日常生活中很常见，比如出国旅行时，有的国家只有三项的插座，这时候我们需要三项转两项插头电源适配器。再比如iPhone7以后耳机接口变成了lightning接口，为了适配圆孔耳机苹果为我们提供了适配器。

```javascript
// 为两个代码库所写的代码兼容运行而书写的额外代码是适配器的一种。
window.A = A = jQuery;
```


### jQuery中的适配器

```javascript
jQuery.fn.css()
jQuery核心cssHook
// (为了控制文数，此处不贴代码，阅读完后大家可自行去官网查看)
```
[jQuery源码地址](https://jquery.com/download/)

### 参数适配器

```javascript
function doSomeThing(name, title, age, color, size, prize) {}
// 我们记住这些参数的顺序是很困难的，所以我们经常是以一个参数对象方式来传入
/**
* obj.name : name
* obj.title : title
* obj.age : age
* obj.color : color
* obj.size : size
* obj.prize : prize
**/
// 当调用它的时候我们要考虑到传递的参数是否完整的问题，如果有一些必须参数没有传入
// 一些参数是有默认值的等等。这时我们可以用适配器来适配传入的参数对象
function doSomeThing (obj) {
  var _adapter = {
    name : '前端食堂',
    title : '设计模式',
    age : 24,
    color : 'blue',
    size : 100,
    prize : 99
  };
  for (var i in _adapter) {
    _adapter[i] = obj[i] || _adapter[i];
  }
  // 或者extend(_adapter, obj) 注: 此时可能会多添加属性
  // do things
}
```
很多插件对于参数配置都是这么做的。

### 数据适配

```javascript
var arr = ['前端食堂', 'restaurant', '记得按时吃饭', '9月30日'];

// 我们发现数组中每个成员代表的意义不同，所以这种数据结构 语义不好，我们将其适配成对象。

function arrToObjAdapter (arr) {
  return {
    name : arr[0],
    type : arr[1],
    title : arr[2],
    data : arr[3]
  }
  var adapterData = arrToObjAdapter(arr);
  console.log(adapterData);  
  // {name:"前端食堂", type:"restaurant", title:"记得按时吃饭",data:"9月30日"}  
}
```

### 服务端数据适配
现在主流的方案是用`node.js`写中间层-`BFF层(Backend for Frontend)`，获取到后台返回的数据后进行处理，
处理为前端想要的数据，这样也不失为一种适配器模式。


**与外观模式的区别：**

相比于外观模式，适配器模式要了解适配对象的内部结构。


## 代理模式 Proxy
`概念：由于一个对象不能直接引用另一个对象，所以需要通过代理对象在这两个对象之间起到中介的作用。`

现实生活中比如我们租房子时回去自如，自如是房屋中介机构，也就是我们的代理。

在`javaScript`中使用最多的就是虚拟代理和缓存代理。

### 虚拟代理
图片loading预加载
```javascript
// 创建本体对象，生成img标签，对外提供setSrc接口
var myImage = (function() {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);

  return {
    setSrc: function (src) {
      imgNode.src = src;
    }
  }
})();

 // 引入代理对象，图片加载之前会有个loading.gif
 var proxyImage = (function () {
   var img = new Image();
   img.onload = function () {
     myImage.setSrc(this.src);
   }

   return {
     setSrc: function (src) {
       myImage.setSrc('loading.gif');
       img.src= src;
     }
   }
 })();
 // 通过代理得到图片
 proxyImg.setSrc('http://...');
```

### 缓存代理
可以为一些开销很大的运算结果提供暂时的储存，若下次传递的参数一致则直接返回之前的结果，大大提高效率和节省开销。
```javascript
var computedFn = function () {
  console.log('开始');
  var a = 1;
  for (var i = 0; l = arguments.length; i < l; i++) {
    a = a * arguments[i];
  }
  return a;
}

var proxyComputed = (function() {
  var cache = {};
  return function() {
    var args = Array.prototype.join.call(arguments, ',');
    if (args in cache) {
      return cache[args];
    }
    return cache[args] = computedFn.apply(this, arguments);
  }
})();

proxyComputed(1, 2, 3, 4); // 24
proxyComputed(1, 2, 3, 4); // 24
```
如上所示，代理模式可以解决系统之间的耦合度以及系统资源开销大的问题，

### ES6中的Proxy
ES6所提供的Proxy构造函数能够让我们轻松的使用代理模式。

```javascript
// target所要代理的对象
// handler设置对所代理的对象的行为
var proxy = new Proxy(target, handler);
```

[ES6中的Proxy](http://es6.ruanyifeng.com/#docs/proxy)

### Vue3.0中的Proxy
vue3.0中的双向数据绑定原理用了es6中的Proxy，并优雅的解决了Proxy细节上的一些问题，从而完美的实现双向绑定，大家可以去阅读源码，或是社区中的[这篇文章](https://juejin.im/post/5d99be7c6fb9a04e1e7baa34)，这里不作展开。


## 装饰者模式 Decorator
`概念：在不改变原对象的基础上，通过对其进行包装拓展(添加属性或者方法)使原有对象可以满足用户的更复杂需求。`

### TypeScript的装饰器@ 
装饰器是一项实验性特性，在未来的版本中可能会发生改变。

具体使用方法请移步[官方文档](https://www.tslang.cn/docs/handbook/decorators.html)

假设我们在开发中有这样一个需求，在不影响原有事件的前提下，新增事件实现业务，就可以通过装饰者来实现。
```javascript
// 装饰者
var decorator = function (input, fn) {
  // 获取事件源
  var input = document.getElementById(input);
  // 若事件源已经绑定事件
  if (typeof input.onclick === 'function') {
    // 缓存事件源原有回调函数
    var oldClickFn = input.onclick;
    // 为事件源定义新的事件
    input.onclick = function () {
      // 事件源原有回调函数
      oldClickFn();
      // 执行事件源新增回调函数
      fn();
    }
  } else {
    // 事件源未绑定事件，直接为事件源添加新增回调函数
    input.onclick = fn;
  }
}
```
**与适配者模式的区别：**

在适配者模式中要了解原有方法实现的具体细节，而在装饰者模式只有当我们调用方法时才会知道其内部细节，这是对原有功能完整性的一种保护。

## 桥接模式 Bridge
`概念：在系统沿着多个维度变化的同时，又不增加其复杂度并已达到解耦。`

```javascript
var spans = document.getElementsByTagName('span');
// 为用户名绑定特效
spans[0].onmouseover = function () {
  this.style.color = 'red';
  this.style.background = '#ddd';
}
spans[0].onmouseout = function () {
  this.style.color = '#333';
  this.style.background = '#f5f5f5';
}
// 为等级绑定特效
spans[1].onmouseover = function () {
  this.getELementsByTagName('strong')[0].style.color = 'red';
  this.getElementsByTagName('strong')[0].style.background = '#ddd';
}
spans[1].onmouseout = function () {
  this.getELementsByTagName('strong')[0].style.color = '#333';
  this.getElementsByTagName('strong')[0].style.background = '#f5f5f5';
}
```
```javascript
// 提取共同点，解除与事件中的this的耦合
function changeColor (dom, color, bg) {
  dom.style.color = color;
  dom.style.background = bg;
}

// 使用匿名函数，事件与业务逻辑之间的桥梁
var spans = document.getElementsByTagName('span');
spans[0].onmouseover = function () {
  changeColor(this, 'red', '#ddd';)
}

// changeColor方法中的dom实质上是事件回调函数中的this，解除它们之间的耦合
// 我们使用了一个桥接方法-匿名回调函数。通过这个匿名回调函数
// 我们将获取到的this传递到changeColor函数中，即可实现需求
spans[0].onmouseout = function () {
  changeColor(this, '#333', '#f5f5f5');
}
// 同样的道理应用在用户等级上
spans[1].onmouseover = function () {
  changeColor(this.getElementsByTagName('strong')[0], 'red', '#ddd');
}
spans[1].onmouseout = function () {
  changeColor(this.getElementsByTagName('strong')[0], '#333', '#f5f5f5');
}
// 如果需求再有变化，只需要修改changeColor的内容就可以了
// 而不必去到每个事件回调函数中去修改，以新增一个桥接函数为代价
```
将实现层(如元素绑定的事件)与抽象层(如修饰页面UI逻辑)解耦分离，使两部分可以独立变化。

注意，有些时候，对于桥梁的添加，会造成开发成本增加，性能上也会受影响。

## 组合模式 Composite
`概念：又称部分-整体模式，将对象组合成树形结构以表示“部分整体”的层次结构，组合模式使得用户对单个对象和组合对象的使用具有一致性。`

**中餐厅：套餐服务** 黄**：我觉得不行。

**webpack构建项目的目录结构**

**公司部门组织架构**

组合模式能够给我们提供一个清晰的组成结构。组合对象类通过继承同一个父类使其具有统一的方法，这样也方便了我们统一管理和使用。


### jQuery中addClass实现
```javascript
// jQuery 3.4.1
addClass: function( value ) {
	var classes, elem, cur, curValue, clazz, j, finalValue,
	i = 0;

	if ( isFunction( value ) ) {
		return this.each( function( j ) {
			jQuery( this ).addClass( value.call( this, j, getClass( this ) ) );
		} );
	}

	classes = classesToArray( value );

	if ( classes.length ) {
		while ( ( elem = this[ i++ ] ) ) {
			curValue = getClass( elem );
			cur = elem.nodeType === 1 && ( " " + stripAndCollapse( curValue ) + " " );

			if ( cur ) {
				j = 0;
				while ( ( clazz = classes[ j++ ] ) ) {
					if ( cur.indexOf( " " + clazz + " " ) < 0 ) {
							cur += clazz + " ";
					}
				}

				// Only assign if different to avoid unneeded rendering.
				finalValue = stripAndCollapse( cur );
				if ( curValue !== finalValue ) {
					elem.setAttribute( "class", finalValue );
				}
			}
		}
	}

	return this;
}
```
我们无须知道addClass的内部结构和实现细节，就可以使用addClass来对标签添加类。

## 享元模式 Flyweight
`概念：运用共享技术有效地支持大量的细粒度的对象，避免对象间拥有相同内容造成多余的开销。`

`核心思想`：共享细粒度对象

`最终目标`：尽量减少共享对象的数量

现实生活中，LOL、农药段位之分是享元模式的思想，咖啡厅的咖啡种类也是享元模式的思想。

在我们熟知的原型链继承中，当子类实例很多的时候，子类可以通过原型来复用父类的方法和属性来优化内存，这也是享元模式的思想。

除此之外，Node.js中的线程池、数据库的连接池、HTTP连接池以及字符常量池都是享元模式或其升级版。


参考：

《JavaScript设计模式》张容铭
