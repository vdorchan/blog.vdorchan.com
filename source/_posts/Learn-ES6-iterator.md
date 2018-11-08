---
title: Learn-ES6-iterator
date: 2018-02-24 14:05:59
tags:
---
## 1.Iterator（遍历器）

Javascript 现有的表示集合的数据结构，出了原本的对象（Object）和数组（Array），ES6 又增加了 Set 和 Map。Iterator 是可以用来统一处理所有不同的数据结构的接口机制。任何数据结构只要部署了 Interator 接口，就可以完成遍历操作。

Iterator 的作用有三个：一是为各种数据结构，提供一个统一的、简便的访问接口；二是使得数据结构的成员能够按某种次序排列；三是 ES6 创造了一种新的遍历命令for...of循环，Iterator 接口主要供for...of消费。

Interator 的遍历过程是首先创建一个指针对象，指向当前数据结构的起始位置，然后调用指针对象的 next 方法，从数据结构的第一个成员开始，依次指向每个成员，直到指向数据结构的结束位置。

每一次调用next方法，都会返回数据结构的当前成员的信息。具体来说，就是返回一个包含value和done两个属性的对象。其中，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。

一个模拟 next 方法返回值的例子

```javascript
const makeInterator = (array) => {
  let nextIndex = 0

  return {
    next() {
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true}
    }
  }
}

var it = makeInterator(['a', 'b'])

it.next() // {value: 'a', done: false}
it.next() // {value: 'b', done: false}
it.next() // {value: undefined, done: true}
it.next() // {value: undefined, done: true}
```

## 2. 默认 Iterator 接口

Iterator 接口的目的，就是为所有数据结构，提供了一种统一的访问机制，即for...of循环。当使用for...of循环遍历某种数据结构时，该循环会自动去寻找 Iterator 接口

一种数据结构只要部署了 Iterator 接口，我们就称这种数据结构是“可遍历的”（iterable）。

下面的代码中，自定义创建的对象因为具有 Symbol.iterator 属性吗，所以该对象是可遍历的。

```javascript
const obj = {
  [Symbol.iterator]: function () {
    return {
      next: function () {
        return {
          value: 1,
          done: true
        }
      }
    }
  }
}
```

原生具备 Iterator 接口的数据结构如下。

* Array
* Map
* Set
* String
* TypedArray
* 函数的 arguments 对象
* NodeList 对象

数组的 Symbol.iterator 属性

```javascript
let arr = ['a','b','c']
let iter = arr[Symbol.iterator]()

iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }
```

下面我们在一个类数组的对象部署 Iterator 接口，我们将直接调用数组的 Symbol.iterator 方法，使其可遍历

```javascript
let arrayLike = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
}

// 假如我们直接使用 for...of 方法遍历
// 会出错，提示该对象不可遍历
for (var v of arrayLike) {
  console.log(v)
}
// TypeError: arrayLike is not iterable

// 我们可以给该对象部署 Iterator 接口
arrayLike.prototype[Symbol.iterator] = Array.prototype[Symbol.iterator]
```

有了遍历器接口，数据结构除了可以用 for...of 循环遍历，也可以使用 while 循环遍历

```javascript
var iterator = arrayLike[Symbol.iterator]()
var result = iterator.next()

while (!result.done) {
  console.log(result.value);

  result = iterator.next()
}
```

## 3.调用 Iterator 接口的场合

* 1.解构赋值

对数组和 Set 结构进行解构赋值时，会默认调用Symbol.iterator方法。

```javascript
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

* 2.扩展运算符（...）也会调用默认的 Iterator 接口。

只要某个数据结构部署了 Iterator 接口，就可以对它使用扩展运算符，将其转为数组。

```javascript
// 例一
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```

* 3.yield*

yield* 后面跟的是可遍历的结构，它会调用该数据结构的遍历器接口。

```javascript
let generator = function* () {
  yield 1
  yield* [2, 3, 4]
  yield 5
}

let iterator = generator()

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }

```

* 4.其它场合

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。下面是一些例子。

* for...of
* Array.from()
* Map(), Set(), WeakMap(), WeakSet()（比如new Map([['a',1],['b',2]])）
* Promise.all()
* Promise.race()

## 4.字符串的 Iterator 接口

字符串是一个类似数组的对象，也原生具有 Iterator 接口。

```javascript
let str = 'hi'

