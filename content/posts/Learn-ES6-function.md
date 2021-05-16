---
title: ES6学习笔记-函数的扩展
date: 2017-10-19 13:57:25
tags: 
  - ES6
  - ES6学习笔记
---
## 1. 函数参数的默认值
在 ES6 之前，我们如果想要为函数参数制定默认的话，我们的做法是
```javascript
function log(x, y) {
  y = y || 'World'
  console.log(x, y)
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello World

// 因为 y 的值为 false 或者 为空的时候，都会被改为默认值，所以更好的做法是
if (typeof y === 'undefined') {
  y = 'World'
}
```
ES6 则允许直接为参数设置默认值，方式是写在参数定义的后面。
```javascript
function log(x, y = 'World') {
  console.log(x, y)
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```
ES6 的这种写法简直是太简洁了，并且，阅读代码的时候，将很容易的意识到，有哪些参数是可以忽略的。

参数变量是默认声明的，所以根据 let 和 const 的特性，是不能用它们再次声明的。
```javascript
function foo(x = 1, y = 2) {
  let x = 3 // error
  const y = 3 // error
}
```
使用函数默认值的时候，函数不能有同名参数。
```javascript
// 不抱错
function foo(x, x, y) {
  // ...
}

// 报错
function foo(x, x, y = 1) {
  // ...
}
```
如果参数默认值是表达式，每次都会重新计算表达式的值。也就是说，参数默认值是惰性求值的。
```javascript
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}

foo() // 100

x = 100;
foo() // 101
```
### 与解构赋值默认值结合使用
下面的代码只使用对象的解构赋值默认值，没有是函数参数的默认值。
```javascript
function foo({x, y = 5}) {
  console.log(x, y);
}

foo({}) // undefined 5
foo({x: 1}) // 1 5
foo({x: 1, y: 2}) // 1 2
foo() // TypeError: Cannot read property 'x' of undefined
```
通过上面的代码可以看出，如果传入的值不是可以结构的对象的话，是会报错的。

如果提供函数参数的默认值，就可以避免这种情况
```javascript
function foo({x, y = 5} = {}) {
  console.log(x, y);
}

foo() // undefined 5 
```
上面代码指定 foo 函数的参数默认值为一个空对象。

下面是另一个解构赋值默认值的例子。
```javascript
function fetch(url, {body = '', method = 'GET', headers = {}}) {
  console.log(method);
}

fetch('http://example.com', {}) // 'GET'

fetch('http://example.com') // 'GET'
```
上面的 fetch 函数的第二个参数如果是个对象，可以为对象的属性分别指定默认值，这种情况下，这个参数是不能省略的。

如果结合参数默认值，就可以省略第二个参数。这时，就出现了双重默认值。
```javascript
function fetch(url, {body = '', method = 'GET', headers = {}} = {}) {
  console.log(method);
}

fetch() // GET
```
上面代码中，函数 fetch 没有第二个参数的时候，参数默认值生效，然后才是解构赋值的默认值生效。

### 参数默认值的位置
通常情况下，设置了默认值的参数，应该是参数的尾参数。如果非尾参数设置了默认值，实际上是无法省略的。
```javascript
function f(x = 1, y) {
  return [x, y];
}
f() // [1, undefined]
f(2) // [2, undefined])
f(, 1) // 报错
f(null, 1) // [null, 1]
f(undefined, 1) // [1, 1]
```
从上面的代码可以看出来，如果是非尾参数设置了默认值，只有给对应的参数传 undefine 才会触发默认值。

### 函数的 length 属性
指定了默认值之后，函数的 length 属性，将返回没有指定默认值的参数个数
```javascript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
```
一会学到的 rest 参数也不会计入 length 属性
```javascript
(function(...args) {}).length // 0
```
如果指定默认值的不是尾参数，那么 length 属性也不会记入该参数后面的参数了。 
```javascript
(function (a, b = 1, c) {}).length // 1
```

### 作用域
一旦设置了默认值，函数进行初始化的时候，参数会形成一个单独的作用域（context），等到初始化结束以后,这个作用域就会消失。
```javascript
var x = 1
function f(x, y = x) {
  console.log(y);
}

f(2) // 2
f() // undefined
```
函数执行的时候，参数的作用域内，x 为2， 参数的默认值等于这个变量。

另一个例子
```javascript
var x = 1
function f(y = x) {
  let x = 3
  console.log(y);
}

f() // 1
```
上面的代码，参数作用域没有 x 变量，所以指向的是外层的变量 x。而函数体内部的局部变量 x 影响不到默认值变量 x。

