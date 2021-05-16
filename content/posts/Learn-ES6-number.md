---
title: ES6学习笔记-数值的扩展
date: 2017-10-18 23:38:09
tags: 
  - ES6
  - ES6学习笔记
---
## 1. 二进制和八进制表示法
ES6 中用前缀 0b（0B）表示二进制。
```javascript
0b00001111 // 15
0b00001111 === 15 // true
```

ES6 中用前缀 0o（或0O）表示八进制。明确不再允许使用前缀 0 来表示。 
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

## 5. Number.EPSILON
ES6 在 Number 对象上面，引入了一个极小的常量  Number.EPSILON。根据规格，它将返回 1 与大于 1 的最小浮点数之差。

对于 64位浮点数老说,大于 1 的最小浮点数相当于二进制的 1.00..001，小数点后面有 51 个 0

todo...

## 6. 安全整数和 Number.isSafeInteger()
JavaScript 能够准确表示的整数范围是 -2^53 到 2^53 之间（不包含两个断点），超过这个返回，就无法准确表示这个值。
```javascript
Math.pow(2, 53) // 9007199254740993  

9007199254740992 // 9007199254740992
9007199254740993 // 9007199254740992
```
ES6 引入了 Number.MAX_SAFE_INTEGER 和 Number.MIN_SAFE_INTEGER 两个常量，用来表示能够准确这个范围的上下限。
```javascript
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1 // true
Number.MIN_SAFE_INTEGER === -Math.pow(2, 53) + 1 // true
```
Number.isSafeInteger() 用来判断一个整数是否在这个范围内。
```javascript
Number.isSafeInteger('a') // false
Number.isSafeInteger(null) // false
Number.isSafeInteger(NaN) // false
Number.isSafeInteger(Infinity) // false
Number.isSafeInteger(-Infinity) // false

Number.isSafeInteger(3) // true
Number.isSafeInteger(1.2) // false
Number.isSafeInteger(9007199254740990) // true
Number.isSafeInteger(9007199254740992) // false

Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1) // false
Number.isSafeInteger(Number.MIN_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1) // false
```
用代码实现这个函数。
```javascript
Number.isSafeInteger = function (n) {
  return (typeof n === 'number' &&
    Math.round(n) == n &&
    Number.MIN_SAFE_INTEGER <= n &&
    Number.MAX_SAFE_INTEGER >= n)
}
```
实际使用这个函数时，不要直接验证计算的结果，而要验证参与计算的每个值。因为参与计算的值都不能正确表示的话，结果肯定是有问题的。

## 7. Math 对象的扩展
ES6 在 Math 对象新增了 17 个与数学相关的方法。

### Math.trunc
Math.trunc 方法用于去除一个小数的小数部分，返回整数部分。
```javascript
Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.9) // -4
Math.trunc(0.0023) // 0
```
对于非数值，Math.trunc 内部会先使用 Number 方法将其转换为数值。
```javascript
Math.trunc('4.1') // 4
Math.trunc(true) // 1
Math.trunc(false) // 0
Math.trunc(null) // 0
```
对于空值和无法截取整数的值，返回 NaN
```javascript
Math.trunc();         // NaN
Math.trunc(NaN);      // NaN
Math.trunc('foo');    // NaN
Math.trunc(undefined)
```
用代码模拟方法
```javascript
Math.trunc = Math.trunc || function (n) {
  return n < 0 ? Math.ceil(n) : Math.floor(n)
}
```

### Math.sign()
Math.sign() 用来判断一个数是正数、负数、还是零。对于非数值会先转换为数值。

返回五种值
* 参数为正数，返回 +1；
* 参数为负数，返回 -1；
* 参数为0️，返回 0；
* 参数为-0，返回 -0；
* 其它值，返回 NaN；

