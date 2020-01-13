# 浏览器中的JavaScript事件循环

每天都在写JavaScript的你，是否清楚JavaScript引擎的原理呢？

想要了解JavaScript引擎，首先我们从它的运行机制`Event Loop`来说起。


首先科普一些基础知识。
<br/>

## 进程和线程

**进程**

应用程序的执行实例，每一个进程都是由私有的虚拟地址空间、代码、数据和其他系统资源所组成。

**线程**

线程是进程内的一个独立执行单元，在不同的线程之间是可以共享进程资源的。

有句老话是这样说的，穷养儿子富养女。

**进程就是一个富二代爸爸，它选择了穷养线程儿子。**

进程拥有独立的堆栈空间和数据段，每当启动一个新的进程必须分配给它独立的地址空间，建立众多的数据表来维护它的代码段、堆栈段和数据段。

线程拥有独立的堆栈空间，但是共享数据段，它们彼此之间使用相同的地址空间，共享大部分数据，比进程更节俭，开销比较小，切换速度也比进程快，效率高。

**一句话解释进程和线程**

进程：资源分配的最小单位

线程：程序执行的最小单位

关于进程和线程方面的知识我们先了解到这，感兴趣的同学们可以移步
[进程和线程的区别](https://www.cnblogs.com/Jones-dd/p/8858995.html)

Q&A再来回答一个问题：

在多线程操作下可以实现应用的并行处理，从而以更高的 CPU 利用率提高整个应用程序的性能和吞吐量。特别是现在很多语言都支持多核并行处理技术，然而 JavaScript 却以单线程执行，为什么呢？

答：JavaScript作为脚本语言，最初被设计用于浏览器。为了避免复杂的同步问题(做人嘛，还是简单点好，语言也一样)，如果JavaScript同时有两个线程，一个线程中执行在某个DOM节点上添加内容，另一个线程执行删除这个节点，这时浏览器
会一脸懵逼。

所以JavaScript的单线程是这门语言的核心，未来也不会改变。  

有人说，那HTML5的新特性`Web Worker`，可以创建多线程呀～

是的，为了解决不可避免的耗时操作(多重循环、复杂的运算)，HTML5提出了`Web Worker`，它会在当前的js执行主线程中开辟出一个额外的线程来运行js文件，这个新的线程和js主线程之间不会互相影响，同时提供了数据交换的接口：`postMessage`和`onMessage`

但是因为它创建的子线程完全受控于主线程，且位于外部文件中，无法访问DOM。所以它并没有改变js单线程的本质。


单线程就意味着，所有的任务都需要排队。

就像还不能自助点餐的时候你去肯德基需要排队，有的人没想好点什么或者点的东西很多，耗时就会长，那么后面的人也只好排队等待。有了自助点餐服务后，一切问题迎刃而解。


语言的设计和生活中的现实情况很像，IO设备(输入输出)很慢(比如Ajax)，那么语言的设计者意识到这一点，就在主线程中挂起处于等待中的任务，先运行后面的任务，等IO设备有了结果，再把挂起的任务执行下去。

## Event Loop 

![event loop](/images/js-eventloop0.jpg)

从上图中我们可以看到，在主线程运行时，会产生堆(heap)和栈(stack)。

堆中存的是我们声明的object类型的数据，栈中存的是基本数据类型以及函数执行时的运行空间。

栈中的代码会调用各种外部API，它们在任务队列中加入各种事件(onClick,onLoad,onDone)，只要栈中的代码执行完毕(js引擎存在`monitoring process`进程，会持续不断的检查主线程执行栈是否为空)，主线程就回去读取任务队列，在按顺序执行这些事件对应的回调函数。

也就是说主线程从任务队列中读取事件，这个过程是循环不断的，所以这种运行机制又成为`Event Loop`(事件循环)。

## 同步任务和异步任务

我们可以将任务分为同步任务和异步任务。

同步任务就是在主线程上排队执行的任务，只能执行完一个再执行下一个。

异步任务则不进入主线程，而是先在event table中注册函数，当满足触发条件后，才可以进入任务队列来执行。只有任务队列通知主线程说，我这边异步任务可以执行了，这个时候此任务才会进入主线程执行。

举个🌰

```javascript
console.log(a);

setTimeout(
  function () {
      console.log(b);
  },1000)
  
console.log(c)  

// a
// c
// b
```
1.console.log(a)是同步任务，进入主线程执行，打印a。

2.setTimeout是异步任务，先被放入event table中注册，1000ms之后进入任务队列。

3.console.log(c)是同步任务，进入主线程执行，打印c。

当a，c被打印后，主线程去事件队列中找到setTimeout里的函数，并执行，打印b。

综上所述，b最持久～(扯个🥚)

## 宏任务和微任务

**本文的`Macrotask`在`WHATWG` 中叫`task`。`Macrotask`为了便于理解，并没有实际的出处。**

同步任务和异步任务的划分其实并不准确，准确的分类方式是宏任务(Macrotask)和微任务(Microtask)。

宏任务包括：`script(整体代码)`, `setTimeout`, `setInterval`, `requestAnimationFrame`, `I/O`,`setImmediate`。

其中`setImmediate`只存在于Node中，`requestAnimationFrame`只存在于浏览器中。


微任务包括： `Promise`, `Object.observe`(已废弃), `MutationObserver`(html5新特性)，`process.nextTick`。

其中`process.nextTick`只存在于Node中，`MutationObserver`只存在于浏览器中。

**注意：**

`UI Rendering`不属于宏任务，也不属于微任务，它是一个与微任务平行的一个操作步骤。
[HTML规范文档](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model)

这种分类的执行方式就是，执行一个宏任务，过程中遇到微任务时，将其放到微任务的事件队列里，当前宏任务执行完成后，会查看微任务的事件队列，依次执行里面的微任务。如果还有宏任务的话，再重新开启宏任务……

![event loop](/images/js-eventloop1.jpg)

再举个🌰
```javascript
setTimeout(function() {
	console.log('a')
});

new Promise(function(resolve) {
	console.log('b');

	for(var i =0; i <10000; i++) {
		i ==99 && resolve();
	}
}).then(function() {
	console.log('c')
});

console.log('d');

// b
// d
// c
// a
```
1.首先执行`script`下的宏任务，遇到`setTimeout`，将其放入宏任务的队列里。

2.遇到`Promise`，`new Promise`直接执行，打印b。

3.遇到`then`方法，是微任务，将其放到微任务的队列里。

4.遇到`console.log('d')`，直接打印。

5.本轮宏任务执行完毕，查看微任务，发现`then`方法里的函数，打印c。

6.本轮`event loop`全部完成。

7.下一轮循环，先执行宏任务，发现宏任务队列中有一个`setTimeout`，打印a。

综上所述，不要说a是最持久的，如果你认为你彻底明白了，给你出道题，看看下面的代码中，谁最持久？

```javascript
console.log('a');

setTimeout(function() {
    console.log('b');
    process.nextTick(function() {
        console.log('c');
    })
    new Promise(function(resolve) {
        console.log('d');
        resolve();
    }).then(function() {
        console.log('e')
    })
})
process.nextTick(function() {
    console.log('f');
})
new Promise(function(resolve) {
    console.log('g');
    resolve();
}).then(function() {
    console.log('h')
})

setTimeout(function() {
    console.log('i');
    process.nextTick(function() {
        console.log('j');
    })
    new Promise(function(resolve) {
        console.log('k');
        resolve();
    }).then(function() {
        console.log('l')
    })
})
```

好，不要怂，我们来逐步分析。

**第一轮事件循环：**

1.第一个宏任务(整体script)进入主线程，`console.log('a')`，打印a。

2.遇到`setTimeout`，其回调函数进入宏任务队列，暂定义为`setTimeout1`。

3.遇到`process.nextTick()`，其回调函数被分发到微任务队列，暂定义为`process1`。

4.遇到`Promise`，`new Promise`直接执行，打印g。`then`进入微任务队列，暂定义为`then1`。

5.遇到`setTimeout`，其回调函数进入宏任务队列，暂定义为`setTimeout2`。

此时我们看一下两个任务队列中的情况

宏任务队列 | 微任务队列
:----------: | :-----:
setTimeout1、setTimeout2|process1、then1

第一轮宏任务执行完毕，打印出a和g。

查找微任务队列中有`process1`和`then1`。全部执行，打印f和h。

第一轮事件循环完毕，打印出a、g、f和h。

**第二轮事件循环：**

1.从`setTimeout1`宏任务开始，首先是`console.lob('b')`，打印b。

2.遇到`process.nextTick()`，进入微任务队列，暂定义为`process2`。

3.`new Promise`直接执行，输出d，`then`进入微任务队列，暂定义为`then2`。

此时两个任务队列中

宏任务队列 | 微任务队列
:----------: | :-----:
setTimeout2| process2、 then2


第二轮宏任务执行完毕，打印出b和d。

查找微任务队列中有`process2`和`then2`。全部执行，打印c和e。

第二轮事件循环完毕，打印出b、d、c和e。

**第三轮事件循环**

1.执行`setTimeout2`，遇到`console.log('i')`，打印i。

2.遇到`process.nextTick()`，进入微任务队列，暂定义为`process3`。

3.`new Promise`直接执行，打印k。

4.`then`进入微任务队列，暂定义为`then3`。

此时两个任务队列中

宏任务队列：空

微任务队列：`process3`、`then3`

第三轮宏任务执行完毕，打印出i和k。

查找微任务队列中有`process3`和`then3`。全部执行，打印j和l。

第三轮事件循环完毕，打印出i、k、j和l。

到此为止，三轮事件循环完毕，最终输出结果为：

```
a、g、f、h、b、d、c、e、i、k、j、l
```
l最持久，你答对了吗？

以上代码仅在浏览器环境中执行顺序如下，`node`环境下可能存在不同。

看完本文希望你能够理解JavaScript引擎的`Event Loop`执行机制，不仅可以让我们更加深刻的认识JavaScript这门语言，而且面试被问起的时候可以和面试官侃侃而谈。

