
# JavaScript设计模式(三)


想要成为一名优秀的软件开发人员，要具有强烈的工匠精神。遇到问题，想办法解决问题。大多数人在前期更多的依靠的是激情，中后期则需要一定的耐心。

工作时间久了，自然对软件系统产生自己的思考，还会面临职业生涯的一个挑战。要不要成为一个技术负责人？


技术负责人当然要从更大的角度来考虑问题，那么设计模式便是一门必修课，学习设计模式不仅能够在日常的业务代码中给我们提供解决问题的思路。在架构上的设计也无处不见设计模式的思想。

这是一件高收益值、长半衰期的事，值得你坚持下去。

话不多说，现在开饭！

本文所讲述的是`行为型设计模式`，包括`模板方法模式`、`观察者模式`、`状态模式`、`策略模式`、`职责链模式`、`命令模式`、`访问者模式`、`中介者模式`、`备忘录模式`、`迭代器模式`、`解释器模式`。

## 模板方法模式 Template Method
**概念：父类中定义一组操作算法骨架，而将一些实现步骤延迟到子类中，使得子类可以不改变父类的算法结构的同时可重新定义算法中某些实现步骤。**

核心在于对方法的重用，将核心方法封装在基类中，让子类继承基类的方法，实现基类方法的共享，达到方法公用。

```javascript
class canteen {
  eat () {
    eat1();
    eat2();
    eat3();
  }
  eat1 () {
    console.log('海底捞');
  }
  eat2 () {
    console.log('日本料理');
  }
  eat3 () {
    console.log('烧烤');
  }
}
```
应用实例：

1、“把大象放冰箱需要三步”是一个顶层逻辑，具体实现可能因冰箱大小有所不同。

2、泡咖啡和泡茶。

3、`VS Code`、`Webstorm`等一些IDE中的模板代码片段功能。

4、蛋糕模具

## 观察者模式 Observer
**概念：又被称作发布-订阅模式或消息机制，定义了一种依赖关系，解决了主体对象与观察者之间功能的耦合。**

进一步说，观察者模式定义了对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

应用实例： 

1.拍卖的时候，拍卖师是观察者观察最高竞价，然后通知给其他竞价者竞价。

2.在`Nodejs`中通过`EventEmitter`实现了原生的对于这一模式的支持。

3.`jquery`中的`trigger/on`。

4.`MVVM`。

5.微信公众号推送文章。

拿微信公众号来举个例子吧，你们(不同的观察者)关注了前端食堂(一个主体)，当公众号文章更新时(主体更新)，你们都能收到我的推送(观察者)。


```javascript
// Subject是一个公众号主体，需要维护订阅的观察者数组。
// new Subject()创建一个新的公众号
// addObserver 添加观察者
// removeObserver 删除观察者
// inform 通知观察者
function Subject() {
  this.observers = [];
}
Subject.prototype.addObserver = function (observer) {
  this.observers.push(observer);
}
Subject.prototype.removeObserver = function (observer) {
  let index = this.observers.indexOf(observer);
  if (index > -1) {
    this.observers.splice(index, 1);
  }
}
Subject.prototype.inForm = function () {
  this.observers.forEach(observer=>{
    observer.update();
  })
}
function Observer (name) {
  this.name = name;
  this.update = function () {
    console.log(name + 'update...');
  }
}

let subject = new Subject();
let observer1 = new Observer('fans1');
subject.addObserver(observer1);
let observer2 = new Observer('fans2');
subject.addObserver(observer2);
subject.inForm();

// fans1 update...
// fans2 update...
```
```javascript
// 下面是ES6的写法，感谢关注前端食堂的小婊，咳咳，小宝贝们～
class Subject {
  constructor () {
    this.observers = [];
  }
  addObserver (observer) {
    this.observers.push(observer);
  }
  removeObserver (observer) {
    let index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }
  inForm () {
    this.observers.forEach(observer=> {
      observer.update();
    })
  }
}
class Observer {
  constructor (name) {
    this.name = name;
    this.update = function () {
      console.log(name + '么么...');
    }
  }
}
let subject = new Subject();
let observer1 = new Observer('baby1');
subject.addObserver(observer1);
let observer2 = new Observer('baby2');
subject.addObserver(observer2);
subject.inForm();

// baby1 么么...
// baby2 么么...
```
## 状态模式 State
**概念：当一个对象的内部状态发生改变时，会导致其行为的改变，这看起来像是改变了对象。**

状态模式的目的是简化分支判断流程。

应用实例：

1.LOL中英雄技能造成的各种状态如沉默、减速、禁锢、击飞等。

2.文件上传有扫描、正在上传、暂停、上传成功、上传失败等状态。

