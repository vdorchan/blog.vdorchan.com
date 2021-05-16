---
title: Learn-ES6-object
date: 2018-01-14 18:13:25
tags: 
  - ES6
  - ES6学习笔记
---
## 1. 属性的简洁表示法

ES6 允许直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁。

```javascript
const age = 18
const person = {age}

console.log(person); // {age: 18}
```

除了属性简写，方法也可以简写。

```javascript
const Person = {
  sayHello() {
    console.log('hello');
  }
}

// 等同于
const Person = {
  sayHello: function () {
    console.log('hello');
  }
}
```

因为简写写法的属性名始终是字符串，所以下面代码里的 class 因为是字符串，所以它不属于关键字，而导致解析错误。

```javascript
const o = {
  class() {}
}

// 等同于
const o = {
  'class': function () { }
}
```

如果某个方法的值是一个 Generator 函数，前面需要加上 * 号。

```javascript
const obj = {
  * m() {
    yield 'hello world'
  }
}
```

## 2. 属性名表达式

在 ES5 中，使用字面量方式表达对象的时候，属性名是不能使用表达式作为属性名的。

```javascript
var obj = {
  foo: true,
  abc: 123
}
```

而在 ES6 中，使用表达式作为属性名的行为将被允许。

```javascript
var obj = {
  foo: true,
  ['a' + 'bc']: 123
}
```

表达式同样可以用来定义方法名

```javascript
var obj = {
  ['a' + 'bc']() {}
}
```

注意，属性名表达式如果是一个对象，默认情况下会自动将对象转为字符串[object Object]

## 3. 方法的 name 属性