```javascript
Math.sign(-5) // -1
Math.sign(5) // +1
Math.sign(0) // +0
Math.sign(-0) // -0
Math.sign(NaN) // NaN
```
用代码模拟方法
```javascript
Math.sign = Math.sign || function (n) {
  n = +n
  if (n === 0 || isNaN(n)) {
    return n
  }
  return n > 0 ? +1 : -1
}
```

### Math.cbrt()
Math.cbrt() 用来计算一个树的立方根。
```javascript
Math.cbrt(-1) // -1
Math.cbrt(0)  // 0
Math.cbrt(1)  // 1
Math.cbrt(2)  // 1.2599210498948734
```
用代码模拟方法
```javascript
Math.cbrt = Math.cbrt || function (n) {
  var a = Math.pow(Math.abs(n), 1/3)
  return n > 0 ? a : -a
}
```

### Math.clz32()
JavaScript 的整数使用 32 位二进制形式表示。

Math.clz32() 用来返回一个数的 32 位无符号整数形式有多少个前导 0。
```javascript
Math.clz32(0) // 32
Math.clz32(1) // 31
Math.clz32(1000) // 22
Math.clz32(0b01000000000000000000000000000000) // 1
Math.clz32(0b00100000000000000000000000000000) // 2
```
左移运算符（<<）与Math.clz32() 直接相关
```javascript
Math.clz32(1) // 31
Math.clz32(1 << 1) // 30
Math.clz32(1 << 2) // 29
```
对于小数，Math.clz32() 只考虑整数部分。
```javascript
Math.clz32(3.2) // 30
Math.clz32(3.9) // 30
```
对于空值或其他类型的值，Math.clz32方法会将它们先转为数值，然后再计算。
```javascript
Math.clz32() // 32
Math.clz32(NaN) // 32
Math.clz32(Infinity) // 32
Math.clz32(null) // 32
Math.clz32('foo') // 32
Math.clz32([]) // 32
Math.clz32({}) // 32
Math.clz32(true) // 31
```

### Math.imul()
Math.imul() 返回两个数以 32 位带符号整数形式相乘的结果，返回的也是一个 32 位带符号的整数。
```javascript
Math.imul(2, 4)   // 8
Math.imul(-1, 8)  // -8
Math.imul(-2, -2) // 4
```
todo...

### Math.fround()
Math.fround() 返回一个数的单精度浮点数形式
```javascript
Math.fround(0)     // 0
Math.fround(1)     // 1
Math.fround(1.337) // 1.3370000123977661
Math.fround(1.5)   // 1.5
Math.fround(NaN)   // NaN
```
用代码模拟
```javascript
Math.fround = Math.fround || function (n) {
  return new Float32Array([n])[0]
}
```

### Math.hypot()
Math.hypot() 返回所有参数的平方和的平方根。

只要有一个参数无法转换为数值，返回结果都为 NaN
```javascript
Math.hypot(3, 4) // 5
Math.hypot() // 0
Math.hypot(NaN) // NaN
```

