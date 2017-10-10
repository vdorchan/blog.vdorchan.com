---
title: ES6学习笔记-let与const
date: 2017-10-10 21:41:03
tags: ES6 learn 笔记 学习
---
## 背景
ES6 即 ECMAScript 6.0 的简称，是 JavaScript 的下一代标准，在2015年6月正式发布了。

### ECMAScript 和 JavaScript 的关系
ECMASCript 和 JavaScript 的关系，在这里简单的概括下。

>*NetScape* 先创造了 JavaScript，然后，为了让这种语言成为国际标准，所以决定将其提交给 *ECMA (国际标准化组织)*。*ECMA* 在次年便发布了 ECMAScript 的 1.0 版。标准是针对 JAVAScript 语言制定的，但因为 JAVA 是 Sun 公司的商标，根据授权协议，只有 *NetScape* 公司可以使用 JAVAScript 这个名字，而且也为了让大家知道，这门语言的制定者是 *ECMA*，不是 *NetScape*。

因此，ECMAScript 和 JavaScript 的关系是，前者是后者的规格，后者是前者的一种实现。由于 JavaScript 的历史原因和市场原因，现实中我们只用 ECMAScript 称呼标准，而使用 JavaScript 来称呼这个语言。

## ES6
在 ECMASCript 诞生后的很长一段时间里, 其并没有多大的变化。ES5 在 2011 年发布之后也没有得到广泛的支持，很多开发者都还是用 ES3 在写页面。这期间很多浏览器厂商都在争相进行自己的语言发展，这也导致了很多的兼容问题。这期间诞生了 jQuery，一个 JavaScript 库， 简化了 JavaScript 编程，同时也帮助开发者解决很多跨浏览器的兼容问题。

而 ECMAScript 本身，2012那年开始，大家开始推动淘汰旧版本IE的支持，于是，大家可以开始用 ES5 来写代码了。同时，一个新的标准规范也开始启动，那个负责制定 ECMAScript 规范草案的组织，*委员会 TC39*，在 ES6 正式发布之前，将其改名为 ECMAScript 2015，在2015年6月发布。委员会同时也决定在每年的六月发布新的标准。在写这篇文章的时候，已经是2017年，ES 2017 在今年六月份也如约发布了。

现在，浏览器对这些新的标准规范都跟进得很及时，JavaScript 发展的比以前规范得很多，现在，你可以继续用 jQuery 去编写页面，但一定不能不去学习和了解这些新的标准规范。在前端领域，JavaScript 应该是最基础的技能了。而用新的标准写代码会更加的规范。

当然，兼容问题还是有的，加上国内低版本的 IE 仍有一定的使用率。这时，就需要 借助[Babel](https://babeljs.io/) 这类工具了。


## let和const

### 1. let

在 ES6 之前，我们使用 var 命令声明变量。

let 命令可以用来代替 var，用来声明变量，用 let 声明的时候，有块作用域，所声明的变量只在 let 命令所在的代码块内有效，不存在变量提升。

因为有暂时性死区，只要当前作用域中有 let 命令，该变量就会被锁定，在声明之前都不允许使用，不允许重复声明。

这些特性可以减少很多运行时错误，能让代码更加的清晰。

```javascript
// 用 var 声明 i 的时候，因为没有块作用域，所以在全局范围内都是有效的
// 所以 i 变量都是同一个，最后输出的时候，i 为 10，输出的都是 10
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10

// 而用 let 声明的 i 仅在本轮的循环中有效。
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6

// ES6 为 let 命令新增了 块作用域
function f1() {
  let n = 5;
  if (true) {
    let n = 10; // 这里声明的 n 只在 { 和 } 中间的代码块中有效
  }
  console.log(n); // 5
}
```

### 2. const

const 声明的是一个只读的常量，该常量不允许被覆盖或重新赋值。

const 声明的变量不变的不是值，而是值所指向的内存地址不变，而对象、数组等复合类型的变量保存的只是一个指向内存地址的指针，所以其指向的数据结构是可变的。

```javascript
const PI = 3.1415;
PI // 3.1415
PI = 3 // 报错

const player = ['curry']
player.push('kobe') // 正常 

const curry = {name: 'curry'}
curry.number = 30 // 正常
```

### 3. 顶层对象的属性

在用 var 声明全局变量的时候，同时也会给顶层对象的属性赋值，而 let 和 const 命令声明的全局变量则不属于顶层对象的属性。

```javascript
var foo = 'a'
window.foo // 'a' // 全局作用域下声明的变量，将作为顶层对象的属性

let foo = 'a'
window.foo // undfined 
```

### 4. 总结

在 ES6 之前，用 var 命令声明变量的时候是不够严格的，var命令声明变量的时候存在变量提升，而 let 和 const 则不存在。let 和 const 有块作用域，所以不会出现内层变量覆盖外层变量的情况，计数变量也不会泄漏为全局变量, 根据场景正确使用 let 和 const 将使代码变得更加清晰。

那 let 和 const 该如何选择使用呢，大部分的说法都是倾向于优先使用 const 的，除非声明的变量在之后的代码中需要重新赋值，则使用let。而 var 基本上可以抛弃了，大多数应用场景实在想不出使用 var 还有什么好处。