```javascript
class HappyYasuo {
  constructor () {
    this._currentState = [];
    this.states = {
      q_key () {console.log('斩钢闪')},
      w_key () {console.log('风之障壁')},
      e_key () {console.log('踏前斩')},
      r_key () {console.log('狂风绝息斩')}
    }
  }

  change (arr) {  // 更改当前动作
    this._currentState = arr;
    return this;
  }

  go () {
    console.log('触发动作');
    this._currentState.forEach(T => this.states[T] && this.states[T]());
    return this;
  }
}

new HappyYasuo()
    .change(['q_key', 'w_key'])
    .go()                      // 触发动作  斩钢闪  风之障壁
    .go()                      // 触发动作  斩钢闪  风之障壁
    .change(['e_key'])
    .go()                      // 触发动作  踏前斩
```

![yasuo](/images/js-desin0.jpg)


## 策略模式 Strategy
**概念：将定义的一组算法封装起来，使其相互之间可以替换。封装的算法具有一定独立性，不会随客户端变化而变化。**


应用实例： 

1.诸葛亮的锦囊妙计，每一个锦囊就是一个策略。 

2.旅行的出游方式，选择骑自行车、坐汽车，每一种旅行方式都是一个策略。 

```javascript
class ZhaoYun {
  constructor (type) {
    this.type = type;
  }
  plan () {
    if (this.type === 'first') {
      console.log('打开第一个锦囊');
    } else if (this.type === 'second') {
      console.log('打开第二个锦囊');
    } else if (this.type === 'third') {
      console.log('打开第三个锦囊');
    }
  }
}
var u1 = new ZhaoYun('first');
u1.plan();
var u2 = new ZhaoYun('second');
u2.plan();
var u3 = new ZhaoYun('third');
u3.plan();
```

```javascript
class FirstStep {
  plan () {
    console.log('打开第一个锦囊');
  }
}
class SecondStep {
  plan () {
    console.log('打开第二个锦囊');
  }
}
class ThirdStep {
  plan () {
    console.log('打开第三个锦囊');
  }
}

var u1 = new FirstStep();
u1.plan();
var u2 = new SecondStep();
u2.plan();
var u3 = new ThirdStep();
u3.plan();
```

## 职责链模式 Chain of Responsibility
**概念：解决请求的发送者与请求的接受者之间的耦合，通过指责链上的多个对象对分解请求流程，实现请求在多个对象之间的传递，直到最后一个对象完成请求的处理。**


应用实例： 

1.`JavaScript`中的事件冒泡。 

2.长安十二时辰中的望楼传信。

```javascript
// 请假审批，需要组长审批、经理审批、最后总监审批
class Action {
  constructor (name) {
    this.name = name;
    this.nextAction = null;
  }
  setNextAction (action) {
    this.nextAction = action;
  }
  handle () {
    console.log(`${this.name} 审批`)
    if (this.nextAction != null) {
      this.nextAction.handle();
    }
  }
}

let a1 = new Action('组长');
let a2 = new Action('经理');
let a3 = new Action('总监');
a1.setNextAction(a2);
a2.setNextAction(a3);
a1.handle();
```

## 命令模式 Command
**概念：将请求与实现解耦并封装成独立对象，从而使不同的请求对客户端的实现参数化。**

也就是执行命令时，将发布者和执行者分开,在中间加入命令对象，作为中转站。

应用实例：

命令模式的实例在生活中很常见，这里我们只举一个例子，留给大家想象的空间。

我们去饭店点菜(发布者)，无须知道厨师的名字(执行者)，只需告诉服务员即可(中转站)。

```javascript
// 接受者 厨师
class Receiver {
  go () {
    console.log('接受者执行请求');
  }
}
// 命令对象 服务员
class Command {
  constructor (receiver) {
    this.receiver = receiver;
  }
  action () {
    console.log('命令对象->接受者->对应接口执行');
    this.recerver.go();
  }
}
// 发布者 顾客吃饭点菜
class Invoker {
  constructor (command) {
    this.command = command;
  }
  invoke () {
    console.log('发布者发布请求');
    this.command.action();
  }
}

let cook = new Receiver();
let waiter = new Command(cook);
let Diner = new Invoker(waiter);
Diner.invoke();
```
## 访问者模式 Visitor
**概念：针对于对象结构中的元素，定义在不改变该对象的前提下访问结构中元素的新方法**。

访问者模式就是将数据操作和数据结构进行分离。

应用实例：

您在朋友家做客，您是访问者，朋友接受您的访问，您通过朋友的描述，然后对朋友的描述做出一个判断，这就是访问者模式。


