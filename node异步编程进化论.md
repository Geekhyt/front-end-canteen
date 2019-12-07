---
title: "Node.js异步编程进化论"
date: 2019-11-20T16:22:42+08:00
lastmod: 2019-08-25T16:22:42+08:00
draft: false
description: "Node.js异步编程进化论"
show_in_homepage: true
show_description: false
license: ''

tags: ['Node.js']
categories: ['前端']

featured_image: ''
featured_image_preview: ''

comment: true
toc: true
autoCollapseToc: true
math: true
---

我们知道，Node.js中有两种事件处理方式，分别是`callback`(回调)和`EventEmitter`(事件发射器)。本文首先介绍的是`callback`。

<!--more-->


## Node.js异步编程callback


`error-first callback` 错误优先是Node.js回调方式的标准。

第一个参数是`error`，后面的参数才是结果。


我们以现实生活中去面试来举个🌰，面试成功我们漏出洋溢的笑容，面试失败我们就哭并尝试找到失败的原因。

```javascript
try {
    interview(function() {
    	console.log('smile');
    });
} catch(e) {
    console.log('cry', e);
}
function interview(callback) {
    setTimeout(() => {
        if (Math.random() < 0.1) {
            callback('success');
        } else {
            throw new Error('fail');
        }
    }, 500);
}
```
如上代码运行后，`try/catch`并不像我们所想，它并没有抓取到错误，错误反而被抛到了`Node.js`全局，导致程序崩溃。（**是由于Node.js的每一个事件循环都是一个全新的调用栈Call Stack**）

为了解决上面的问题，Node.js官方形成了如下规范：

```javascript
interview(function (res) {
    if (res) {
        return console.log('cry');
    }
    console.log('smile');
})
function interview (callback) {
    setTimeout(() => {
        if (Math.random() < 0.8) {
            callback(null, 'success');
        } else {
            callback(new Error('fail'));
        }
    }, 500);
}
```

### 回调地狱`Callback hell`

XX大厂有三轮面试，看下面的🌰

```javascript
interview(function (err) {
    if (err) {
        return console.log('cry at 1st round');
    }
    interview(function (err) {
        if (err) {
            return console.log('cry at 2nd round');
        }
        interview(function (err) {
            return console.log('cry at 3rd round');
        })
        console.log('smile');
    })
})
function interview (callback) {
    setTimeout(() => {
        if (Math.random() < 0.1) {
            callback(null, 'success');
        } else {
            callback(new Error('fail'));
        }
    }, 500);
}
```

我们再来看并发情况下callback的表现。

同时去两家公司面试，当两家面试都成功时我们才会开心，看下面这个🌰

```javascript
var count = 0;
interview(function (err) {
    if (err) {
        return console.log('cry');
    }
    count++;
})
interview(function (err) {
    if (err) {
        return console.log('cry');
    }
    count++;
    if (count) {
        //当count满足一定条件时，面试都通过
        //...
        return console.log('smile');
    }
})
function interview (callback) {
    setTimeout(() => {
        if (Math.random() < 0.1) {
            callback(null, 'success');
        } else {
            callback(new Error('fail'));
        }
    }, 500);
}
```


异步逻辑的增多随之而来的是嵌套深度的增加。如上的代码是有很多缺点的：

- 代码臃肿，不利于阅读与维护
- 耦合度高，当需求变更时，重构成本大
- 因为回调函数都是匿名函数导致难以定位bug

为了解决回调地狱，社区曾提出了一些解决方案。


