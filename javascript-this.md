# JavaScript This详解


你皮任你皮，我当你瓜皮。


众所周知，`this`在JavaScript中的指向一直很难让人理解，想要学好JavaScript，`this`也是我们必须要搞清楚的。其实，`this`并没有那么难，本文将力争带你治疗`this`的“皮”。


首先，来科普三个问题！

## this是什么？

**`this`是声明函数时附加的参数**，指向特定的对象，也就是隐藏参数。

比如LOL中各种隐藏的彩蛋，当水晶先锋斯卡纳在草丛里停留超过15秒不动，会模仿宠物小精灵。

“皮卡！皮卡！皮卡丘！”

“斯卡！斯卡！斯卡纳！” 

好，相信大家已经理解什么是`this`了，就是个隐藏参数，没有多么的神奇，其实 每个函数都可以访问`this`。

## 为什么要使用this？

`this`提供了一种更加优雅的方式来隐式的传递对象引用。

通俗地说：就是说我们可以把API设计的更加简洁而且易于复用。

说人话：那就是`this`可以帮我们省略参数。

## this的指向


只需要理解并记住一句话，外加几种小情况，大家就可以完完全全的理解`this`。

好，注意听讲。

这句话就是
“**`this`的指向在函数声明的时候是不会被确定的，只有函数执行的时候才被确定，`this`最终指向的是调用它的对象**”。

有人说这也太长了，记不住。

好，那缩短点。

一句话。

“**`this`的指向决定于函数的调用方式**”。

总结：
> 1.`this`是声明函数时附加的参数，指向特定的对象，也就是隐藏参数。
>
> 2.`this`可以帮我们省略参数。
>
> 3.`this`的指向决定于函数的调用方式。



是不是很简单，搞清楚了这三点，我们下面将从八种情况来彻底理解`this`的指向问题。

直接上代码。

### 一、直到世界尽头

(对，没错，有没有想起灌篮高手主题曲)

我们首先要了解一下世界观，分为三种情况。

> 1.在非严格模式下，浏览器中的尽头当然就是`window`。
>
> 2.在严格模式下也就是开启了`"use strict"`的情况下，尽头就是`undefined`。
>
> 3.`node`的全局环境中尽头是`global`。

下文中情况主要从第一种非严格模式下来对`this`的指向进行解释和说明。

```javascript



// 我们来看下面的两个🌰。

function demo(){
  var user = “前端食堂”;
  console.log(this.user);   // undefined
  console.log(this);       // window
}

demo();

// 这里的函数demo实际上是被window对象使用点语法所点出来的，如下：

function demo(){
    var user = “前端食堂”;
    console.log(this.user);    // undefined
    console.log(this);        // window
}

window.demo();

// 可以发现和上面的代码结果一样

```


### 二、画龙点睛


```javascript
var obj = {
  user:”前端食堂”,
  fn:function(){
     console.log(this.user);   // 前端食堂
  }
}

obj.fn();

// 这里的this指向对象obj
// 注意看最后一行调用函数的代码
// obj.fn();

// 重要的事情说两遍！！
// this的指向在函数创建时是决定不了的
// 在调用的时候才可以决定，谁调用就指向谁

// this的指向在函数创建时是决定不了的
// 在调用的时候才可以决定，谁调用就指向谁

```



### 三、点石成金


