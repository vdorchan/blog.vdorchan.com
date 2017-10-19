---
title: ES6学习笔记-数值的扩展
date: 2017-10-18 23:38:09
tags: ES6 ES6学习笔记 学习 let const
---
## 1. 二进制和八进制表示法
ES6 中用前缀 0b（0B）表示二进制。
```javascript
0b00001111 // 15
0b00001111 === 15 // true
```

ES6 中用前缀0o（或0O）表示八进制。明确不再允许使用前缀 0 来表示。 
```javascript
0o100 // 64
0o100 === 64 // true
```
使用 Number 方法转换成十进制
```javascript
Number(0o100) // 64
Number(0b00001111) // 15

Number(0o100) === parseInt(0o100) // true
Number(0b00001111) === parseInt(0b00001111) // true
```

## 2. Number.isFinite(), Number.isNaN()
ES6 新增了 Number.isFinite() 和 Number.isNaN() 两个方法。

Number.isFinite() 用来检查一个值是否为有限的（finite）。
```javascript
Number.isFinite(18) // true
Number.isFinite(0.8) // true
Number.isFinite(NaN) // false
Number.isFinite(Infinity) // false 
Number.isFinite(-Infinity) // false
Number.isFinite('foo') // false
Number.isFinite('18') //false 
Number.isFinite(true) //false
```
Number.isNaN() 用来检查一个值是否为有限的（NaN）。
```javascript
Number.isNaN(NaN) // true
Number.isNaN(15) // false
Number.isNaN('15') // false
Number.isNaN(true) // false
Number.isNaN(9/NaN) // true
Number.isNaN('true'/0) // true
Number.isNaN('true'/'true')// true
```
这两个新增的方法和传统的 isFinite() 和 isNaN() 的区别是，传统的两个方法，在进行判断的时候会先用 Number() 将非数值的值转换成数值。而两个新增的方法仅对数值有效，Number.isFinite() 对于非数值都返回 false，Number.isNaN 只有在值为 NaN 的时候才返回 true。 

## 3. Number.parseInt(), Number.parseFloat()
ES6 新增的 Number.parseInt() 和 Number.parseFloat 和传统的 parseInt 和 parseFloat 是一模一样的

移植到 Number 的目的是逐步减少全局性方法，使得语言逐步模块化
```javascript
parseInt('12.34') // 12
parseFloat('12.34%') // 12.34

Number.parseInt('12.34') // 12
Number.parseFloat('12.34%') // 12.34
```

## 4. Number.isInteger()
Number.isInteger() 用来判断一个值是否为整数。需要注意的是，在 JavaScript 内部，类似 3 和 3.0 的存储方式是相同的，所以 3 和 3.0 被视为同一个值。
```javascript
Number.isInteger(18) // true
Number.isInteger(18.0) // true
Number.isInteger('18') // false
```
ES5 可以通过下面的代码，来部署 Number.isInteger()
```javascript
(function (global) {
  const floor = Math.floor,
    isFinite = global.isFinite

  Object.defineProperty(Number, 'isInteger', {
    value: function isInteger(value) {
      return typeof value === 'number' && 
        isFinite(value) && 
        floor(value) === value
    },
    configuable: true,
    enumerable: false,
    writable: false
  })
})(this)
```