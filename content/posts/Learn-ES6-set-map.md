---
title: Learn-ES6-set-map
date: 2018-02-24 09:59:34
tags: 
  - ES6
  - ES6学习笔记
---
## 1.Set

ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。

Set 本身是一个构造函数，用来生成 Set 数据结构。

```javascript
const s = new Set()

[2, 3, 5, 4, 5, 2, 2].foreach(x => s.add(x))

for (const i of s) {
  console.log(i)
}
// 2 3 5 4
```

上面代码通过add方法向 Set 结构加入成员，结果表明 Set 结构不会添加重复的值。

```javascript
// Set 函数可以接受一个数组（或者具有 iterable 接口的其它数据结构）作为参数， 来初始化
const set = new Set([1, 2, 3, 4, 4])
[...set] // [1 2 3 4]
set.size // 4

const divSet = new Set([...document.querySelectorAll('div')])
divSet.size // 56
```

可以利用 Set 结构去除数组中的重复值

```javascript
[...new Set(arr)]
// 或者
Array.from(new Set(arr))
```

在 Set 对象内部，判断值是否相等用的类似精确相等运算符的方法，主要的区别是在 Set 对象内部，NaN 等于自身，但是精确相等运算符认为 NaN 不等于自身。

```javascript
const set = new Set()

set.add(NaN)
set.add(NaN)

console.log(set) // Set [NaN]

set.add(5)
set.add('5')

console.log(set); // Set [NaN, 5 , '5']
```

### Set 实例的属性和方法

Set 结构的实例有以下属性

* Set.prototype.constructor: 构造函数， 默认就是 Set 函数
* Set.prototype.size: 返回 Set 实例的成员总数

Set 结构的实例有四个操作方法，add(value), delete(value), has(value), clear() 分别用来添加某个值、删除某个值，返回是否拥有某个值，清除所有成员。

### 遍历操作

Set 结构的实例有四个操作方法，keys(), values(), entries() 分别用来获取键名、 键值、 键值对， forEach() 使用回调函数遍历每个成员

Set 的遍历顺序就是插入顺序。

```javascript
let set = new Set(['red', 'green', 'blue']);

for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```

上面代码中，由于 Set 没有键名，或者说键名和键值是一样的，所以 kes 和 values 方法的行为是一致的。

Set 结构的实例默认可遍历，它的默认遍历器生成函数就是它的values方法

这意味着，可以省略values方法，直接用for...of循环遍历 Set。

```javascript
let set = new Set(['red', 'green', 'blue']);

for (const x of set) {
  console.log(x);
}
// red
// green
// blue
```

Set 结构的 forEach 方法的使用和数组的类似

```javascript
let set = new Set(['red', 'green', 'blue']);

set.forEach((value, key) => console.log(key + ': ' + value))
// red: red
// green: green
// blue: blue
```

Set 结构通过间接的使用 map 方法和 filter 方法可以很容易的实现并集（union）、交集（intersect）、差集（difference）。

```javascript
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b])

// 交集
let intersect = new Set([...a]).filter(x => b.has(x))

// 差集
let intersect = new Set([...a]).filter(x => !b.has(x))
```

## 2. WeakSet

## 3. Map

JavaScript 的对象（Object），本质上是键值对的集合，但是传统上，只能用字符串用作键。

ES6 提供了 Map 数据结构，它类似于对象，但是不再只能用字符串用作键，而是各种类型的值（包括对象）都可以当作键。Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。

```javascript
const m = new Map()
const o = { p: 'Hello World' }

m.set(o, 'content') // Map { { p: 'Hello World' } => 'content' }
m.get(o) // 'content'

m.has(o) // true
m.delete(o) // true
m.has(o // false
```

Map 接受数组作为参数，来进行初始化

```javascript
let map = new Map([
  ['name', '张三']，
  ['title', 'author']
])

map.size // 2

map.has('name') // true

map.get('name') // '张三'
map.get('title') // 'author'

```

事实上，不仅仅是数组，任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构都可以当作Map构造函数的参数。这就是说，Set和Map都可以用来生成新的 Map。

```javascript
const set = new Set([
  ['foo', 1],
  ['bar', 2]
]);
const m1 = new Map(set);
m1.get('foo') // 1

const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);
m3.get('baz') // 3
```

Map 中的键实际是跟内存地址绑定的，只要内存地址不一样，即使值相同，也视为不同的键。

```javascript
const map = new Map();

const k1 = ['a'];
const k2 = ['a'];

map
.set(k1, 111)
.set(k2, 222);

map.get(k1) // 111
map.get(k2) // 222
```

### Map 实例的属性和方法

size 属性、 set(key, value)、 get(key)、 has(key)、 delete(key)、 clear()

```javascript
// set方法设置键名key对应的键值为value，然后返回整个 Map 结构。如果key已经有值，则键值会被更新，否则就新生成该键。
// set方法返回的是当前的Map对象，因此可以采用链式写法。
let map = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c')

// size属性返回 Map 结构的成员总数。
map.size // 3

// get方法读取key对应的键值，如果找不到key，返回undefined
map.get(1) // 'a'

// has 方法返回一个布尔值，表示某个键是否在当前 Map 对象之中
map.has(1) // true

// delete方法删除某个键，返回true。如果删除失败，返回false
map.delete(1) // true
map.size // 2
map.has(1) // false

// clear方法清除所有成员，没有返回值
map.clear()
map.size // 0
```

### 遍历方法

Map 和 Set 类似，原声提供了 keys、 values、 entries、 forEach 方法

和 Set 结构一样， 遍历的顺序是插入顺序

```javascript
let map = new Map([
  ['R', 'red'],
  ['G', 'green'],
  ['B', 'blue']
]);

for (let item of map.keys()) {
  console.log(item);
}
// R
// G
// B

for (let item of map.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of map.entries()) {
  console.log(item);
}
// ["R", "red"]
// ["G", "green"]
// ["B", "blue"]

// 或者
for (let [key, vlaue] of map.entries()) {
  console.log(key, vlaue);
}

// "R" "red"
// "G" "green"
// "B" "blue"

// Map 结构的默认遍历器接口就是 entries 方法
for (let [key, vlaue] of map) {
  console.log(key, vlaue);
}

// forEach 方法和数组的类似
map.forEach((value, key, map) => {
  console.log('Key: %s, Value: %2', key, value);
})

//Key: "R", Value: "red"
//Key: "G", Value: "green"
//Key: "B", Value: "blue"
```

## 4.WeekMap