如果此时外层的变量 x 也不存在，将会报错
```javascript
function f(y = x) {
  let x = 3
  console.log(y);
}

f() // ReferenceError： x is not defined
```
下面这样写也会报错，因为参数作用域内赋值的时候，实际上执行的是使用 let 赋值，let 有暂时性死区，也就是变量在声明之前是不允许被使用的。
```javascript
var x = 1;

function foo(x = x) {
  // ...
}

foo() // ReferenceError: x is not defined
```
如果默认值是一个函数，该函数的作用也遵守这个规则。
```javascript
let foo = 'outer'
function bar(func = () => foo) {
  let foo = 'inner'
  console.log(func());
}

bar() // outer
```
一个更复杂的例子
```javascript
var x = 1
function foo(x, y = function () { x = 2 }) {
  var x = 3
  y()
  console.log(x);
}

foo() // 3
x // 1
```
上面代码中，全局变量 x， 参数作用域内的变量 x， 函数体内部的变量 x，他们都不在同一个作用域。所以并不会互相影响。

参数 y 的默认值是一个匿名函数，该匿名函数内的变量 x，指向的是第一个参数 x， 处于参数作用域。另外，foo 函数的函数体重新声明了变量 x，和 y 函数的函数体内的变量 x 由于不是同一个作用域，因此执行 y 函数后，内部变量 x 和全局变量 x 都没变。

如果将 foo 函数内部的 var x = 3 的 var 去除，函数 foo 的内部 x 就指向第一个参数 x。
```javascript
var x = 1;
function foo(x, y = function() { x = 2; }) {
  x = 3;
  y();
  console.log(x);
}

foo() // 2
x // 1
```

## 2. rest 参数
ES6 引入了 rest 参数（形式为 ...变量名），用来获取函数的多余参数。
```javascript
function add(...values) {
  let sum = 0

  for (var val of values) {
    sum += values
  }  

  return sum
}

add(2, 3, 4) // 9
```
下面是一个 rest 参数代替 arguments 变量的例子。
```javascript
// arguments 变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort()
}

// rest 参数的写法
function sortNumbers(...numbers) {
  return numbers.sort()
}

// 更简洁的 rest 参数的写法
const sortNumbers = (...numbers) => numbers.sort()
```
arguments 变量的写法中，因为 arguments 并不是数组，只是一个类似数组的对象，所以必须使用 Array.prototype.slice.call() 先将其转换为数组。

rest 参数只能是最后一个参数，后面不能再有其它参数，否则会报错。

函数的 length 属性，不包括 rest 参数。

## 3. 严格模式
从 ES5 开始，函数内部可以设定为严格模式。
```javascript
function doSomething(a, b) {
  'use strict'
  // ...
}
```

ES2016 做了一点修改，规定只要函数参数使用了默认值、解构赋值、扩展运算符，那么函数内部就不能显示设定为严格模式。
```javascript
// 报错
function doSomething(a, b = a) {
  'use strict';
  // code
}

// 报错
const doSomething = function ({a, b}) {
  'use strict';
  // code
};

// 报错
const doSomething = (...a) => {
  'use strict';
  // code
};

const obj = {
  // 报错
  doSomething({a, b}) {
    'use strict';
    // code
  }
};
```
这样规定的原因是，函数内部的严格模式，是同时使用于函数体和函数参数的，但是，函数执行的时候，是先执行函数参数的，这样就会出现一个不合理的地方，就是执行完函数参数，执行函数体的时候才能知道是否以严格模式执行，而函数参数却是先于函数体执行的。

两种方法可以规避这种限制

第一种是设定全局性的严格模式
```javascript
'use strict'
function doSomething() {
  // ...
}
```

第二种是把函数包在一个无参数的立即执行函数里面。
```javascript
const doSomething = (function () {
  'use strict'
  return function () {
    // ...
  }
}())
```

## 4. name 属性
函数的 name 属性，返回该函数的函数名。
```javascript
function foo() {}
foo.name // foo
```
这个标准很早之前就被浏览器广泛支持，直到 ES6 才被写入标准。

ES5 的时候如果将一个匿名函数赋值给一个变量，ES5 的 name 属性，会返回空字符串，而 ES6 的 name 属性会返回实际的函数名。
```javascript
var f = function (params) {}

// ES5
f.name // ""

// ES6 
f.name // "f"
```
如果将一个具名函数赋值给变量，那么 ES5 和 ES6 属性都会返回这个具名函数原本的名字。
```javascript
const bar = function foo() {}

// ES5 和 ES6
bar.name // foo
```
Function 构造函数返回的函数实例，name 属性的值为 anonymous。
```javascript
(new Function).name // "anonymous"
```
bind 返回的函数，name 属性会加上 bound 前缀
```javascript
function foo() {}
foo.bind({}).name // "bound foo"

(function () {}).bind({}).name // "bound "
```

