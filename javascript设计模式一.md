# JavaScript设计模式(一)


在开始阅读本文之前，大家可以先去看一下这个问题[前端未来几年的发展方向是什么？](https://www.zhihu.com/question/286700472/answer/464694260)


有一个简单的大局观，造完了火箭，再回归正文，我们的日常生活和工作中的大部分还是需要脚踏实地搬砖的，为了应对不断变换的需求，为了不加班，掌握设计模式的思想可以大大提高我们的搬砖效率。

其实无论前端怎么发展，设计模式就在那里，不悲不喜…………


本文将会介绍`创建型设计模式`，包括`简单工厂模式`、`工厂方法模式`、`抽象工厂模式`、`建造者模式`、`原型模式`和`单例模式`。

首先来介绍工厂模式，它包括上面说的前三种。

## 简单工厂模式 Simple Factory
它的核心就是创建对象，对不同的类进行实例化。

我们直接上代码🌰。

```javascript
// 可爱萝莉
var lovelyGirl = function () {
  this.slogan = '性感在可爱面前一文不值';
}
lovelyGirl.prototype = {
  sayHi : function () {
    console.log('你好呀，小哥哥');
  },
  habit : function () {
    console.log('洛丽塔');
  }
}
// 性感女郎
var sexyLady = function () {
  this.slogan = '可爱在性感面前一文不值';
}
sexyLady.prototype = {
  sayHi : function () {
    console.log('嗨！帅哥');
  },
  habit : function () {
    console.log('黑丝');
  }
}
// 美女工厂
var lookerFactory = function (name) {
  switch (name) {
    case 'lovely':
      return new lovelyGirl();
    case 'sexy':
      return new sexyLady();
  }
}

var marilynMonroe = lookerFactory('sexy'); 
console.log(marilynMonroe);                // sexyLady {slogan: "可爱在性感面前一文不值"}
console.log(marilynMonroe.slogan);         // 可爱在性感面前一文不值    
marilynMonroe.sayHi();                     // 嗨！帅哥                   
marilynMonroe.habit();                     // 黑丝
```

我们也可以通过简单工厂模式提取相似的东西，对不相似的地方进行单独的处理。

```javascript
// 动物工厂
function createAnimals (name,sound,color) {
  var o = new Object();
  o.name = name;
  o.sound = sound;
  o.color = color;
  o.say = function () {
    console.log(this.sound);
  };
  return o;
}

var animals1 = createAnimals("cat","喵喵喵","yellow");
var animals2 = createAnimals("dog","汪汪汪","white");

animals1.say();  // 喵喵喵
animals2.say();  // 汪汪汪
```

美女工厂为男同胞们提供，动物工厂为小姐姐们提供，我很照顾我的女粉丝的。

接着对比一下两种工厂的区别。

第一种是通过类实例化对象进行创建的。

第二种是通过创建一个新对象，然后对其属性和方法进行包装来实现的。

这种区别就会造成在第一种工厂中，继承同一父类的类是可以共用他们父类原型上的方法的。但第二种工厂中创建的对象都是独立的个体，就不可以共用。

### 局限性
使用场合单一，通常用于创建逻辑不复杂的单一对象，或是创建对象数量比较少的情况。

## 工厂方法模式 Factory Method
那么如果我们想要创建多类产品怎么办？

工厂方法模式就是提供给我们的解决方案。

```javascript
// 安全模式创建的工厂类 屏蔽new关键字可能造成的错误
var Factory = function (type, content) {
    // 判断当前对象this是不是指向Factory
    if (this instanceof Factory) {
        var s = new this[type](content);
        return s;
    } else {
        return new Factory(type, content);
    }
}
// 在原型中设置创建对象的基类
Factory.prototype = {
    Html : function (content) {
        //...
    },
    Css : function (content) {
        //...
    },
    JavaScript : function (content) {
        //...
    }
}
```
假如数据是这样的
```json
var data = [
    {type:'Html', content:'我会Html'},
    {type:'Css', content:'我会Css'},
    {type:'JavaScript', content:'我会JavaScript'}
]
```
使用
```javascript
for (var i = 2; i >= 0; i--) {
    Factory(s[i].type, s[i].content);
}
```

当需求变化，想要添加其他类时，只需在`Factory`的原型中添加就行了，妈妈再也不用担心我的学习。

工厂方法将对象类和使用者之间的关系进行解耦，使用者无需关心创建对象的具体类，直接调用工厂方法就行。

## 抽象工厂模式 Abstract Factory
`JavaScript`中`abstract`是一个保留字，不能像传统面向对象语言那样轻松的创建抽象类。

而抽象类是一种声明但不能使用的类，当你使用时就会报错，但是因为`JavaScript`的灵活性，我们可以在类的方法中手动的抛出错误来模拟抽象类。


我们也不能用它来创建一个真实的对象，而是用它作为父类来创建一些子类，从而实现子类继承父类的方法。

```javascript
var CarFactory = function (subType, superType) {
  // 判断抽象工厂中是否有该抽象类
  if (typeof CarFactory[superType] === 'function') {
    // 缓存类
    function F () {}
    // 继承父类属性和方法
    F.prototype = new CarFactory[superType]();
    // 将子类constructor指向子类
    subType.constructor = subType;
    // 子类原型继承“父类”
    subType.prototype = new F();
  } else {
    // 不存在该抽象类抛出错误
    throw new Error('未创建该抽象类');
  }
}
// 汽车抽象类
CarFactory.Car = function () {
  this.type = 'car';
}
CarFactory.Car.prototype = {
  getPrice : function () {
    return new Error('抽象方法不能调用');
  },
  getSpeed : function () {
    return new Error('抽象方法不能调用');
  }
}
// 布加迪威龙汽车子类
var BuggatiVeyron = function (price, speed) {
  this.price = price;
  this.speed = speed;
}
// 抽象工厂实现对Car抽象类的继承
CarFactory(BuggatiVeyron, 'Car');
BuggatiVeyron.prototype.getPrice = function () {
  return this.price;
}
BuggatiVeyron.prototype.getSpeed = function () {
  return this.speed;
}

var testCar = new BuggatiVeyron(999999999, 1000);
console.log(testCar.getPrice());   // 999999999
console.log(testCar.type);         // car
```

通过抽象，我们可以知道子类属于哪一种类别，同时他们也拥有了该类的属性和方法。

这就是抽象工厂模式，它创建出的结果不是真实的对象实例，而是一个类簇，里面制定了类的结构。


### 局限性
由于JavaScript中不支持抽象化创建与虚拟方法，所以导致这种模式不能像其他面向对象语言中应用的那么广泛。

### 小结
工厂模式的核心作用：

1.主要用于隐藏实例的复杂度，只需对外提供一个接口

2.实现构造函数和创建者的分离，满足开放封闭的原则

## 建造者模式 Builder

有些人做事在乎结果，有些人则更加喜欢充满想象和细节的过程。


工厂模式主要的在乎的是最终的输出，但建造者模式的关注点则是过程。

建造者模式也就是将一个复杂对象的构建层与它的表示层相互分离解耦。

```javascript
    // 创建一个类
   var Human = function (param) {
       // 技能
       this.skill = param && param.skill || '保密';
       // 兴趣爱好
       this.hobby = param && param.hobby || '保密';
   }
   // 类原型方法
   Human.prototype = {
       getSkill : function () {
           return this.skill;
       },
       getHobby : function () {
           return this.hobby;
       }
   }
  // 实例化姓名类
   var Names = function (name) {
       var that = this;
       // 构造器
       // 构造函数解析姓名和姓与名
       (function(name, that){
           that.wholeName = name;
           if(name.indexOf(' ') > -1){
               that.FirstName = name.slice(0,name.indexOf(' '));
               that.secondName = name.slice(name.indexOf(' '));
           }
       })(name, that);
   }
  // 实例化职位类
   var Work = function (work) {
       var that = this;
       // 构造器
       // 构造函数中通过传入的职位特征来设置响应职位以及描述
       (function(work, that){
           switch (work) {
               case 'frontEndEngineer':
                 that.work = '前端工程师';
                 that.workDescript = '后端说我们是打辅助的？';
                 break;
               case 'UI':
                 that.work = '设计师';
                 that.workDescript = '一个像素的执着';
                 break;
               case 'secretary':
                 that.work = '女秘书';
                 that.workDescript = '了解老板才是第一准则';
                 break;
               default:
                 that.work = work;
                 that.workDescript = '对不起，我们还不清楚您所选择职位的相关描述';
           }
       })(work, that);
   }
   // 更换期望的职位
   Work.prototype.changeWork = function (work) {
       this.work = work;
   }
   // 添加对职位的描述
   Work.prototype.changeDescript = function (setence) {
       this.workDescript = setence;
   }

   /****
   * 应聘者建造者
   * 参数 name: 姓名(全名)
   * 参数 work: 期望职位
   **/
   var Person = function (name, work) {
           var _person = new Human();
           _person.name = new Names(name);
           _person.work = new Work(work);
           return _person;
   }

   var person = new Person('xiao fang', 'secretary');
   console.log(person.skill);                       
   console.log(person.name.FirstName);
   console.log(person.work.work);
   console.log(person.work.workDescript);
   person.work.changeDescript('更改一下职位描述！');
   console.log(person.work.workDescript);

   // 保密
   // xiao
   // 女秘书
   // 了解老板才是第一准则
   // 更改一下职位描述！
```


### 局限性
这种模式对于整体对象类的拆分也产生了一定的副作用，增加了结构的复杂性，在实际应用中要结合模块间的复用率来进行合理的应用。

## 原型模式 Prototype
毋庸置疑，原型模式是JavaScript的灵魂。

在实际应用中，对于每次创建的一些简单而又差异化的属性我们可以放在构造函数中，将一些消耗资源比较大的方法放在原型中。

你也可以在任何时候对原型上的方法进行扩展，无论是基类还是子类，且所有被实例化的对象都能获取这些方法，可以给我们不设限的自由。

```javascript
var Person = function (name, age) {
    this.name = name;
    this.age = age;
}

Person.prototype = {
    basketball : function () {
        console.log('我会打篮球');
    },
    rap : function () {
        console.log('我会rap');
    }
}

var Man = function (name, age) {
    Person.call(this, name, age);
}
Man.prototype = new Person();
// 个性化修改
Man.prototype.rap = function () {
    console.log('yoyoyo');
}
// 拓展
Man.prototype.sing = function () {
    console.log('ooooo');
}

var man = new Man('kunkun', 18);
man.rap();               // yoyoyo
man.sing();              // ooooo
console.log(man.name)    // kunkun
```



## 单例模式 Singleton
单例模式是只允许实例化一次的对象类，保证了一个类仅有一个实例，并提供了全局访问。

孤独的单例模式正如同此时此刻孤独的你。。

去冰箱拿吃的了吧，冰箱没有的去美团点外卖，我等你回来接着看。

它也被称作单体模式，可以用一个对象来规划一个命名空间，更好的管理对象上的属性与方法。

```javascript
Var T = {
    Ajax : {
      get : function () {},
      post : function () {}
    },
    Utils : {
      utils_method1 : function () {},
      utils_method2 : function () {}
    }
    Others : {
        // ...
    }
    // ...
}

T.Ajax.get();
T.Utils.utils_method1();
```

结构清晰，一目了然，使用起来也很舒服。

除此之外，单例模式还可以管理静态变量。

JavaScript中没有静态变量，但是我们可以通过将变量放到一个函数内部，规定它只能通过特权方法来访问，不提供赋值变量的方法，只提供获取变量的方法，就可以做到限制变量的修改并且可以供外界访问。

```javascript
var State = (function(){
    // 私有变量
    var state = {
        FRIEND : 0
    }
    // 返回取值器对象
    return {
        // 取值器方法
        get : function (name) {
            return state[name] ? state[name] : null;
        }
    }
})

var result = State.get('COUNT');
console.log(result);   // 0
```

对于某些特定场景需要对单例对象延迟创建，也就是需要用它的时候才创建。

```javascript
var LazySingle = (function(){
    var _instance = null;
    function Single () {
        return {
            publicMethod : function () {},
            publicProperty : '1.0'
        }
    }
    
    return function () {
        if (!_instance) {
            _instance = Single();
        }
        return _instance;
    }
})();

console.log(LazySingle().publicProperty); // 1.0
```

单例模式的应用场景是比较广泛的，比如jQuery库、登陆的弹窗、Vuex和Redux中的store、websocket连接、数据库连接池等等。



参考：

《JavaScript设计模式》张容铭
