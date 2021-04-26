---
title: "javascript-设计模式与开发实践二"
date: 2021-03-13T23:37:45+08:00
tags: 
  - 设计模式
---

> 设计模式

<a name="uDuhz"></a>
## 单例模式
单例模式的定义是：保证一个类只有一个实例，并提供一个访问它的全局访问点。<br />
<br />传统面向对象语言中，单例对象一般从“类”中创建而来。但 JavaScript 是一门无类（class-free）语言，创建对象的方法很简单，先创建一个“类”，其实很没必要。<br />
<br />按照上面其定义，虽然全局变量不是单例模式，但在 JavaScript 中我们经常会把全局变量当成单例来使用，但要留意命名空间污染，它很容易被覆盖。
```javascript
var a = {};
```
<a name="zpqt6"></a>
### 惰性单例
惰性单例指的是在需要的时候才创建对象实例，是单例模式的重点。
```javascript
var createDiv = (function () {
  var div
  return function () {
    if (!div) {
      div = document.createElement('div')
      div.style.display = 'none'
      document.body.appendChild(div)
    }

    return div
  }
})()

// 根据单一职责原则，可以写个通用的
var getSignle = (function (fn) {
  var result

  return function () {
    return fn || (result = fn.apply(this, arguments))
  }
})()
```


<a name="civXP"></a>
## 策略模式
策略模式的定义是：定义一系列的算法，把他们一个个封装起来，并且是他们可以相互替换。<br />
<br />一个基于策略模式的程序至少由两个部分组成。第一个部分是一组策略类，策略类封装了具体的算法，并负责具体的计算过程。第二个部分是环境类 Context，Context 接受客户的请求，随后把请求委托给某一个策略类。<br />例子：缓动动画 Tween、表单验证P86。<br />计算年终奖的代码：
```javascript
var strategies = {
  'S': function (salary) {
    return salary * 4
  },
  'A': function (salary) {
    return salary * 3
  },
  'B': function (salary) {
    return salary * 2
  },
}

// Context
var calculateBonus = function (level, salary) {
  return strategies[level](salary)
}

console.log(calculateBonus('S', 40000));
console.log(calculateBonus('B', 30000));
```
<a name="SzybI"></a>
### 策略模式的优缺点

- 策略模式利用组合、委托和多态等技术和思想，可以有效地避免 if 语句。
- 将算法封装在独立的 strategy 中，易于切换、理解和扩展。
- 策略模式中的算法容易复用。
- 利用组合和委托的来让Context 拥有执行算法的能力，这也是继承的一种替代方案。


<br />当 Context 将请求委托给策略对象的时候，这些策略对象会根据不同的请求返回不同的结果，这便表现出对象的多态性。<br />
<br />在函数作为一等对象的 JS 中，策略类往往会被函数所代替，所以容易认不出一个策略模式的实现。<br />

<a name="QnJBB"></a>
## 代理模式
代理模式是为一个对象提供一个代用品或占位符，以便控制对他的访问。<br />

<a name="HXZP7"></a>
### 保护代理和虚拟代理
_保护代理_用于控制不同权限的对象对目标对象的访问。_虚拟代理_把一些开销很大的对象，延迟到真正需要它的时候去创建。<br />
<br />因为单一职责原则，通常额外的功能放到代理对象去。之后，当不需要这个额外的功能的时候，可以更加快速的替换。<br />
<br />例子：

- 虚拟代理合并 HTTP 请求：将用户触发的请求进行延时。
- 惰性加载中的应用：编写一个功能库的时候，在用户真正需要之前，用简单的代码先接管并缓存记录下来，当实际用到时候，再加载真正的函数，并遍历缓存队列，执行他们。
- 缓存代理可以为一些开销大的运算结果提供暂时的存储，如果传参跟之前的一致，可以直接返回结果。

其它代理模式：

- 防火墙代理
- 远程代理：为一个对象再不同的地址空间提供局部代表。
- 保护代理
- 智能引用代理：取代了简单的指针，他在访问对象时执行一些附加操作，比如计算一个对象被引用的次数。
- 写时复制代理：通常用于复制一个庞大对象的情况。写时复制代理延迟了复制的过程，当对象真正修改的时候，才对他进行复制操作。DLL是其典型运用场景。



<a name="JD80G"></a>
## 迭代器模式
迭代器模式是指提供一种方法顺序访问一个聚合对象的各个元素，而又不需要暴露该对象的内部表示。<br />
<br />迭代器可以分为内部迭代器和外部迭代器。

