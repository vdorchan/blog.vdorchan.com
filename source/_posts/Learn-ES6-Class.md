---
title: Learn-ES6-Class
date: 2018-03-08 16:49:47
tags: 
  - ES6
  - 学习
  - class
  - ES6学习笔记
---
## 1.简介

JavaScript 语言中，传统的生成实例对象的方法是通过构造函数 。

```javascript
function Person(name, age) {
  this.name = name
  this.age = age
}

Person.prototype.say = function () {
  console.log('my name is ' + this.name + ', i am ' + this.age + ' years old')
}

var person = new Person('kobe', 30)

person.say() // my name is kobe, i am 30 years old
```

ES6 引入了 Class （类）这个概念，通过 class 关键字可以定义类，写法和其它传统语言类似，可以看作是一个语法糖，新的 class 写法只是让对象原型的写法更加清晰、更像面向对象编程的语法。

```javascript
class Person {
  contructor(name, age) {
    this.name = name
    this.age = age
  }

  say() {
    console.log('my name is ' + this.name + ', i am ' + this.age + ' years old')
  }
}

// 对类的使用和 ES5 中使用构造函数的方法一样，直接对类使用 new 命令即可。
var person = new Person('kobe', 30)
```

Person 类中的 constructor 方法其实就相当于 Person 构造函数

```javascript
class Person {
  // ...
}

Person = Person.prototype.constructor // true
```

构造函数的 prototype 属性在“类”中依然存在，事实上，类的所有方法都定义在类的prototype属性上面。

```javascript
class Person1 {
  contructor(name, age) {
    // ...
  }

  say() {
    // ,,,
  }

  jump() {
    // ...
  }
}

// 等同于
Person.prototype = {
  contructor(name, age) {
    // ...
  },
  say() {
    // ,,,
  },
  jump() {
    // ...
  }
}

// 在类的实例上面调用方法，其实就是调用原型上的方法。
class B {}
let b = new B();

b.constructor === B.prototype.constructor // true

```

和 ES5 不同的是，类内部所定义的方法是不可枚举的。

```javascript
class Person {
  contructor(name, age) {
    // ...
  }

  say() {
    // ,,,
  }
}

Object.keys(Person.prototype) // []
Object.getOwnPropertyNames(Person.prototype) // ['constructor', 'say']


var Person = function () {}
Person.prototype.say() {}

Object.keys(Person.prototype) // ['say']
Object.getOwnPropertyNames(Person.prototype) // ['constructor', 'say']
```

类的属性名，可以采用表达式
```javascript
let methodName = 'getArea';

class Square {
  constructor(length) {
    // ...
  }

  [methodName]() {
    // ...
  }
}
```

## 2. 严格模式

类和模块的内部，默认就是严格模式。ES6 实际上把整个语言升级到了严格模式。

## 3. constructor

constructor方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法。一个类必须有constructor方法，如果没有显式定义，一个空的constructor方法会被默认添加.

```javascript
class Point {
}

// 等同于
class Point {
  constructor() {}
}
```

constructor方法默认返回实例对象（即this），完全可以指定返回另外一个对象。

```javascript
class Foo {
  constructor() {
    return Object.create(null)
  }
}

new Foo() instanceOf Foo // false
```

类必须使用new调用，否则会报错。

## 4.类的实例对象

与 ES5 一样，所以实例对象共享一个原型对象。

```javascript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__
```

这也意味着，可以通过实例的 __proto__ 属性为“类”添加方法

__proto__ 并不是语言本身的特性，这是各大厂商具体实现时添加的私有属性，虽然目前很多现代浏览器的 JS 引擎中都提供了这个私有属性，但依旧不建议在生产中使用该属性，避免对环境产生依赖。

可以使用 Object.getPrototypeOf 方法代替

另外改写原型，必须相当谨慎，不推荐使用，因为这会改变“类”的原始定义，影响到所有实例。

```javascript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__.printName = function () { return 'Oops' };

p1.printName() // "Oops"
p2.printName() // "Oops"

var p3 = new Point(4,2);
p3.printName() // "Oops"
```

## 5. Class 表达式

与函数一样，类也可以使用表达式的形式定义。

```javascript
const MyClass = class Me{
  getClassName() {
    return Me.name
  }
}
```

上面代码使用表达式定义了一个类。需要注意的是，这个类的名字是MyClass而不是Me，Me只在 Class 的内部代码可用，指代当前类, Me 也可以省略

```javascript
let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined
```

利用 Class 表达式，可以写出立即执行的 Class

```javascript
let person = new class {
  constructor(name) {
    this.name = name
  }

  sayName() {
    console.log(this.name)
  }
}('张三')

person.sayName()
```

## 6.不存在变量提升

类不存在变量提升（hoist），这一点与 ES5 完全不同。

```javascript
new Foo(); // ReferenceError
class Foo {}
```

假如存在变量提升的话，在继承类的时候就会出现问题。

```javascript
let Foo = class {}
class Bar extends Foo {}
```

上面的代码，因为 let 命令是没有变量提升的，加入 class 提升到前面的话，这个时候 Foo 类还没有定义，就会出错了。

## 7.私有方法和私有属性

ES6 不提供私有方法，只能通过变通方法模拟实现。

一种做法是在命名上加以区别。

只是这种做法也仅仅就是命名上的区别，在类的外部，仍然可以访问到私有方法。

