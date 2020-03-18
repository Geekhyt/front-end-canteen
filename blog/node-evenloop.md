# Node.js事件循环

从源码角度探求Node.js的事件循环本质。

## 事件循环


![事件循环](/images/node0.png)


事件循环的执行顺序从图中可以看出，每次的事件循环都包含了上图中的6个阶段，接下来我们来一一解读它们。

### timers 定时器

计时器分为两类：

- Immediate 在下一个check阶段执行
- Timeout 定时器过期后执行(delay参数默认值为1ms)

Timeout计时器又有两种类型：
- Interval
- Timeout

`这个阶段会执行setTimeout()和setInterval()设定的回调`

`timers的执行是由poll阶段控制的`

setTimeout()和setInterval()和浏览器中的API是相同的。它们的实现原理与异步I/O比较类似，但是不需要I/O线程池的参与。

这两个定时器创建后会被插入到定时器观察者内部的一个红黑树中。每次Tick执行时，都会从红黑树中取出定时器对象，来检查它们是否超过定时时间，超过便执行它们的回调。

注意：定时器存在一个问题，就是它不是绝对精确的(在容忍范围内)。一旦某个事件循环中，有一个任务占用了较多的时间，那么再次轮到定时器执行时，时间就会受到影响。

#### 无IO处理情况

```javascript
setTimeout(function timeout () {
    console.log('timeout');
},0);
setImmediate(function immediate () {
    console.log('immediate');
});
```
通过执行上面的代码我们可以发现，输出结果是不确定的。

因为setTimeout（fn, 0)具有几毫秒的不确定性，无法保证进入timers阶段，定时器能立即执行处理程序。

#### 有IO处理情况
```javascript
var fs = require('fs');
fs.readFile(__filename, () => {
    setTimeout(() => {
        console.log('timeout');
    }, 0);
    setImmediate(() => {
        console.log('immediate');
    });
})
// immediate
// timeout
```
此时setImmediate优先于setTimeout执行，因为poll阶段执行完成后进入check阶段，而timers阶段则处于下一个事件循环阶段了。

### pending callbacks 待定回调

`执行大部分回调，除了close，times和setImmediate()设定的回调`

### idle，perpare

`仅供内部使用`

### poll 轮询
`获取新的I/O事件，在适当的条件下，Node.js会在这里阻塞`

`这个阶段的主要任务是执行到达delay时间的timers定时器的回调，并且处理poll队列里的事件。`

当事件循环进入poll阶段，并且没有调用定时器时，将会发生以下两种情况：

1.如果poll队列不为空，事件循环将遍历同步执行它们的回调队列。

2.如果poll队列为空，又分为两种情况：

- 如果被setImmediate()回调调用，事件循环会结束poll阶段，进入到check阶段。

- 如果没有被setImmediate()回调调用，事件循环将阻塞并等待回调添加到poll队列中执行。

一旦poll队列为空，事件循环将查看计时器是否到达delay时间，如果一个或多个定时器已达到delay时间，事件循环将回滚到timers定时器阶段，执行它们的回调。


### check 检测
`setImmediate()设定的回调会在这一阶段执行`

如同上文poll阶段的第二种情况中，如果poll队列为空，并且被setImmediate()回调调用，事件循环将直接进入check阶段。


### close callbacks 关闭的回调函数
`socket.on('close',callback)的回调会在这个阶段执行`


## libuv

`libuv为Node.js提供了整个事件循环功能。`

![libuv](/images/node1.png)

如上图所示，在Windows下，事件循环基于`IOCP`创建，在linux下通过`epoll`实现，FreeBSD下通过`kqueue`实现，在Solaris下通过`Event ports`实现。


我们再细心的去看上图，Network I/O和file I/O、DNS等实现方式是被分隔开的，这是因为他们的本质是由两套机制来实现的。我们一会儿来通过源码窥探它们的本质。


实质上，当我们写JavaScript代码去调用Node的核心模块时，核心模块会调用C++内建模块，内建模块通过libuv进行系统调用。


### libuv主要解决的问题

在现实世界中，在所有不同类型的操作系统平台下，支持不同类型的I/O是非常困难的。那么为了支持跨平台I/O的同时，能更好的管理整个流程，抽象出了libuv。

简单说，就是libuv抽象出一层API，可以帮助你调用各个平台和机器上各种系统特性，包括操作文件、监听socket等，而你不需要了解它们的具体实现。


### 核心源码解读

#### 核心函数uv_run