> 1.[async.js](https://www.npmjs.com/package/async) npm包，是社区早期提出的解决回调地狱的一种异步流程控制库。
>
> 2.thunk 编程范式，著名的`co`模块在v4以前的版本中曾大量使用Thunk函数。Redux中也有中间件`redux-thunk`。

不过它们都退出了历史舞台。

毕竟软件工程没有银弹，取代他们的方案是`Promise`

## Promise
[Promise/A+](https://promisesaplus.com)规范镇楼，ES6采用的这个规范实现的Promise。

Promise 是异步编程的一种解决方案，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。

简单说，Promise就是当前事件循环不会得到结果，但未来的事件循环会给到你结果。

毫无疑问，Promise是一个渣男。

Promise也是一个状态机，**只能从`pending`变为以下状态(一旦改变就不能再变更)**

- fulfilled(本文称为resolved)
- rejected

```javascript
// nodejs 不会打印状态
// Chrome控制台中可以
var promise = new Promise(function(resolve, reject){
    setTimeout(() => {
        resolve();
    }, 500)
}) 
console.log(promise);
setTimeout(() => {
    console.log(promise);
}, 800);
// node.js中
// promise { <pending> }
// promise { <undefined> }
// 将上面代码放入闭包中扔到google控制台里
// google中
// Promise { <pending> }
// Promise { <resolved>: undefined }
```

Promise
- then
- catch

> `resolved`状态的Promise会回调后面的第一个`.then`
>
>`rejected`状态的Promise会回调后面的第一个`.catch`
>
>任何一个`rejected`状态且后面没有`.catch`的Promise，都会造成`浏览器/node环境`的全局错误。

Promise比callback优秀的地方，是可以解决异步流程控制问题。

```javascript
(function(){
    var promise = interview();
    promise
        .then((res) => {
            console.log('smile');
        })
        .catch((err) => {
            console.log('cry');
        });
    function interview() {
        return new Promise((resoleve ,reject) => {
            setTimeout(() => { 
               if (Math.random() > 0.2) {
                   resolve('success');
               } else {
                   reject(new Error('fail'));
               }
            }, 500);
        });
    }
})();
```


执行`then`和`catch`会返回一个新的Promise，该Promise最终状态根据`then`
和`catch`的回调函数的执行结果决定。我们可以看下面的代码和打印出的结果：


```javascript
(function(){
  var promise = interview();
  var promise2 = promise
      .then((res) => {
          throw new Error('refuse');
      });
      setTimeout(() => {
          console.log(promise);
          console.log(promise2);
      }, 800);   
  function interview() {
      return new Promise((resoleve ,reject) => {
          setTimeout(() => { 
             if (Math.random() > 0.2) {
                 resolve('success');
             } else {
                 reject(new Error('fail'));
             }
          }, 500);
      });
  }
})();
// Promise { <resolved>: "success"}
// Promise { <rejected>: Error:refuse }
```


> 如果回调函数最终是`throw`，该Promise是`rejected`状态。
>
> 如果回调函数最终是`return`，该Promise是`resolved`状态。
>
> 但如果回调函数最终`return`了一个Promise，该Promise会和回调函数`return Promise`状态保持一致。

### Promise解决回调地狱
我们来用Promise重新实现一下上面去大厂三轮面试代码。

```javascript
(function() {
    var promise = interview(1)
        .then(() => {
            return interview(2);
        })
        .then(() => {
            return interview(3);
        })
        .then(() => {
            console.log('smile');
        })
        .catch((err) => {
            console.log('cry at' + err.round + 'round');
        });
    function interview (round) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                if (Math.random() > 0.2) {
                    resolve('success');
                } else {
                    var Error = new Error('fail');
                    error.round = round;
                    reject(error);
                }
            }, 500);
        });
    }
})();
```

与回调地狱相比，Promise实现的代码通透了许多。

Promise在一定程度上把回调地狱变成了比较线性的代码，去掉了横向扩展，回调函数放到了then中，但其仍然存在于主流程上，与我们大脑顺序线性的思维逻辑还是有出入的。

### Promise处理并发异步

```javascript
(function() {
    Promise.all([
        interview('Alibaba'),
        interview('Tencent')
    ])
    .then(() => {
        console.log('smile');
    })
    .catch((err) => {
        console.log('cry for' + err.name);
    });
    function interview (name) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                if (Math.random() > 0.2) {
                    resolve('success');
                } else {
                    var Error = new Error('fail');
                    error.name = name;
                    reject(error);
                }
            }, 500);
        });
    }
})();
```

上面代码中的`catch`是存在问题的。注意，**它只能获取第一个错误**。
## Generator

Generator和Generator Function是ES6中引入的新特性，是在Python、C#等语言中借鉴过来。

生成器的本质是一种特殊的迭代器。

```javascript
function * doSomething() {}
```
如上所示，函数后面带“*”的就是Generator。

```javascript
function * doSomething() {
    interview(1);
    yield; // Line (A)
    interview(2);
}
var person = doSomething();
person.next();  // 执行interview1，第一次面试，然后悬停在Line(A)处
person.next();  // 恢复Line(A)点的执行，执行interview2，进行第二次次面试
```

**next的返回结果**
> 第一个person.next()返回结果是{value:'', done:false}
>
> 第二个person.next()返回结果是{value:'', done:true}

关于next的返回结果，我们要知道，如果`done`的值为`true`，即代表Generator里的异步操作全部执行完毕。

为了可以在Generator中使用多个yield，`TJ Holowaychuk`编写了`co`这个著名的ES6模块。[co的源码](https://es6.ruanyifeng.com/#docs/generator-async)有很多巧妙的实现，大家可以自行阅读。


## async/await
Generator的弊端是没有执行器，它本身是为了计算而设计的迭代器，并不是为了流程控制而生。co的出现较好的解决了这个问题，但是为什么我们非要借助于co而不直接实现呢？

`async/await`被选为天之骄子应运而生。

`async function` 是一个穿越事件循环存在的function。

`async function`实际上是Promise的语法糖封装。它也被称为**异步编程的终极方案-以同步的方式写异步**。

`await`关键字可以"暂停"`async function`的执行。

`await`关键字可以以同步的写法获取Promise的执行结果。

`try/catch`可以获取`await`所得到的任意错误，解决了上面Promise中catch只能获取第一个错误的问题。

### async/await解决回调地狱
```javascript
(async function () {
  try {
      await interview(1);
      await interview(2);
      await interview(3);
  } catch (e) {
      return console.log('cry at' + e.round);
  }
  console.log('smile');
})();
```
### async/await处理并发异步

```javascript
(async function () {
    try {
        await Promise.all([interview(1), interview(2)]);
    } catch (e) {
        return console.log('cry at' + e.round);
    }
    console.log('smile');
})();
```

无论是相比`callback`，还是`Promise`，`async/await`只用短短几行代码便实现了异步流程控制。

遗憾的是，`async/await`最终没能进入`ES7`规范(只能等到`ES8`)，但在`Chrome V8`引擎里得以实现，`Node.js v7.6`也集成了`async`函数。

### 实践经验总结

在常见的Web应用中，在DAO层使用Promise较好，在Service层使用async函数较好。


参考：

- 狼书-更了不起的Node.js
- Node.js开发实战