- 内部迭代器在迭代函数内部将接手真个迭代过程，外部只需要一次调用。
- 外部迭代器必须显示地请求迭代下一个元素。

例子：迭代得到有用的对象。<br />
<br />绝大数语言都内置了迭代器模式。<br />

<a name="krfpe"></a>
## 发布-订阅模式
发布-订阅模式又叫观察者模式，它定义对象的一种一对多的以来关系，当一个对象发生改变时，所有依赖它的对象都将得到通知。在 JavaScript 中，我们一般用事件来模拟发布-订阅模式。<br />
<br />在 DOM 节点上绑定事件函数，就是发布-订阅模式。<br />
<br />发布-订阅模式的通用实现。P114.
```javascript
var event = {
  clientList: {},
  listen(key, fn) {
    //...
  },
  trigger() {
  	//...
  },
  remove(key, fn) {
  	//...
  },
}
```


<a name="RqHI8"></a>
### 必须先订阅再发布吗
实际上是有类似需求的，我们可以在发布的时候，如果没有订阅着来订阅这个时间，我们暂时把发布事件的动作包裹在一个函数里，这些包装函数将存入堆栈中，等到有对象来订阅事件的时候，再去遍历堆栈并执行，也就是重新发布里面这些事件。就像 QQ 的未读消息指挥重新阅读一次，这样的操作我们只能进行一次。<br />
<br />在 JavaScript 中，我们用注册回调函数的形式来代替传统的发布-订阅模式。<br />发布-订阅模式的优点：一为时间上的结构，二位对象之间的解耦。它应用广泛，既可以用在异步编程中，也可以帮助我们完成更松耦合的代码编写。<br />发布订阅模式的缺点：

- 创建订阅者本身需要一定的事件和内存，订阅后即使这条消息到最后也没发生，它依然始终在内存中。
- 过度使用的话，会导致对象和对象之间的必要联系被深埋在背后，会导致程序难以跟踪维护和理解。
<a name="WkEUg"></a>
## 命令模式
命令模式是最简单和最优雅的模式之一，命令模式中的命令（command）指的是一个执行某些特定事情的指令。<br />

<a name="gLkUS"></a>
### 撤销和重放
命令模式的作用除了封装运算快，而且可以很方便地给命令对象添加撤销（多步）操作和重做。<br />我们可以将所有执行过的命令都储存在一个历史列表中，然后倒叙循环来一次执行这些命令的 undo 操作。<br />在 canvas 倒序执行命令并不能达到想要的结果，我们还可以选择一开始将所有的命令存在历史堆栈中，然后重复执行他们。借助这个还可以实现想“回放”的功能。<br />

<a name="YMMOf"></a>
### 命令队列
我们还可以生成一个命令队列，当前的 command 对象职责完成了，才去通知队列，然后取出正在队列中等待的第一个命令对象，并且执行它。通知的方式可以选择 回调函数 或者 发布-订阅模式。<br />

<a name="iVwpg"></a>
### 宏命令
宏命令是一组命令的集合，通过执行宏命令的方式，可以一次执行一批命令。<br />

<a name="sfD1J"></a>
### 智能命令和傻瓜命令
一般来说，命令模式都会在 command 对象中保存一个接收者来负责真正执行客户的请求，这种情况下命令对象是“傻瓜式”的。<br />
<br />智能命令本身就包揽了执行请求的行为，和策略模式很像，代码结构上已经无法分辨他们，能分辨的只有他们意图的不同。策略模式指向问题的问题域更小，所有策略对象的目标是一致的。智能命令模式指向的问题域更广， command 对象解决的目标更具发散性。命令模式还可以完成撤销、排队等功能。<br />

```javascript
// 傻瓜命令
var setCommand = function (button, func) {
  button.onclick = function () {
    func()
  }
}

var MenuBar = {
  refresh: function () {
    console.log('refresh menubar');
  }
}

var RefreshMenuBarCommand = function (reciver) {
  return {
    execute: function () {
      receiver.refresh()
    }
  }
}

var refreshMenubarCommand = RefreshMenuBarCommand(MenuBar)

setCommand(button, refreshMenubarCommand)

// 智能命令
var closeDoorCommand = {
  execute: function () {
    console.log('close door');
  }
}
```
<a name="VkYzO"></a>
### 组合模式
组合模式就是用小的对象来构建更大的对象，而这些小的子对象本身也许是由更小的“孙对象”构成的。<br />
<br />命令模式中的宏命令就是一种组合模式，包含了组合对象，和叶对象，组合对象有 add 方法，而叶对象没有。<br />
<br />组合模式将对象组合成一种树结构，以表示“部分-整体”的层次结构。<br />
<br />组合模式的另一个好处是，通过对象的多态性，使得用户对单个对象和多个对象的使用具有一致性。<br />
<br />例子：扫描文件夹（File 和 Folder）<br />