### 对数方法
ES6 新增了 4 个与对数相关的方法
* 1.Math.expm1() 
  Math.expm1(x) 返回 e^x - 1，即 Math.exp(x) - 1
  ```javascript
  Math.expm1(-1) // -0.6321205588285577
  Math.expm1(0)  // 0
  Math.expm1(1)  // 1.718281828459045
  ```
  用代码模拟方法
  ```javascript
  Math.expm1 = Math.expm1 || function(x) {
    return Math.exp(x) - 1;
  }
  ```
  * 2.Math.log1p() 
  Math.log1p(x) 返回1 + x的自然对数，即Math.log(1 + x)。如果x小于-1，返回NaN
  ```javascript
  Math.log1p(1)  // 0.6931471805599453
  Math.log1p(0)  // 0
  Math.log1p(-1) // -Infinity
  Math.log1p(-2) // NaN
  ```
  用代码模拟方法
  ```javascript
  Math.log1p = Math.log1p || function(x) {
    return Math.log(1 + x)
  }
  ```
  * 3.Math.log10()
  Math.log10(x) 返回以10为底的x的对数。如果x小于0，则返回NaN。
  ```javascript
  Math.log10(2)      // 0.3010299956639812
  Math.log10(1)      // 0
  Math.log10(0)      // -Infinity
  Math.log10(-2)     // NaN
  Math.log10(100000) // 5
  ```
  用代码模拟方法
  ```javascript
  Math.log10 = Math.log10 || function(x) {
    return Math.log(x) / Math.LN10
  }
  ```
  * 4.Math.log2()
  Math.log10(x) 返回以2为底的x的对数。如果x小于0，则返回NaN。
  ```javascript
  Math.log2(3)       // 1.584962500721156
  Math.log2(2)       // 1
  Math.log2(1)       // 0
  Math.log2(0)       // -Infinity
  Math.log2(-2)      // NaN
  Math.log2(1024)    // 10
  Math.log2(1 << 29) // 29
  ```
  用代码模拟方法
  ```javascript
  Math.log2 = Math.log2 || function(x) {
    return Math.log(x) / Math.LN2
  }
  ```

### 双曲函数方法
* Math.sinh(x) 返回x的双曲正弦（hyperbolic sine）
* Math.cosh(x) 返回x的双曲余弦（hyperbolic cosine）
* Math.tanh(x) 返回x的双曲正切（hyperbolic tangent）
* Math.asinh(x) 返回x的反双曲正弦（inverse hyperbolic sine）
* Math.acosh(x) 返回x的反双曲余弦（inverse hyperbolic cosine）
* Math.atanh(x) 返回x的反双曲正切（inverse hyperbolic * tangent）

## 8. * Math.signbit()
Math.sign() 用来判断一个值的正负。但是如果参数是 -0，它会返回 -0。

这导致在用来判断符号位的正负的时候，这个方法没什么用。因为 0 和 -0 是相等的，所以判断一个值是 0 还是 -0 是很麻烦的。

目前有一个提案，引入了 Math.signbit()，用来判断一个数的符号位是否设置了。
```javascript
Math.signbit(2) //false
Math.signbit(-2) //true
Math.signbit(0) //false
Math.signbit(-0) //true
```
该方法的返回结果如下
* 如果参数是NaN，返回false
* 如果参数是-0，返回true
* 如果参数是负值，返回true
* 其他情况返回false

## 9. 指数运算符
ES2016 新增了一个指数运算符（**）
```javascript
2 ** 3 // 8
2 ** -2 // 0.25

let a = 2
a **= 3 // 8
```

## 10. * Integer 数据类型
JavaScript 所有数字都保存成 64 位浮点数，这决定了整数的精确程度只能达到 53 个二进制位。大于这个返回，JavaScript 便无法精确表示了。

现在有一个提案，引入新的数据类型 Integer（整数），来解决这个问题。

整数类型的数据只用来显示整数，没有位数的限制。

为了与 Number 类型区别，Integer 类型的数据必须使用后缀 n 表示。
```javascript
1n + 2n // 3n

// 二进制、八进制、十六进制的表示法也一样
0b1101n // 二进制
0o777n // 八进制
0xFFn // 十六禁止
```
typeof 运算符对于 Integer 类型的数据返回 integer
```javascript
typeof 123n // integer
```
JavaScript 原生提供Integer对象，用来生成 Integer 类型的数值。转换规则基本与Number()一致。
```javascript
nteger(123) // 123n
Integer('123') // 123n
Integer(false) // 0n
Integer(true) // 1n
```
以下的用法会报错。
```javascript
new Integer() // TypeError
Integer(undefined) //TypeError
Integer(null) // TypeError
Integer('123n') // SyntaxError
Integer('abc') // SyntaxError
```
在数学运算方面，+、-、*和**的运算和 Number 类型的行为一致。除法运算会舍去小数部分。
```javascript
9n / 5n // 1n
```