---
title: Learn-ES6-class-extends
date: 2018-03-09 10:21:28
tags: 
  - ES6
  - ES6学习笔记
---

## 1.简介

Class 可以通过extends关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多

```javascript
class Point {

}

class ColorPoint extends Point {

}
```

上面代码，ColorPoint 继承了 Point，因为没有部署代码，所以两个类是完全一样的。

子类必须在 constructor 中调用 super 方法，否则会出错。因为子类没有自己的 this 对象，所以需要 super 方法继承父类的 this 对象。

```javascript
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y)

    this.color = color;
  }
}
```

ES5 的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6 的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。

作为子类的默认 constructor 方法

```javascript
class ColorPoint extends Point {
}
class ColorPoint extends Point {
  constructor(...args) {
    super(...args)
}
```

下面代码中，实例对象cp同时是ColorPoint和Point两个类的实例，这与 ES5 的行为完全一致。

```javascript
let cp = new ColorPoint(25, 8, 'green');

cp instanceof ColorPoint // true
cp instanceof Point // true
```

最后，父类的静态方法，也会被子类继承。

## 2.Object.getPrototypeOf()

```javascript
Object.getPrototypeOf(ColorPoint) === Point // true
```

## 3.super 关键字

super 关键字可以作为函数和对象使用。

作为函数时，代表父类的构造函数，用于在子类的构造函数中创建子类的实例对象 this

super虽然代表了父类A的构造函数，但是返回的是子类B的实例，即super内部的this指的是B，因此super()在这里相当于 A.prototype.constructor.call(this)

并且作为函数时，只能放在子类的构造函数中。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

作为对象时，在普通方法中，指向父类的原型对象，在静态方法中，指向父类。

```javascript
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```

因为这时 super 指向的是父类的原型对象，所以定义在父类实例上的方法和属性是访问不到的。

```javascript
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

根据上面的特性就有下面的例子

```javascript
class A {
  constructor() {
    this.x = 1
  }
}

class B extends A {
  constructor() {
    super()

    this.x = 2
    console.log(super.x) // undefined
    super.x = 3
    console.log(this.x) // 3
    console.log(super.x) // undefined
  }
}

new B()
```

作为对象用在静态方法中，super 将指向父类。

```javascript
class A {
  say() {
    console.log('hello')
  }
  static say() {
    console.log('world')
  }
}

class B extends A {
  say() {
    super.say()
  }
  static say() {
    super.say()
  }
}
```

super 在使用的时候必须显示的指定是作为函数还是作为对象使用的，否则将会报错。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super); // 报错
  }
}
```

## 4.类的 prototype 属性和 __proto__ 属性

大多数浏览器的 ES5 实现之中，每一个对象都有__proto__属性，指向对应的构造函数的prototype属性。Class 作为构造函数的语法糖，同时有prototype属性和__proto__属性，因此同时存在两条继承链。

* extends 的继承目标

* 实例的 __proto__ 属性

## 5.原生构造函数的继承

一个继承 Array 的例子

```javascript
class MyArray extends Array {
  constructor() {
    super(arguments)
  }
}
```

## 6.Mixin 模式的实现
