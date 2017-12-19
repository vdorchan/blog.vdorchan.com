---
title: ES6学习笔记-数组的扩展
date: 2017-10-26 16:20:54
tags: ES6 ES6学习笔记 学习 变量 数组
---
## 1. 扩展运算符
扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。
```javascript
console.log(...[1, 2, 3]);
// 1 2 3

console.log(1, ...[2, 3, 4], 5);
// 1 2 3 4 5

[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]
```
该运算符可以将一个数组变为参数序列。
```javascript
function push(array, ...items) {
  array.push(...items)
}

function add(x, y) {
  return x + y
}
const number = [2, 3]
add(...number) // 5
```
扩展运算符后面可以放置表达式。
```javascript
const arr = [
  ...( x > 0 ? ['a'] : []),
  'b'
]
```
如果扩展运算符后面是一个空数组，则不产生任何效果。
```javascript
[...[], 1]
// [1]
```
### 替代数组的 apply 方法
扩展运算符可以代替 apply 方法，将数组转转为函数的参数。
```javascript
function f(x, y, z) {
  // ...
}

const arr = [1, 2, 3]

// ES5
f.apply(null, arr)

// ES6
f(...arr)
```
使用 Math.max 方法的时候也可以用上。
```javascript
// ES5
Math.max.apply([1, 2, 3])

// ES6
Math.max(...[1, 2, 3])
```
push 函数的参数不支持数组，ES5 要使用 apply 变通使用，ES6 也可以使用扩展运算符代替。
```javascript
const arr1 = [1, 2, 3]
const arr2 = [4, 5]

// ES5
Array.prototype.push.apply(arr1, arr2)

// ES6
arr1.push(...arr2)
```
### 扩展运算符的运用 
**(1)复制数组**
我们都知道 JavaScript 直接复制数组的，只是复制了指向底层数据的指针，而不是赋值一个全新的数组
```javascript
let arr1 = [1, 2]
let arr2 = arr1

arr1[0] = 0
arr2 // [0, 2]
```
我们可以利用合并数组的方法 concat()　来复制数组。
```javascript
let arr1 = [1, 2]
let arr2 = arr1.concat()

arr1[0] = 0
arr2 // [1, 2]
```　　
而有了扩展运算符之后，写法就更简单了。
```javascript
let arr1 = [1, 2]
// 写法1
let arr2 = [...arr1]
// 写法2
let [...arr2] = arr1

```
**(2)合并数组**
```javascript
let arr1 = [1, 2]
let arr2 = [3, 4, 5]

// ES5
arr1.concat(arr2)

// ES6
[...arr1, ...arr2]
[1, 2, ...arr2]
```
**(3)与解构赋值结合**
```javascript
// ES5
a = list[0], rest = list.slice(1)
// ES6
[a, ...rest] = list
```
另外一些例子。
```javascript
const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

const [first, ...rest] = [];
first // undefined
rest  // []

const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []
```
如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错。
**(4)字符串**
扩展运算符可以将字符串转为真正的数组
```javascript
[...'Hello']
// ["H", "e", "l", "l", "o"]
```
将字符串转换成数组的话，那就能够正确识别 4 个字节的 Unicode 字符了。
```javascript
// '𢄄'的 Unicode 写法是　'\ud848\udd04'
'𢄄'.length　// 2
[...'𢄄'].length // 1
```
所以可以利用扩展运算符来获取字符串正确的长度。
```javascript
function length(str) {
  return [...str].length
}+
```
所以凡是涉及到四个字节 Unicode 字符的函数，最好都用扩展运算符来写。
**(5)实现了 Interator 接口的对象**
扩展运算符内部调用的是数据结构的 Iterator 接口，因此只要具有 Iterator 接口的对象，都可以使用扩展运算符

任何 Interator 接口的对象，都可以用扩展运算符转换成数组。
```javascript
let nodeList = document.querySelectorAll('div')
[...nodeList] // [<div>, <div>, <div>]
```
上面的 querySelectorAll 方法返回的是一个 nodeList 对象，可以使用扩展运算符转换成数组的原因，是因为 nodeList 对象实现了 Interator 方法。

Map 结构
```javascript
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three']
])
```
Generator 函数运行后，返回一个遍历器对象，因此也可以使用扩展运算符
```javascript
const go = function*(params) {
  yield 1;
  yield 2;
  yield 3;
}
[...go()] // [1, 2, 3]
```
上面代码中，变量go是一个 Generator 函数，执行后返回的是一个遍历器对象，对这个遍历器对象执行扩展运算符，就会将内部遍历得到的值，转为一个数组

如果对没有 Iterator 接口的对象，使用扩展运算符，将会报错。
```javascript
onst obj = {a: 1, b: 2};
let arr = [...obj]; // TypeError: Cannot spread non-iterable object
```

## 2. Array.from()
Array.from 方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括ES6新增的数据结构Set和Map）。

下面是一个类似数组的对象，Array.from将它转为真正的数组。
```javascript
let arrayLike = {
  '0': 'a',
  '1': 'b',
  '3': 'c',
  length: 3
}

// ES5
var arr1 = [].slice.call(arrayLike) // ['a', 'b', 'c']

// ES6
var arr1 = Array.from(arrayLike) // ['a', 'b', 'c']
```
像是 NodeList 集合以及函数内部的 arguments 对象，都可以使用 Array.from() 将它们转换为真正的数组。
```javascript
// NodeList 对象
let ps = document.querySelectorAll('div')
// 转换成数组以后可以使用 foreach 方法遍历
Array.from(ps).forEach(p => {
  
});

// arguments 对象
function foo() {
  const args = Array.from(arguments)
  // ...
}
```