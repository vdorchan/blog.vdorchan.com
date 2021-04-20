---
title: 让人脑壳疼的继承与原型链
date: 2019-03-27 10:43:12
tags: 
  - JavaScript
  - 原型链
  - 继承
---
JavaScript 的继承是基于*原型链*实现的。虽然在 ES2015/ES6 中引入了`class`关键字，但那仅仅是语法糖。

*原型链*是一种机制，指的是 JavaScript 每个对象都有一个内置的 `__proto__` 属性指向创建它的构造函数的 prototype （原型）属性。

比如

```javascript
function Person() {
}
var person = new Person()

console.log(person.__proto__ === Person.prototype) // true
```

## 函数也是对象

普通对象是这样子的：

```javascript
var o1 = {}
var o2 = new object()
```

凡是像下面代码使用 `function` 关键字或 `Fucntion` 构造函数创建的对象都是函数对象。只有函数对象才拥有 `prototype` （原型）对象。

```javascript
function f1 () {}
var f2 = function (){}
var f3 = new Function('str', 'console.log(str)')
```

## 构造函数和 prototype

`ECMAScript` 中提供了构造函数来创建新对象。构造函数本身就是一个函数，它和普通函数没有任何的区别。

前面示例代码中的 `Person` 就是一个构造函数，首字母大写并不是它被称为构造函数的原因，这是管理，但不是必须的。

而是因为**函数被 `new` 关键字调用时就是构造函数。**

那么当 `Person` 构造函数被 `new` 关键字调用的时候都发生了什么呢？

new 关键字的内部实现机制：

* 创建一个新对象；
* 将构造函数的作用域赋值给新对象（因此 this 就指向了这个新对象）；
* 执行构造函数中的代码；
* 返回新对象

### 显式原型（explicit prototype property） prototype

> 每一个函数在创建之后都会拥有一个名为prototype的属性，这个属性指向函数的原型对象

Note：通过Function.prototype.bind方法构造出来的函数是个例外，它没有prototype属性。

在默认情况下，所有原型对象都会自动获得一个 constructor（构造函数）属性，这个属性包含一个指向 prototype 属性所在函数的指针。

通过这个属性可以判断实例是由哪个构造函数创建的

```javascript
Person.prototype.constructor === Person // true

person.constructor === Person // true
```

一般情况下，你可以这么添加一个原型方法

```javascript
Person.prototype.say = function () {}
```

你也可以重写整个原型对象，像下面这样

```javascript
Person.prototype = {
  say: function () {}
}
```

这样子重写整个原型对象的缺点是他失去了原本的 `constructor` 属性，不再指向 `Person` 构造函数了。你也就无法再通过这个属性去确定对象的类型了。

```javascript
Person.prototype = {
  say: function () {}
}

var person = new Person()

console.log(person instanceof Person) // true
console.log(person instanceof Object) // true
console.log(person.constructor === Person) // false
console.log(person.constructor === object) // true
```

从上面代码可知，虽然通过 `instance` 操作符仍可以判断到对象的类型。但 `person` 的 `constructor` 属性则不再指向 `Person` 了。而是指向了 `Object`。

当然你可以像这样子把它带回来

```javascript
Person.prototype = {
  constructor: Person,
  say: function () {}
}
```

因为原生的 `constructor` 属性是不可枚举，因此使用下面代码显得更严谨些：

```javascript
Person.prototype = {
  say: function () {}
}

Object.defineProperty(Person.prototype, 'constructor', {
  value: Person,
  emuerable: false
})
```

 ```javascript
function Person() {
}
var person = new Person()

console.log(person.__proto__ === Person.prototype) // true
```

由于在原型中查找值的过程是一次搜索，因此我们对原型对象所做的任何修改都能够立即从实例上反映出来，即使是在创建实例之后，再去修改原型对象内的属性，实例也能立即反映出来。但假如在实例创建之后，重写整个原型对象，将会导致实例失去与最初原型之间的联系。

**实例中的指针仅指向原型，而不指向构造函数。**