```javascript
// 其实以上两点说的还不够准确，我们接着往下看

var obj = {
  user:”前端食堂”,
  fn:function(){
     console.log(this.user); // 前端食堂
  }
}
window.obj.fn(); 

// 这段代码跟上面的代码几乎是一样的
// 但是这里为什么没有指向window呢？
// 按你上面说的不是window调用的方法吗？
// 大家先停下来打个debugger思考一下为什么
// 想不明白没关系我们带着疑问来看下段代码

var obj = {
    a:1,
    b:{
        a:2,
        fn:function(){
            console.log(this.a); // 2
        }
    }
}

obj.b.fn();

// 这里执行的时候同样是对象obj通过点语法进行的执行
// 但是this同样没有指向window，这是为什么呢？

// 好，我们有几种情况需要记住：

// 1.如果一个函数中有this
// 但是它没有被上一级的对象所调用
// 那么this就会指向window（非严格模式下）

// 2.如果一个函数中有this
// 这个函数又被上一级的对象所调用
// 那么this就会指向上一级的对象

// 3.如果一个函数中有this
// 这个函数中包含多个对象
// 尽管这个函数是被最外层的对象所调用
// this却会指向它的上一级对象

var obj = {
  a:1;
  b:{
    // a:2,
    fn:function(){
       console.log(this.a);   // undefined
    }
  }
}
obj.b.fn();

// 我们可以看到，对象b中没有属性a，这个this指向
// 的也是对象b，因为this只会指向它的上一级对象
// 不管这个对象中有没有this要的东西

// 我们再来看一种情况

var obj = {
    a:1,
    b:{
        a:2,
        fn:function(){
            console.log(this.a);  // undefined
            console.log(this);   // window
        }
    }
}
var demo = obj.b.fn;
demo();

// 在上面的代码中，this指向的是window
// 你们可能会觉得很奇怪
// 其实是这样的，有一句话很关键，再次敲黑板

// this永远指向的都是最后调用它的对象
// 也就是看它执行的时候是谁调用的

// 上面的例子中虽然函数fn是被对象b所引用了
// 但是在将fn赋值给变量demo的时候并没有执行
// 所以最终this指向的是window
```



### 四、青梅竹马


```javascript
function returnThis(){
  return this;
}
var user = {name:"前端食堂"};

returnThis();            // window
returnThis.call(user);   // 前端食堂
returnThis.apply(user) ; // 前端食堂

// 这里就是Object.prototype.call
// 和Object.prototype.apply方法
// 他们可以通过参数来指定this

```




### 五、矢志不渝


```javascript
function returnThis(){
  return this;
}

var user1 = {name:"前端食堂"};
var user1returnThis = returnThis.bind(user1);
user1returnThis();             // 前端食堂
var user2 = {name:"前端小食堂"};
user1returnThis.call(user2);   // still 前端食堂

// Object.prototype.bind通过一个新函数来提供了永久的绑定
// 而且会覆盖call和apply的指向

```



### 六、乾坤大挪移


```javascript
function Fn(){
  this.user = "前端食堂";
}
var demo = new Fn();
console.log(demo.user);  // 前端食堂

// 这里new关键字改变了this的指向
// new关键字创建了一个对象实例
// 所以可以通过对象demo点语法点出函数Fn里面的user
// 这个this指向对象demo

// 注意：这里new会覆盖bind的绑定
function demo(){
  console.log(this);
}

demo();             // window
new demo();         // demo
var user1 = {name:"前端食堂"};
demo.call(user1);   // 前端食堂

var user2 = demo.bind(user1);
user2();            // 前端食堂
new user2();        // demo

```



### 七、爱转角遇上return


```javascript
// 当this遇上return时
function fn(){
  this.user = "前端食堂";
  return{};
}
var a = new fn;
console.log(a.user);  // undefined

function fn(){
  this.user = "前端食堂";
  return function(){};
}
var a = new fn;
console.log(a.user);  // undefined

function fn(){
  this.user = "前端食堂";
  return 1;
}
var a = new fn;
console.log(a.user); // 前端食堂

function fn(){
  this.user = "前端食堂";
  return undefined;
}
var a = new fn;
console.log(a.user); // 前端食堂

// 总结：如果返回值是一个对象
// 那么this指向就是返回的对象
// 如果返回值不是一个对象
// 那么this还是指向函数的实例

// null比较特殊，虽然它是对象
// 但是这里this还是指向那个函数的实例

function fn(){
  this.user = "前端食堂";
  return null;
}
var a = new fn;
console.log(a.user); // 前端食堂

```



### 八、英雄登场


```javascript

// 最后我们介绍一种在ES6中的箭头函数
// 这个箭头函数中的this被加里奥的英雄登场锤的不行
// 皮不起来了
// 而且，在代码运行前就已经被确定了下来
// 谁也不能把它覆盖

// 这样是为了方便让回调函数中this使用当前的作用域
// 让this指针更加的清晰
// 所以对于箭头函数中的this指向
// 我们只要看它创建的位置即可

function callback(qdx){
  qdx();
}
callback(()=>{console.log(this)});        // window

var user = {
    name:"前端食堂",
    callback:callback,
    callback1(){
      callback(()=>{console.log(this)});
    }
}
user.callback(()=>{console.log(this)});  // still window
user.callback1(()=>{console.log(this)}); // user
```


怎么样？this其实不过如此吧。再皮也要治住他~