let iterator = str[Symbol.iterator]()

iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }

let strArr = [...str] // [ 'h', 'i' ]
```

## 5. Iterator 接口和 Generator 函数

利用 Generator 函数实现 Symbol.iterator 方法

```javascript
let myIterable = {
  [Symbol.iterator]: function* (){
    yield 'a'
    yield 'b'
    yield 'c'
  }
}

// 或者用简洁写法
let myIterable = {
  * [Symbol.iterator] () {
    yield 'a'
    yield 'b'
    yield 'c'
  }
}

let myIterator = myIterable.[Symbol.iterator]()

myIterator.next() // { value: 'a', done: false }
myIterator.next() // { value: 'b', done: false }
myIterator.next() // { value: 'c', done: false }
myIterator.next() // { value: undefined, done: true }

[...myIterable] // ['a', 'b', 'c']

for (let i of myIterable) {
  console.log(i);
}

// a
// b
// c
```

## 6. 遍历器对象的 return(), throw()

遍历器对象除了必须要有的 next 方法，还可以拥有可选的 return 和 throw 方法

## 7. for...of 循环

一个数据结构只要部署了Symbol.iterator属性，就被视为具有 iterator 接口，就可以用for...of循环遍历它的成员。也就是说，for...of循环内部调用的是数据结构的Symbol.iterator方法

```javascript
// 数组
let arr = ['red', 'green', 'blue']

for (let color of arr) {
  console.log(color);
}

// red
// green
// blue

// for...in 方法循环取得键名
for (let color of arr) {
  console.log(color);
}
// 0
// 1
// 2

// Set 和 Map 结构
let engines = new Set(['Gecko', 'Trident', 'Webkit', 'Webkit'])
for (let e of engines) {
  console.log(e);
}

// Gecko
// Trident
// Webkit

let es6 = new Map()
es6.set('edition', 6)
es6.set('committee', 'TC39')
es6.set('standard', 'ECMA-262')

for (let [name, value] of es6) {
  console.log(name + ': ' + value);
}

// edition: 6
// committee: TC39
// standard: ECMA-262

// 计算生成的数据结构
// 有些数据结构是在现有数据结构的基础上计算生成的。
// 比如，ES6 的数组、Set、Map 都部署了以下三个方法，调用后都返回遍历器对象。
// entries() 返回一个遍历器对象，用来遍历[键名, 键值]组成的数组。对于数组，键名就是索引值；对于 Set，键名与键值相同。Map 结构的 Iterator 接口，默认就是调用entries方法。
// keys() 返回一个遍历器对象，用来遍历所有的键名。
// values() 返回一个遍历器对象，用来遍历所有的键值。
// 这三个方法调用后生成的遍历器对象，所遍历的都是计算生成的数据结构。
let arr = ['a', 'b', 'c'];
for (let pair of arr.entries()) {
  console.log(pair);
}
// [0, 'a']
// [1, 'b']
// [2, 'c']

// 类似数组的对象
// 字符串
let str = "hello";

for (let s of str) {
  console.log(s); // h e l l o
}

// DOM NodeList对象
let paras = document.querySelectorAll("p");

for (let p of paras) {
  p.classList.add("test");
}

// arguments对象
function printArgs() {
  for (let x of arguments) {
    console.log(x);
  }
}
printArgs('a', 'b');

// 对象
let es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
}

// 直接用 for...of 会报错，可以用 for...in
for (let key in es6) {
  console.log(key);
}

// 间接使用 for...of 方法
// 1.借用 Object.keys 方法
for (let key of Object.keys(es6)) {
  console.log(key);
}
// 2.借用 Generator 方法重新包装一下
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]]
  }
}

for (let [key, value] of entries(es6)) {
   console.log(key, '->', value);
}

// for...of 可以配合 break、 continue 和 return 使用
let arr = [1,2,3,4,5,6,7]
for (var n of arr) {
  if (n > 3)
    break;
  console.log(n);
}
// 1
// 2
// 3
```