```javascript
function Person() {
}
var person = new Person()
Person.prototype.say = function () {
  console.log('Hello')
}

person.say() // 'Hello'

Person.prototype = {
  constructor: Person,
  say: function () {
    console.log('Bye')
  },
  run: function () {
    console.log('run');
  }
}

person.say() // 'Hello'
person.run() // Uncaught TypeError: person.run is not a function
```

## 隐式原型（implict prototype link）__proto__

### 继承属性

JavaScript中任意对象都有一个内置属性`[[prototype]]`，在ES5之前没有标准的方法访问这个内置属性，但是大多数浏览器都支持通过`__proto__`来访问。ES5中有了对于这个内置属性标准的Get方法`Object.getPrototypeOf()`。

当调用构造函数创建一个新的实例之后，该实例的内部将包含一个指针 `__proto__`，并指向构造函数的显示原型（prototype），因此也得以继承了构造函数原型上所有的属性和方法。

接下来使用 new 操作符创建对象的过程可以利用 `__proto__` 来完成

```javascript
function Person () {}

// var person = new Person()
// 上一行代码等同于以下过程 ==>
var person = {}
person.__proto__ = Person.prototype
Person.call(person)
```

### 创建对象和生成原型链的不同方法

1、使用字面量创建对象

> ```javascript
> var o = { a: 1 }
>
> console.log(o.__proto__ === Object.prototype) // true
> // o 这个对象继承了Object.prototype上面的所有属性
> // 原型链如下：
> // o ---> Object.prototype ---> null
> ```

2、使用构造函数创建对象

```javascript
function Person (name) {
  this.name = 'curry'
}
Person.protype.say = function () {
  console.log('I am handsome')
}
var person = new Person()

console.log(person.__proto__ === Person.prototype) // true
console.log(Person.prototype === Object.prototype)

// person 是生成的新对象，'name' 是它的自身属性
// 这个对象在实例化的时候，person.__proto__ 指向了Person.prototype，并继承了它的所有属性和方法（say方法）
// 然后 Person.prototype 又指向了 Object.prototype
// 所以，原型链为：
// person ---> Person.prototype ---> Object.prototype ---> null
```

3、Object.create 创建的对象

`Object.create()` 是 ECMAScript 5 引入的一个新方法。可以调用这个方法来创建一个新对象。传入的第一个参数为新对象的原型。

```javascript
var a = { a: 1 }
// a ---> Object.prototype ---> null

var b = Object.create(a)
// b ---> a ---> Object.prototype ---> null
console.log(b.a) // 1 继承而来

var d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); // undefined, 因为d没有继承Object.prototype
```

## Object.create

`Object.create()` 创建一个新对象，使用现有的对象来提供新创建的对象的`__proto__`。

```javascript
const person = {
  job: '',
  say: function () {
    console.log(`My name is ${this.name},my job is ${this.job}`)
  }
}

const curry = Object.create(person)

curry.name = 'curry'
curry.job = 'basketball player'

curry.say()
```

可以使用 `Object.create` 实现类式继承

```javascript
// Person - 父类(superclass)
function Person () {
  this.msg = 'hello'
}

// 父类的方法
Person.prototype.say = function () {
  console.log(this.msg)
}

// Student - 子类(subclass)
function Student() {
  Person.call(this) // 调用父类的构造器
}

// 子类继承父类
Student.prototype = Object.create(Person.prototype)
Student.prototype.constructor = Student

var curry = new Student()

console.log(curry instanceof Student)
console.log(curry instanceof Person)
curry.say()

```

ES5 之前如何实现 Object.create 呢，这里不考虑第二个参数，只包括核心代码

```javascript
if (typeof Object.create !== 'function') {
  // ... 
  Object.create = function (proto) {
    function F () {}
    F.prototype = proto
    return new F()
  }
}
```

## instanceof