[源码](https://github.com/libuv/libuv/blob/v1.x/src/unix/core.c)

```c
int uv_run(uv_loop_t* loop, uv_run_mode mode) {
    int timeout;
    int r;
    int ran_pending;
    // 检查loop中是否有异步任务，没有就结束。
    r = uv__loop_alive(loop);
    if (!r)
      uv__update_time(loop);
    // 事件循环while
    while (r != 0 && loop->stop_flag == 0) {
        // 更新事件阶段
        uv__update_time(loop);  
        // 处理timer回调
        uv__run_timers(loop);        
        // 处理异步任务回调
        ran_pending = uv__run_pending(loop);      
        // 供内部使用
        uv__run_idle(loop);
        uv__run_prepare(loop);        
        // uv_backend_timeout计算完毕后，会传给uv__io_poll
        // 如果timeout = 0,则uv__io_poll会直接跳过
        timeout = 0;
        if ((mode == UV_RUN_ONCE && !ran_pending || mode == UV_RUN_DEFAULT))
          timeout = uv_backend_timeout(loop);
        uv__io_poll(loop, timeout);
        // check阶段
        uv__run_check(loop);
        // 关闭文件描述符等操作
        uv__run_closing_handles(loop);
        // 检查loop中是否有异步任务，没有就结束。
        r = uv__loop_alive(loop);
        if (mode == UV_RUN_ONCE || mode == UV_RUN_NOWAIT)
          break;
    }
    return r;
}
```
事件循环的真实面目是一个while。

上文说到Network I/O与file I/O、DNS等是由两套机制来实现的。

首先我们来看Network I/O，它最后的调用都会归结到`uv__io_start`这个函数，而该函数会将需要执行的I/O事件和回调放入watcher队列中，而`uv__io_poll`阶段会从watcher队列中取出事件调用系统的接口并执行。

(`uv__io_poll`部分的代码过长大家感兴趣可自行查看)
### uv__io_start
```c
void uv__io_start(uv_loop_t* loop, uv__io_t* w, unsigned int events) {
  assert(0 == (events & ~(POLLIN | POLLOUT | UV__POLLRDHUP | UV__POLLPRI)));
  assert(0 != events);
  assert(w->fd >= 0);
  assert(w->fd < INT_MAX);
  w->pevents |= events;
  maybe_resize(loop, w->fd + 1);
  if (w->events == w->pevents)
    return;
  if (QUEUE_EMPTY(&w->watcher_queue))
    QUEUE_INSERT_TAIL(&loop->watcher_queue, &w->watcher_queue);
  if (loop->watchers[w->fd] == NULL) {
    loop->watchers[w->fd] = w;
    loop->nfds++;
  }
}
```

如上所示就是我们libuv中Network I/O这条主线实现过程。

而另外一条主线是Fs I/O和DNS等操作则会调用`uv__work_sumit`这个函数，这个函数是执行线程池初始化`uv_queue_work`中最终调用的函数。
```c
void uv__work_submit(uv_loop_t* loop,
                     struct uv__work* w,
                     enum uv__work_kind kind,
                     void (*work)(struct uv__work* w),
                     void (*done)(struct uv__work* w, int status)) {
  uv_once(&once, init_once);
  w->loop = loop;
  w->work = work;
  w->done = done;
  post(&w->wq, kind);
}
```

```c
int uv_queue_work(uv_loop_t* loop,
                  uv_work_t* req,
                  uv_work_cb work_cb,
                  uv_after_work_cb after_work_cb) {
  if (work_cb == NULL)
    return UV_EINVAL;
  uv__req_init(loop, req, UV_WORK);
  req->loop = loop;
  req->work_cb = work_cb;
  req->after_work_cb = after_work_cb;
  uv__work_submit(loop,
                  &req->work_req,
                  UV__WORK_CPU,
                  uv__queue_work,
                  uv__queue_done);
  return 0;
}
```



## Node.js中的事件队列

Node.js中有多个队列，不同类型的事件在各自的队列中排队。在一个阶段结束后，进入下一个阶段之前，事件循环会在这中间处理中间队列。

原生的libuv事件循环中的队列主要又4种类型：

- 过期的定时器和间隔队列

- IO事件队列

- Immediates队列

- close handlers队列

除此之外，Node.js还有两个中间队列

- Next Ticks队列

- Other Microtasks队列

## Node.js与浏览器的Event Loop差异

我们可以回顾下浏览器中JavaScript事件循环，请移步我的另一篇系列专栏[浏览器中JavaScript的事件循环](/2019/07/javascript-evenloop/)

回来后，先说结论：

**在浏览器中，microtask的任务队列是每个macrotask执行完之后执行。**

**在Node.js中，microtask会在事件循环的各个阶段之间执行，也就是一个阶段执行完毕，就会去执行microtask队列的任务。**

(本文的Macrotask在WHATWG 中叫task。Macrotask为了便于理解，并没有实际的出处。)

相比于浏览器，node多出了`setImmediate(宏任务)`和`process.nextTick(微任务)`这两种异步操作。

`setImmediate`的回调函数被放在`check`阶段执行。而`process.nextTick`会被当做一种`microtask`，每个阶段结束后都会执行所有的`microtask`，你可以理解为`process.nextTick`可以**插队**，在下个阶段前执行。

#### process.nextTick插队带来的危害

process.nextTick的回调会导致事件循环无法进入到下一个阶段。I/O处理完成或者定时器过期后仍然无法执行。会让其他的事件处理程序处于饥饿状态，为了防止这个问题，Node.js提供了一个`process.maxTickDepth`(默认为1000)。

### Node.js中的微任务

- process.nextTick()
- Promise.then()

```javascript
Promise.resolve().then(function(){
    console.log('then')
})
process.nextTick(function(){
    console.log('nextTick')
});
// nextTick
// then
```

我们可以看到nextTick要早于then执行。

## Node.js v11变更的事件循环

从Node.js v11开始，事件循环的原理发生了变化，在同一个阶段中只要执行了macrotask就会立即执行microtask队列，与浏览器表现一致。具体请参考这个[pr](https://github.com/nodejs/node/pull/22842#discussion_r218142520)。

