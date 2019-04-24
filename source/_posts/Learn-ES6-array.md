---
title: ES6学习笔记-数组的扩展
date: 2017-10-26 16:20:54
tags: 
  - ES6
  - 学习
  - 数组
  - ES6学习笔记
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
  return p.textContent.length > 100;
});

// arguments 对象
function foo() {
  const args = Array.from(arguments)
  // ...
}
```

只要是部署了 Iterator 接口的数据结构，Array.from 都能将其转换为数组
```JavaScript
Array.from('hello')
// ['h', 'e', 'l', 'l', 'o']

let namesSet = new Set(['a', 'b'])
Array.from(namesSet) // ['a', 'b']
```

扩展运算符背后调用的是遍历器接口（System.interator），如果一个对象没有该接口，就无法进行转换。
而 Array.from 方法支持所有类似数组的对象。其特征只有一个，即必须有 length 属性。任何具有 length 属性的对象，都可以使用 Array.from 将其转换为数组
```javascript
Array.from({length: 3}) // [undefined, undefined, undefined]
```

没有部署改方法的浏览器，可以使用 slice 方法代替
```javascript
const toArray = () => {
  return Array.from ? Array.from : obj => [].slice.call(obj)
```

Array.from还可以接受第二个参数，作用类似于数组的map方法，用来对每个元素进行处理，将处理后的值放入返回的数组。
```javascript
Array.from(ArrayLike, x => x * x)
// 等同于
Array.from(ArrayLike).map(x => x * x)
```

如果map函数里面用到了this关键字，还可以传入Array.from的第三个参数，用来绑定this。

## 3. Array.of()
数组构造函数 Array() 有个不足，根据传递参数数量的不同，返回结果会有所不同.
```javascript
Array() // []
Array(3) // [, , ,]
Array(3, 11, 8) // [3, 11, 8]
```
参数个数只有一个的时候，指的是数组的数量，不少于两个参数的时候，才会返回由参数组成的数组。

而 Array.of() 则可以替代 Array()，弥补其不足的地方。
```javascript
Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```
Array.of总是返回参数值组成的数组。如果没有参数，就返回一个空数组。

Array.of 方法可以用下面的代码模拟实现
```javascript
function ArrayOf() {
  return [].slice.call(arguments)
}
```

## 4.数组实例的 copyWithin()
数组实例的 copyWithin 在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。也就是说，使用这个方法，会修改当前数组。
```javascript
Array.prototype.copyWithin(target, start = 0, end = this.length)
```
例子
```javascript
[1, 2, 3, 4, 5].copyWithin(0, 3)
// [4, 5, 3, 4, 5]

// 将3号位复制到0号位
[1, 2, 3, 4, 5].copyWithin(0, 3, 4)
// [4, 2, 3, 4, 5]

// -2相当于3号位，-1相当于4号位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]

// 将3号位复制到0号位
[].copyWithin.call({length: 5, 3: 1}, 0, 3)
// {0: 1, 3: 1, length: 5}
```

## 5.数组实例的 find() 和 findIndex()
数组实例的find方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员，然后返回该成员。如果没有符合条件的成员，则返回undefined。
```javascript
[1, 2, 3, 4].find(x => x >2) // 3
```
findIndex 方法和 find 方法类似，只不过返回值改为符合值的位置。如果没有则返回 -1。
```javascript
[1, 2, 3, 4].findIndex(x => x >2) // 2
```
这两个方法都可以接受第二个参数，用来绑定回调函数的this对象。
```javascript
const Jay = {age: 18}

[12, 14, 20, 25].find(function (age){ 
  return age> this.age
}, Jay)
// 20
```

这两个方法都可以发现NaN，弥补了数组的indexOf方法的不足。
```javascript
[NaN].indexOf(NaN) // -1

[NaN].findIndex(x => Object(Nan, x))  // 0
```
上面代码中，indexOf方法无法识别数组的NaN成员，但是findIndex方法可以借助Object.is方法做到。

## 6. 数组实例的 fill()
fill 方法使用给定的值，填充一个数组
```javascript
['a', 'b', 'c'].fill(6)
// [6, ,6, 6]

new Array(3).fill(7)
// [7, 7, 7]
```
上面代码表明，fill方法用于空数组的初始化非常方便。数组中已有的元素，会被全部抹去。

fill方法还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置。
```javascript
['a', 'b', 'c'].fill(6, 1, 2)
// ['a', 6, 'c']
```

## 7. 数组实例的 entries()，keys() 和 values()
ES6 提供三个新的方法——entries()，keys()和values()——用于遍历数组。它们都返回一个遍历器对象,可以用for...of循环进行遍历，唯一的区别是keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。
```javascript
for (const index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (const elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (const [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 'a'
// 1 'b'
```

如果不使用for...of循环，可以手动调用遍历器对象的next方法，进行遍历。
```javascript
const letter = ['a', 'b', 'c']
const entries = letter.entries()
console.log(entries.next().value); // 0 'a'
console.log(entries.next().value); // 1 'b'
console.log(entries.next().value); // 2 'c'
```

## 8. 数组实例的 includes()
Array.prototype.includes方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似。ES2016 引入了该方法。
```javascript
[1, 2, 3].includes(1) // true
[1, 2, 3].includes(4) // false
[1, 2, NaN].includes(NaN) // true
```
该方法的第二个参数表示搜索的起始位置，默认为0。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为3），则会重置为从0开始。
```javascript
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
```

includes 方法假如运行唤醒不支持，可以部署一个简易的替代版本。
```javascript
const contains = (() =>
  Array.prototype.includes
    ? (arr, value) => arr.includes(value)
    : (arr, value) => arr.some(el => el === value)
)();
contains(['foo', 'bar'], 'baz'); // => false
```

## 9. 
数组的空位指，数组的某一个位置没有任何值。比如，Array构造函数返回的数组都是空位。
```javascript
Array(3) // [, , ,]
```

注意，空位不是undefined，一个位置的值等于undefined，依然是有值的。空位是没有任何值，in运算符可以说明这一点。
```javascript
0 in [undefined, undefined, undefined] // true
0 in [, , ,] // false
```
上面代码说明，第一个数组的 0 号位置是有值的，第二个数组的 0 号位置没有值。

ES5 对空位的处理，已经很不一致了，大多数情况下会忽略空位。

* forEach(), filter(), reduce(), every() 和some()都会跳过空位。
* map()会跳过空位，但会保留这个值
* join()和toString()会将空位视为undefined，而undefined和null会被处理成空字符串。
```javascript
[,'a'].forEach((x,i) => console.log(i)); // 1

// filter方法
['a',,'b'].filter(x => true) // ['a','b']

// every方法
[,'a'].every(x => x==='a') // true

// reduce方法
[1,,2].reduce((x,y) => return x+y) // 3

// some方法
[,'a'].some(x => x !== 'a') // false

// map方法
[,'a'].map(x => 1) // [,1]

// join方法
[,'a',undefined,null].join('#') // "#a##"

// toString方法
[,'a',undefined,null].toString() // ",a,,"
```

ES6 则是明确将空位转为undefined。

Array.from方法会将数组的空位，转为undefined，也就是说，这个方法不会忽略空位。

由于空位的处理规则非常不统一，所以建议避免出现空位。