判断一个变量的类型尝尝会用 typeof 运算符，在使用 typeof 运算符时采用引用类型存储值会出现一个问题，无论引用的是什么类型的对象，它都返回 "object"。ECMAScript 引入了另一个 Java 运算符 instanceof 来解决这个问题。instanceof 运算符与 typeof 运算符相似，用于识别正在处理的对象的类型。与 typeof 方法不同的是，instanceof 方法要求开发者明确地确认对象为某特定类型。

instanceof 表示的是一种继承关系，它通过沿着原型链不断想上寻找原型来确定继承关系。

用代码表示大致如下

```javascript
function instance_of (L, R) { //L 表示左表达式，R 表示右表达式
 var O = R.prototype // 取 R 的显示原型
 L = L.__proto__ // 取 L 的隐式原型
 while (true) {
   if (L === null)
     return false
   if (O === L) // 这里重点：当 O 严格等于 L 时，返回 true
     return true
   L = L.__proto__
 }
}
```

## 类

ECMAScript 2015 中引入的 JavaScript 类实质上是 JavaScript 现有的基于原型的继承的语法糖。类语法不会为JavaScript引入新的面向对象的继承模型。

类实际上是个“特殊的函数”，就像你能够定义的函数表达式和函数声明一样，类语法有两个组成部分：类表达式和类声明。

1、类声明

使用 calss 关键字来声明，虽然**函数声明**会提升，但**类声明**并不会。

```javascript
class Person {
  constructor (name) {
    this.name = name
  }
}
```

2、类表达式

```javascript
/* 匿名类 */
let Person = class Person {
  constructor (name) {
    this.name = name
  }
}

/* 声明的类 */
let Person = class Person {
  constructor (name) {
    this.name = name
  }
}
```

> 一个类的类体是一对花括号/大括号 {} 中的部分。这是你定义类成员的位置，如方法或构造函数。
> 类声明和类表达式的主体都执行在严格模式下。比如，构造函数，静态方法，原型方法，`getter`和`setter`都在严格模式下执行。
> `constructor` 方法是一个特殊的方法，这种方法用于创建和初始化一个由 `class `创建的对象。一个类只能拥有一个名为 “constructor”的特殊方法。如果类包含多个`constructor`的方法，则将抛出 一个`SyntaxError` 。
> 一个构造函数可以使用 super 关键字来调用一个父类的构造函数
> static 关键字用来定义一个类的一个静态方法。
> 使用 extends 创建子类

下面是使用 `class` 的例子

```javascript
class Person {
  constructor (name) {
    this.name = name
  }

  say () {
    console.log(`I am ${this.name}`)
  }
}

class Student extends Person {
  /* 如果没有显式指定构造方法，则会添加默认的 constructor 方法。 */
  constructor (name) {
    super(name)
  }

  study () {
    console.log('I am studing')
  }
}

const curry = new Student('curry')
curry.say() // I am curry
curry.study() // I am studing
```

## 性能

在原型链上查找属性比较耗时，对性能有副作用，这在性能要求苛刻的情况下很重要。另外，试图访问不存在的属性时会遍历整个原型链。

遍历对象的属性时，原型链上的每个可枚举属性都会被枚举出来。要检查对象是否具有**自己定义的属性**，而不是其原型链上的某个属性，则必须使用所有对象从 `Object.prototype` 继承的 `hasOwnProperty` 方法。

`hasOwnProperty` 是 JavaScript 中处理属性并且不会遍历原型链的方法之一。(另一种方法: `Object.keys()`)

注意：检查属性是否undefined还不够。该属性可能存在，但其值恰好设置为 `undefined`。

当谈到继承时，JavaScript 只有一种结构：对象。每个实例对象（object ）都有一个私有属性（称之为__proto__）指向它的原型对象（prototype）。该原型对象也有一个自己的原型对象(__proto__) ，层层向上直到一个对象的原型对象为 null。根据定义，null 没有原型，并作为这个原型链中的最后一个环节。

参考链接：
https://github.com/stone0090/javascript-lessons/tree/master/2.5-Prototype
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain
https://www.zhihu.com/question/34183746
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create
https://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/