<a name="C4Aup"></a>
### 一些值得注意的地方

1. 组合模式布什父子关系：组合模式是一种 HSA-A （聚合）的关系，而不是 IS-A。
1. 对叶对象操作的一致性。
1. 双向映射关系。
1. 用职责链模式提高组合模式性能。
<a name="K4uAD"></a>
### 引用父节点
一般情况下，组合对象保存了它下面的字节点的引用，但我们可以让子节点保存对父对象的引用。<br />

<a name="m1O31"></a>
### 何时使用组合模式

- 表示对象的部分-整体结构。
- 客户希望统一对待树中所有的对象。省了写一堆 `if else` 。
<a name="YMch8"></a>
## 模板方法（Template Method）模式
模板方法是一种基于继承的设计模式。<br />
<br />模板方法有两部分结构组成，第一部分是抽象类，第二部分是具体的实现子类。前者比如饮料，后者比如茶、咖啡。<br />
<br />在抽象父类通常封装了子类的基本的算法框架，包括一些公共方法以及封装子类中所有方法的执行顺序。子类通过继承这个抽象类，也继承了整个算法结构，并且可以选择重写父类的方法。
<a name="E00hu"></a>
### 抽象方法和具体方法
抽象方法被声明在抽象类中，抽象类没有具体的实现过程。当子类继承这个抽象类的时候，必须要重写这些抽象方法。<br />当每个子类有一些同样的具体实现方法，那这些方法可以选择放在抽象类中，这些方法叫具体方法。<br />
<br /> 可以使用钩子方法（hook）来隔离变化，具体看下面代码。<br />
<br />JavaScript 并没有从语法层面提供对抽象类的支持，当我们使用原型继承来模拟传统的类式继承的时候，并没有编译器帮助我们进行检查，我们也没有办法保证子类会重写父类中的“抽象方法”。<br />
<br />比起通过原型去模拟传统的类式继承，高阶函数是更好的选择。 <br />
<br />模板方法模式是一种典型的通过封装变化提高系统扩展性的设计模式。
```javascript
var Berverage = function () {}

Berverage.prototype.boilWater = function () {}
Berverage.prototype.brew = function () {}
Berverage.prototype.pourInCup = function () {}
Berverage.prototype.addCondiments = function () {}
// 钩子方法
Berverage.prototype.customWantsCondiments = function () {}
Berverage.prototype.init = function () {
  this.boilWater()
  this.brew()
  this.pourInCup()
  if (this.customWantsCondiments()) {
    this.addCondiments()
  }
}

var Coffee = function () {}
Coffee.prototype = new Berverage()
Coffee.prototype.brew = function () {}
Coffee.prototype.pourInCup = function () {}
Coffee.prototype.addCondiments = function () {}
Coffee.prototype.customWantsCondiments = function () {
  return window.confirm('want condiments?')
}

// 闭包实现
var Berverage = function (params) {
  var boilWater = function () {}
  var brew = params.brew || function () {}
  var pourInCup = params.pourInCup || function () {}
  var addCondiments = params.addCondiments || function () {}

  var F = function () {}
  F.prototype.init = function () {
    boilWater()
    brew()
    pourInCup()
    addCondiments()
  }

  return F
}

var Coffee = Berverage({
  brew: function () {},
  pourInCup: function () {},
  addCondiments: function () {},
})
```
<a name="Tkwi3"></a>
## 享元模式
享元模式是一种用于性能优化的模式。享元模式的核心是运用共享技术来有效支持大量细粒度的对象。<br />尝试理解：现在有50件男装，50件女装需要为它们找模特穿上并拍照，不适用享元模式，我们为每件衣服拍照都要 `new`  一个对象，然而我们真正需要的模特其实就两个，一个男模特，一个女模特。
<a name="YElfK"></a>
### 内部状态
享元模式要求将对象划分为内部状态和外部状态。内部状态储存于对象内部。剥离了外部状态的的对象成为共享对象，外部状态在必要的时候传入共享对象来组成一个完整的对象。这个过程需要花费一点微不足道的时间，所以享元模式是一种用时间换空间的优化模式。<br />
<br />文件上传的例子：P172<br />

