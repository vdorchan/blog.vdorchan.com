---
title: ES6学习笔记-变量的解构赋值
date: 2017-10-10 21:41:03
tags: 
  - ES6
  - 学习
  - 变量
  - ES6学习笔记
---
> ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为 **解构**（Destructuring）

ES6 之前， 声明多个变量我们可以这样子

```javascript
var a = 1,
    b = 2,
    c = 3
```
        
而 ES6 增加了解构赋值， 赋值变得更加的高大上了

```javascript
var [a, b, c] = [1, 2, 3]
```


## 1. 数组的解构赋值

下面代码表示，可以从数组中提取值，按照对应位置，对变量赋值。

本质上，这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值

```javascript       
let [a, b, c] = [1, 2, 3]

// 解构不成功的情况下，该变量的值为 undefined
let [a, b, ...c] = [1] // a: 1, b: undefined, c: []

// 不完全解构，也可以成功
let [a, [b], c] = [1, [2, 3], 4] // a: 4, b: 2, c: 4

// 等号的右边不是数组( 不是可遍历的结构 )的话， 会报错
let [foo] = 1

// 对于 Set 结构，也可以使用数组的解构赋值。
let [x, y, z] = new Set(['a', 'b', 'c'])
```

结解构赋值允许指定默认值

当对应的数组成员值为 undefined 时， 默认值才生效, 可以引用已声明的解构赋值的其它变量
```javascript
let [x, y = 2] = [1]
let [x, y = 'b'] = ['a', undefined]
let [x = 1, y = x] = []
```
当默认值为表达式的时候， 那么这个表达式是惰性求值的（要用到的时候才执行）， 像下面的情况， f 函数根本没有执行
```javascript
function f () {
    console.log('aaa')
}
let [x = f()] = [1]
```

## 2. 对象的解构赋值
数组的解构赋值的成员变量是由排序位置决定， 一一对应的， 但是对象没有次序， 所以就要求变量和属性必须同名。
```javascript
// 变量和同名的属性一一对应
let {x, y} = {x: 1, y: 2}
x // 1
y // 2

// 等号右边没有对应的同名属性， 所以取不到值
let {z} = {x: 1, y: 2}
z // undefined

// 上面是下面形式的简写
let {x: x, y: y} =  {x: 1, y: 2}
x // 1
y // 2

let {z: z} = {x: 1, y: 2}
z // undefined

// 这么写就能取到值了, 注意被赋值的是后者
let {x: z} = {x: 1, y: 2}
z // 1
x // error: x is not defined
```
数组的解构赋值可以用于嵌套结构
```javascript
// 用于嵌套结构
let obj = {
    p: [
        'hello',
        { y: 'world' }
    ]
}

let {p: [x, {y}]} = obj
x // 'hello'
y // 'world'

let {p, p: [x, {y}]} = obj
x // 'hello'
y // 'world'
p // [ 'hello', { y: 'world' } ]

// 来多个例子
let obj = {}
let arr = []

// 因为如果以 ‘{’ 开头的话， 被 ‘{’ 和 ‘}’ 包裹起来的区域会被视为一个代码块
// 加上括号以后， 可以避免这个问题
({foo: obj.prop, bar: arr[0] } = {foo: 1, bar: 2})
{foo: obj.prop, bar: arr[0] } = {foo: 1, bar: 2} // 会报错

obj // {prop: 1}
arr // [2]
```
数组的解构赋值可以给变量指定默认值，默认值生效的条件是同名属性为 undefined