## 5. 箭头函数
ES6 允许使用“箭头”（=>）定义函数。
```javascript
var f = v => v

// 等同于
var f = function (v) {
  return v
}
```
如果箭头函数不需要参数或者需要多个参数，就是用一个圆括号代表参数部分。
```javascript
var f = () => 5
// 等同于
var f = function () { return 5 }

var f = (x, y) => x + y
// 等同于
var f = function (x, y) { return x + y}
```
如果箭头函数的代码块部分多于一条语句，那就要使用下面的形式
```javascript
var f = (x, y) => {
  var a = x + y;
  return a
}
```
因为大括号会被解释为代码块，所以才返回对象的时候，需要加上括号。
```javascript
// 报错
let getTempItem = id => { id: id, name: "Temp" };

// 不报错
let getTempItem = id => ({ id: id, name: "Temp" });
```

### 使用注意点
* 1. 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。
* 2. 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。
* 3. 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
* 4. 不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。

第一点提到的，本来 this 对象的指向是可变的，但是在箭头函数中，它是固定的。
```javascript
function foo() {
  setTimeout(function() {
    console.log('id:', this.id);
  }, 100);
}

var id = 10

foo.call({id: 20}) // id:20

```
上面的代码，如果 setTimeout 的参数是一个普通函数，那么函数执行时，函数体内的 this  应该是始终只想 window 的，结果应该为 10。然而， 当 setTimeout 的参数为箭头函数时，输出的结果是 20。那是因为，箭头函数会导致 this 总是指向函数定义生效时所在的对象（本例是 {id: 20} ），所以输出的是 20。

箭头函数可以让 setTimeout 里面的 this，绑定定义时所在的作用域，而不是指向运行是所在的作用域。下面是另一个例子。
```javascript
function Timer() {
  this.s1 = 0
  this.s2 = 0

  // 箭头函数
  setInterval(() => this.s1++, 1000)

  // 普通函数
  setInterval(function () {
    this.s2++
  }, 1000)
}

var timer = new Timer()

setTimeout(() => console.log('s1: ', timer.s1, ' s2: ', timer.s2), 3100)
// s1:  3  s2:  0
```
从上面的代码可以看出，timer.s1 被刷新了 3 次，而 timer.s2 一次都没有刷新到。

箭头函数可以让 this 指向固定化，这种特性很有利于封装函数。下面的例子，DOM 事件的回调函数封装在一个回调函数里。
```javascript
var handler = {
  id: '123456',

  init: function () {
    document.addEventListener('click', event => this.doSomething(event.type), false)
  },

  doSomething: function (type) {
    console.log('Handling ' + type  + ' for ' + this.id);
  }
}
```
上面的代码中，init 方法中使用了箭头函数，导致 this 总是指向对象 handler，否则回调函数运行时，this.doSomething 这一行会报错，因为此时 this 指向 document 对象。

this指向的固定化，并不是因为箭头函数内部有绑定this的机制，实际原因是箭头函数根本没有自己的this，导致内部的this就是外层代码块的this。正是因为它没有this，所以也就不能用作构造函数。

所以，箭头函数转成 ES5 的代码如下。
```javascript
// ES6
function foo() {
  setTimeout(() => {
    console.log('id: ', this.id);
  }, 100);
}

// ES5
function foo() {
  var _this = this

  setTimeout(function() {
    console.log('id: ', _this.id);
  }, 100);
}
```
从上面的代码可以看出，箭头函数里面根本没有自己的 this，而是直接引用外层的 this。

除了 this，arguments、super、new.target 三个变量都指向外层对应的变量，而在箭头函数内部并不存在。

由于箭头函数中没有自己的 this，所以也就不能用 call()、apply()、bind() 这些方法去改变this 的指向。
```javascript
(function () {
  return [
    (() => this.x).bind({ x: 'inner' })
  ]
}).call({ x: 'outer' })
// ["outer"]
```
### 嵌套的箭头函数
先来看一个 ES5 语法的多重嵌套函数
```javascript
function insert(value) {
  return { into: function (array) {
    return { after: function (afterValue) {
      array.splice(array.indexOf(afterValue) + 1, 0, value)
      return array
    }}
  }}
}

insert(2).into([1, 3]).after(1) // [1, 2, 3]
```
箭头函数内部，也可以使用箭头函数，所以上面的方法，也可以用箭头函数改写。
```javascript
let insert = (value) => ({ into: (array) => ({ after: (afterValue) => {
  array.splice(array.indexOf(afterValue) + 1, 0, value)
  return array
}})})

insert(2).into([1, 3]).after(1) // [1, 2, 3]
```