```javascript
class Widget {
  // 公有方法
  foo(baz) {
    this._bar(baz)
  }

  // 私有方法
  _bar(baz) {
    return this.snaf = baz
  }
}
```

另一种方法就是索性将私有方法移出模块，因为模块内部的所有方法都是对外可见的。

```javascript
class Widget {
  foo(baz) {
    bar.call(this, baz)
  }

}

function bar(baz) {
  return this.snaf = baz
}
```

还有一种方法是利用Symbol值的唯一性，将私有方法的名字命名为一个Symbol值。 因为都是Symbol值，导致第三方无法获取到它们，因此达到了私有方法和私有属性的效果。

```javascript
const bar = Symbol('bar')
const snaf = Symbol('snaf')

class Widget {
  // 公有方法
  foo(baz) {
    this.[bar](baz)
  }

  // 私有方法
  [bar](baz) {
    return this.[snaf] = baz
  }
}
```

ES6 同样也没有私有属性，目前，有一个提案，是在属性名前加 # 号表示。当然同样的也可以应用在方法上。

## 8. this 的指向


## 9. name 属性

继承了 ES5的很多特性， 包括 name 属性

```javascript
class Point {}
Point.name // "Point"
```

## 10.Class 的取值函数（getter）和存值函数（setter）

与 ES5 一样，在“类”的内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

```javascript
class MyClass {
  constructor() {
    // ...
  }

  get Prop() {
    return 'getter'
  }

  set Prop(value) {
    console.log('setter: '+value)
  }
}

let inst = new MyClass();

inst.prop = 123;
// setter: 123

inst.prop
// 'getter'
```

和 ES5 一致，存值函数和取值函数是设置在属性的 Descriptor 对象上的。

```javascript
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, "html"
);

"get" in descriptor  // true
"set" in descriptor  // true
```

## 11.class 的 Generator 方法

如果某个方法之前加上星号，就表示该方法是一个 Generator 函数

```javascript
class Foo {
  constructor (...args) {
    this.args = args
  }

  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg
    }
  }
}

for (let f of new Foo('Hello', 'World')) {
  console.log(f);
}

// 'Hello'
// 'World'
```

## 12.Class 的静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

```javascript
class Foo {
  static classMethod () {
    return 'hello'
  }
}

Foo.classMethod()

var foo = new Foo() // 'hello'

foo.classMethod() // TypeError: foo.classMethod is not a function
```

注意，如果静态方法包含this关键字，这个this指的是类，而不是实例

```javascript
class Foo {
  static bar () {
    return this.baz()
  }

  static baz () {
    return 'hello'
  }

  baz () {
    return 'world'
  }
}

Foo.bar() // 'hello'
```

父类的静态方法，可以被子类继承。

```javascript
class Foo {
  static bar () {
    return 'hello'
  }
}

class Bar extends Foo {
}

Bar.bar() // 'hello'
```

静态方法也可以从 super 对象上调用的

```javascript
class Foo {
  static bar () {
    return 'hello'
  }
}

class Bar extends Foo {
  static bar () {
    return super.bar() + ',too'
  }
}

Bar.bar()
```

## 13.Class 的静态属性和实例属性

静态属性指的是 Class 上的属性，即 Class.propName，而不是定义在实例对象（this）上的属性。

```javascript
class MyClass {

}

MyClass.prop = 'hello'
MyClass.prop // 'hello'
```

上面的写法是 ES6 中唯一的写法，因为 ES6 规定 Class 内部只有静态方法，没有静态属性。

目前有一个静态属性的提案，对实例属性和静态属性都规定了新的写法。

* （1）类的实例属性

类的实例属性可以用等式，写入类的定义之中。

```javascript
class MyClass {
  myProp = 42

  constructor() {
    console.log(this.myProp); // 42
  }
}
```

以前，我们定义实例属性，只能写在类的constructor方法里面,有了新的写法以后，可以不在constructor方法里面定义。

```javascript
class ReactCounter extends React.Component{
  constructor(props) {
    super(props)
    this.state = {
      count: 0
    }
  }
}

// 等同于

class ReactCounter extends React.Component{
  state = {
    count: 0 
  }
}

// 对于在 constructor 里面已经定义的实例属性，新写法允许直接列出。

class ReactCounter extends React.Component{
  state;
  constructor(props) {
    super(props)
    this.state = {
      count: 0
    }
  }
}

```

* （2）类的静态属性

类的静态属性只要在上面的实例属性写法前面，加上static关键字就可以了。

```javascript
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
```

## 14.new.target 属性

new是从构造函数生成实例对象的命令。ES6 为new命令引入了一个new.target属性，该属性一般用在构造函数之中，返回new命令作用于的那个构造函数。如果构造函数不是通过new命令调用的，new.target会返回undefined，因此这个属性可以用来确定构造函数是怎么调用的。

```javascript
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}

// 另一种写法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}

var person = new Person('张三'); // 正确
```

Class 内部调用new.target，返回当前 Class，同时，子类继承父类时，new.target会返回子类，所以可以写出不能独立使用、必须继承后才能使用的类

```javascript
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error('本类不能实例化')
    }
  }
}

class Rectangle extends Shape {
  constructor (length, width) {
    // ...
  }
}

var x = new Shape();  // 报错
var y = new Rectangle(3, 4);  // 正确
```

上面代码中，Shape类不能被实例化，只能用于继承。

注意，在函数外部，使用new.target会报错。