使用访问者模式，我们可以使对象拥有像数组一样的`push`、`pop`和`splice`方法。
```javascript
var Visitor = (function() {
      return {
        splice: function(){
          // splice方法参数，从原参数的第二个参数开始算起
          var args = Array.prototype.splice.call(arguments, 1);
          // 对第一个参数对象执行splice方法
          return Array.prototype.splice.apply(arguments[0], args);
        },
        push: function(){
          // 强化类数组对象，使他拥有length属性
          var len = arguments[0].length || 0;
          // 添加的数据从原参数的第二个参数算起
          var args = this.splice(arguments, 1);
          // 校正length属性
          arguments[0].length = len + arguments.length - 1;
          // 对第一个参数对象执行push方法
          return Array.prototype.push.apply(arguments[0], args);
        },
        pop: function(){
          // 对第一个参数对象执行pop方法
          return Array.prototype.pop.apply(arguments[0]);
        }
      }
    })();
```
## 中介者模式 Mediator
**概念：通过中介者对象封装一系列对象之间的交互，使对象之间不再相互引用，降低他们之间的耦合。**


应用实例： 

1.中国加入 WTO 之前是各个国家相互贸易，结构复杂，现在是各个国家通过 WTO 来互相贸易。

2.机场调度系统。 

3.MVC 框架，其中C（控制器）就是 M（模型）和 V（视图）的中介者。


```javascript
class Mediator {
  constructor (a, b) {
    this.a = a;
    this.b = b;
  }
  setA () {
    let number = this.b.number;
    this.a.setNumber(number * 100);
  }
  setB () {
    let number = this.a.number;
    this.b.setNumber(number / 100);
  }
}

class A {
  constructor () {
    this.number = 0;
  }
  setNumber (num, m) {
    this.number = num;
    if (m) {
      m.setB();
    }
  }
}

class B {
  constructor () {
    this.number = 0;
  }
  setNumber (num, m) {
    this.number = num;
    if (m) {
      m.setA();
    }
  }
}
```
## 备忘录模式 Memento
**概念：在不破坏对象的封装性的前提下，在对象之外捕获并保存该对象内部的状态以便日后对象使用或者对象恢复到以前的某个状态。**


应用实例：

分页组件的时候，点击下一页获取新的数据，但是当点击上一页时，又重新获取数据，造成无谓的流量浪费，这时可以对数据进行缓存。

```javascript
// Memento 备忘录对象 只供原发器使用 提供状态提取方法
var Memento = function(state){
    var _state = state;
    this.getState = function(){
        return _state;
    }
}
// Originator 原发器
var Originator = function(){
    var _state;
    this.setState = function(state){
        _state = state;
    }
    this.getState = function(){
        return _state;
    }
    // saveStateToMemento 创建一个备忘录 存储当前状态
    this.saveStateToMemento = function(){
        return new Memento(_state);
    }
    this.getStateFromMemento = function(memento){
        _state = memento.getState();
    }
}
// 负责人 负责保存备忘录 但是不能对备忘录内容进行操作或检查
var CareTaker = function(){
    var _mementoList = [];
    this.add = function(memento){
        _mementoList.push(memento);
    }
    this.get = function(index){
        return _mementoList[index];
    }
}
 
var originator = new Originator();
var careTaker = new CareTaker();
originator.setState("State 1");
originator.setState("State 2");
careTaker.add(originator.saveStateToMemento());
originator.setState("State 3");
careTaker.add(originator.saveStateToMemento());
originator.setState("State 4");
 
console.log("当前状态: " + originator.getState());
// 当前状态: State 4
originator.getStateFromMemento(careTaker.get(0));
console.log("恢复第一次保存状态: " + originator.getState());
// 恢复第一次保存状态: State 2
originator.getStateFromMemento(careTaker.get(1));
console.log("恢复第二次保存: " + originator.getState());
// 恢复第二次保存: State 3
```
局限性：备忘录模式对资源的消耗过大。
## 迭代器模式 Iterator
**概念：在不暴露对象内部结构的同时，可以顺序地访问聚合对象内部的元素。**

这种模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示。

应用实例：

绝大部分语言内置了迭代器。

1.`JavaScript的Array.prototype.forEach`

2.`jQuery中$.each`
```javascript
$.each([1, 2, 3], function(i ,n){
  console.log('当前下标为： '+ i,'当前值为:' + n);
})
// 下标： 0 当前值:1 
// 下标： 1 当前值:2  
// 下标： 2 当前值:3
```

[还有ES6中的Iterator](http://es6.ruanyifeng.com/#docs/iterator)

## 解释器模式 Interpreter
**概念：对于一种语言，给出其文法表示形式，并定义一种解释器，通过使用这些解释器来解释语言中定义的句子。**

这种模式实现了一个表达式接口，该接口解释一个特定的上下文。

应用实例：

1.编译器

2.正则表达式  



参考：

《JavaScript设计模式》张容铭