<a name="hl9Sa"></a>
### 对象池
对象池维持一个装载空闲对象的池子，如果需要对象，不是直接 new，而是转从对象池里获取。
<a name="jtpFE"></a>
## 职责链模式
职责链模式的定义是：使多个对象都有机会处理请求，从而别面请求的发送者和接收者之间的耦合关系，将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。<br />

<a name="fGI2o"></a>
## 中介者模式
中介者模式的作用是解除对象之间的紧耦合关系。增加一个中介对象后，所有的相关对象都通过中介者对象来通信，而不是互相引用，所以当一个对象改变时，只需要通知中介者对象即可。<br />
<br />中介者模式使网状的多对多关系变成了相对简单的的一对多关系。<br />

<a name="fYamy"></a>
### 现实中的中介者

1. 机场指挥塔
1. 博彩公司


<br />例子：泡泡堂、购买商品 <br />
<br />中介者模式时迎合迪米特法则的一种实现。迪米特法则也叫最少知识原则，是指一个对象尽可能少了解另外的对象。如果一个对象之间的耦合性太高，一个对象发生改变之后，难免会影响其它的对象。而在中介者模式里，对象之间不知道彼此的存在，只能通过中介者来互相影响对方。<br />中介者模式最大的缺点是，系统中新增了一个中介者对象，因为对象之间交互的复杂性，转移成了中介者对象的复杂性，是的中介者对象经常是巨大的。其本身往往就是一个难以维护的对象。
<a name="FC5B4"></a>
## 装饰者模式
装饰者（decorator）模式可以给类动态地增加一些额外地职责，而不会影响到这个类中派生地其它对象。<br />
<br />跟继承相比，装饰者是一种更轻便灵活地做法，这是一种“即用即付”的方式。<br />
<br />例子：数据上报、统计函数的执行时间、动态改变函数以及插件式的表单验证。<br />

<a name="TsiUM"></a>
### 装饰者模式和代理模式
两者很像，最重要的区别在于他们的意图和设计目的。<br />代理模式强调一种关系（Proxy与它的实体之间的关系），这种关系可以静态表达，也就是说，这种关系在一开始就可以确定。<br />而装饰者模式用于一开始不能确定对象的全部功能时。<br />代理模式通常只有一层代理-本体的引用，而装饰者模式经常会形成一条长长的装饰链。
<a name="1ukbP"></a>
## 状态模式
状态模式的关键是区分事物内部的状态，事物内部的状态往往会带来事物的行为改变。<br />

<a name="4TYAW"></a>
### 电灯程序
```javascript
// 不 使用状态模式
var Light = function () {
  this.state = 'off'
}
Light.prototype.init = function () {
  button.onclick = () => {
    this.buttonWasPressed()
  }
}
Light.prototype.buttonWasPressed = function () {
  if (this.state === 'off') {
    this.state = 'on'
  } else if (this.state === 'on') [
    this.state = 'off'
  ]
}

const light = new Light()
light.init()

// 使用状态模式
var Light = function () {
  this.onlightState = new OnlightState(this)
  this.offLightState = new OffLightState(this)
}
Light.prototype.setState = function (state) {
  this.currState = state
}
Light.prototype.init = function (state) {
  this.currState = this.offLightState
  button.onclick = () => {
    this.currState.buttonWasPressed()
  }
}

var OnlightState = function (light) {
  this.light = light
}
OnlightState.prototype.buttonWasPressed = function () {
  this.light.setState(this.light.offLightState)
}

var OffLightState = function (light) {
  this.light = light
}
OffLightState.prototype.buttonWasPressed = function () {
  this.light.setState(this.light.onlightState)
}
```


<a name="h7f6y"></a>
## 适配器模式
适配器模式的作用是解决两个软件实体间的接口不兼容的问题。使用适配器模式之后，原本由于接口不兼容的而不能工作的两个软件实体就可以一起工作。<br />适配器的别名是包装器（wrapper）。
```javascript
var googleMap = {
	show: () => {}
}

var baiduMap = {
	display: () => {}
}

var baiduMapAdapter = {
  show: () => {
    return baiduMap.display()
  }
}
  
```