```javascript
// 对应的 x 属性 为undefined，默认值生效
let {x: y= 3} = {}
y // 3

// 对应的 x 属性 不为undefined，默认值不生效
let {x: y= 3} = {x: 2}
y // 2

let {msg: msg= 'no msg'} = {}
msg // no msg

let {msg: msg= 'no msg'} = {msg: null}
msg // null

let {msg: msg= 'no msg'} = {msg: 'this is a msg'}
msg // this is a msg

// 使用解构赋值， 可以将对象的方法赋值给变量
let {log, sin, cos} = Math
```
数组其实就是特殊的对象， 所以也可以对数组进行对象属性的解构
```javascript
let arr = [2, 4, 6]

let {0: first, [arr.length - 1]: last} = arr
first // 2
last // 6
```
上面代码中，有个是属于 未学到的 ES6 的内容。当对象的属性为表达式的时候，可以使用 arr[arr.length - 1] 的形式，将表达式放在 方括号中，到了 ES6，使用字面量定义对象的时候，也允许使用表达式作为属性名，同样的也是把表达式放在 方括号中

## 3.字符串的解构赋值

字符串在进行解构赋值的时候会转换成类似数组的对象

```javascript
const [a, b, c, d, e] = 'hello'
a // 'h'

// 字符串有个 length 属性
let {length: len} = 'hello'
len // 5
```

## 4.数值和布尔值的解构赋值

解构赋值的时候， 都会等号右边的值转换成对象

```javascript
let {valueOf: v} = 123
v = Number.prototype.valueOf // true

let {valueOf: v} = true
v = Number.prototype.valueOf // true
```
当右边的值无法转为对象的时候， 会提示语法错误
```javascript
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```

## 5.函数参数的解构赋值

```javascript
function add([x, y]) {
    return x + y
}

add([1, 2]) // 3

// 使用默认值
function move({x = 0, y = 0} = {}) {
    return [x, y]
}

move({x: 1, y: 2}) // [1, 2]
move({}) // [0, 0]
move() // [0, 0]
```
上面的代码是为 x 和 y 指定默认值， 而下面的代码将为函数 move 的参数指定默认值
```javascript
function move({x, y} = {x: 0, y: 0}) {
    return [x, y]
}

move({x: 1, y: 2}) // [1, 2]
move({x: 1}) // [1, undefined]
move({}) // [undefined, undefined]
move() // [0, 0]
```

## 6. 圆括号的问题

```javascript
// 变量声明语句的模式不能使用圆括号
let [(a)] = [1] // 报错

// 下面的代码就可以
[(a)] = [1]

// 函数参数也属于变量声明，所以也不能使用圆括号
function f([(a)]) {return a} // 报错

// 赋值操作的时候，将整个模式或部分模式都放在圆括号中都是会报错的
([a]) = [1]

[({x: x}), y] = [{x: 2}, 3]
```

## 7. 总结用途

**（1）交换变量的值**

```javascript
let x = 1
let y = 2
let [x, y] = [y, x]
```

**（2）从函数返回多个值**

```javascript
function f() {
    return [1, 2, 3]
}

let [first, second, third] = f()
first // 1
second // 2
third // 3

function getName() {
    return {firstName:'allen', lastName:'iverson'}
}

let {firstName, lastName} = getName()
firstName // allen
lastName // iverson
```

**（3）函数参数的定义**

```javascript
function add([x, y]) {
    return x + y
}

add([1, 2]) // 3

function getFullName({firstName, lastName} = {}) {
    return firstName + ' ' + lastName
}

getFullName({firstName: 'allen', lastName: 'iverson'})
```

**（4）提取 JSON 数据**

```javascript
let jsonData = {
    id: 40,
    data: [23, 43, 34]
}

let {id, data: number} = jsonData
id // 40
number // [23, 43, 34]
```

**（5）函数参数的默认值**

```javascript
function F(id, {
    x = false,
    y = true,
    cb = function () {}
    }) {
        // ....
}
```

**（6）遍历Map结构**

```javascript
const map = new Map()
map.set('first', 'hello')
map.set('second', 'world')

for (let [key, value] of map) {
    console.log(key + ' is ' + value)
}

// first is hello
// second is world

// 只获取键名
for (let [key] of map) {
    // ...
}

// 只获取键值
for (let [, value] of map) {
    // ...
}
```

**（7）输入模块的指定方法**

```javascript
const { SourceMapConsumer, SourceNode } = require("source-map")
```

    
