---
title: ES6学习笔记-正则的扩展
date: 2017-10-13 15:56:52
tags: 
  - ES6
  - 正则
  - ES6学习笔记
---
正则一直是块难啃的骨头，乍一看就好复杂，各种符号字母交叉也不知道什么意思。编写一个正则，使用的时候是需要适应多种情况的，所以在掌握的不够深的时候，可能写出来的正则就容易出问题了。于是乎，大家就更倾向于复制粘贴大法咯，毕竟有些通用的正则，是能保证正确且足够可靠的。除了校验手机号码、邮箱这些常用的功能之外，其实正则是足够强大应用在很多方面的。正则很深奥，同时又很枯燥，要学好正则，可谓任重而道远啊。

## 1. RegExp 构造函数

通常使用 RegExp 构造函数有两种情况
第一种情况就是参数为字符串，这个时候第二个参数就是正则表达式的修饰符（flag）

```javascript
var regexp = new RegExp('[A-Z]', 'i')
```

另一种情况，参数是一个正则表达式，返回的是这个正则表达式的拷贝

```javascript
var regexp = new RegExp(/A-Z/i)
```

上面的这种情况，是没有没办法传正则表达式的修饰符作为第二个参数，ES6 则允许了这种情况

```javascript
var regexp = new RegExp(/A-z/i, 'g')
regexp.flags // g
```

上面的代码中，第二个参数指定的修饰符，会覆盖掉原有的正则表达式的修饰符

## 2. 字符串的正则方法

to do ...

## 3. u 修饰符

在字符串的扩展里也知道了很多 ES6 之前 JavaScript 是没办法识别大于 0xFFFF 的 Unicode 字符的，所以正则表达式也不能正确的处理大于 0xFFFF 的 Unicode 字符的，ES6 增加了 u 修饰符来解决这个问题。


```javascript
/\ud848\udd04/.test('\ud848') // true
/\ud848\udd04/u.test('\ud848') // false
```

出了上面代码的情况，加了 u 修饰符之后还会改变下面这些代码的行为

* 点标识符

  原本的（.）字符是没办法识别大于 0xFFFF 的 Unicode 字符的，ES6 中可以加上 u 修饰符

```javascript
// "𢄄" 的 UNICODE 编码是 /\ud848\udd04/
var str = '𢄄'
/^.$/.test(str) // false
/^.$/u.test(str) // true
```

* Unicode 字符表示法

  ES6 新增了使用大括号表示 Unicode 字符的方法，正则表达式中必须加上 u 修饰符才能正确识别这种表示方法

  ```javascript
  /\u61/.test('a') // false
  /\u61/u.test('a') // true
  /\u{22104}/u.test('𢄄') // true
  ```

* 量词

  ```javascript
  /𢄄{2}/.test('𢄄𢄄') // false
  /𢄄{2}/u.test('𢄄𢄄') // true
  ```

* 预定义模式

  ```javascript
  /^\S$/.test('𢄄') // false
  /^\S$/u.test('𢄄') // true
  ```

  利用这一点可以写个正确返回字符串长度的函数

  ```javascript
  function codePointLength(text) {
    const result = text.match(/\s\S/gu)
    return result ? result.length : 0
  }

  '𢄄𢄄'.length // 4
  codePointLength('𢄄𢄄') // 2
  ```

* i 修饰符
  有些 Unicode 字符的编码不同，但是字型很相近，比如，\u004B与\u212A都是大写的K

  ```javascript
  /[a-z]/i.test('\u212A') // false
  /[a-z]/iu.test('\u212A') // true
  ```

## 4. y 修饰符

ES6 还为正则表达式添加了 y 修饰符，叫做“粘连”（sticky）修饰符。

和 g 修饰符类似，也是全局匹配，不同点在于， y 修饰符规定后一次匹配必须从剩余位置的第一位开始。看下例子就明白了。

```javascript
var s = 'aaa_aa_a'
var r1 = /a+/g
var r1 = /a+/y

r1.exec(s) // ['aaa']
r2.exec(s) // ['aaa']

// 两者匹配完第一次以后，剩余的字符串为 _aa_a
r1.exec(s) // ['aa']
r2.exec(s) // null

```

使用 lastIndex 属性

```javascript
var s = 'abab'
var r = /a/y

r.lastIndex = 1
r.exec(s) // null

r.lastIndex = 2
r.exec(s) // ['a']
```

单单一个 y 修饰符对 match 方法，只能返回第一个匹配,必须与 g 修饰符配合使用，才能返回所有匹配。

```javascript
'a1b1c1'.match(/a\d/y) // ['a1']
'a1b1c1'.match(/a\d/gy) // ['a1', 'b1', 'c1']
```

y 修饰符的一个应用是提取 token（词元），可以确保匹配之间不会有漏掉的字符。

```javascript
const TOKEN_Y = /\s*(\+|[0-9]+)\s*/y;
const TOKEN_G  = /\s*(\+|[0-9]+)\s*/g;

tokenize(TOKEN_Y, '3 + 4')
// [ '3', '+', '4' ]
tokenize(TOKEN_G, '3 + 4')
// [ '3', '+', '4' ]

tokenize(TOKEN_Y, '3x + 4')
// [ '3' ]
tokenize(TOKEN_G, '3x + 4')
// [ '3', '+', '4' ]

function tokenize(TOKEN_REGEX, str) {
  let result = [];
  let match;
  while (match = TOKEN_REGEX.exec(str)) {
    result.push(match[1]);
  }
  return result;
}
```

上面代码中，使用 g 修饰符的正则表达式会忽略非法字符，而 y 修饰符不会，这样就很容易发现错误。

## 5. sticky 属性

ES6 新增的 sticky 属性用来表示是否设置了 y 修饰符。

## 6. flags 属性

ES6 新增的 flags 属性返回正则表达式的修饰符。

```javascript
var r = /a/ig

// ES5 的 source 属性返回正则表达式的正文
r.source // 'a'

// ES6 的 flags 属性返回正则表达式的正文
r.flags // 'a'
```


## 7. * s 修饰符： dotALL 模式

正则表达式中，（.）代表任意的单个字符，行终止符（line terminator character）除外。

现在有个提案，使用 s 修饰符的话，正则表达式中的（.）就能匹配包括行终止符的任意单个字符。

```javascript
/foo.bar/.test('foo/nbar/') // false
// 一种变通的写法
/foo[^]bar/.test('foo/nbar/') // true

/foo.bar/s.test('foo/nbar/') // true
```

另外，还引入一个 dotAll 属性用来表示是否处在了 dotAll 模式。

## 8. * 后行断言

“先行断言”指的是，x 必须在 y 前面才匹配，写做 /x(?=y)/

“先行否定断言”指的是，x 只有不在 y 前面才匹配，写做 /x(?!y)/

括号中的部分不计入返回结果。


```javascript
var s = '15% of 100 is 15'

s.match(/\d+(?=%)/g) // ["15"]

s.match(/\d+(?!%)/g) // ["1", "100", "15"]
```


目前有个提案是引入后行断言。

“后行断言”指的是，x 必须在 y 后面才匹配，写做 /(?<=y)x/

“后行否定断言”指的是，x 只有不在 y 后面才匹配，写做 /(?<!y)x/

```javascript
var s = 'there are 4 shoes, they are worth about $60'

s.exec(/(?<=\$)d+/) // 60
s.exec(/(?<!\$)d+/) // 4
```

后行断言的匹配顺序和通常的数序是反过来的

正常情况下

```javascript
/^(\d+)(\d+)$/.exec('1053') // ["1053", "105", "3"]
```

上面正则匹配的顺序是，先是整个正则匹配成功的结果，然后是第一个括号、第二个括号对应的匹配成功的结果。

但如果是后行断言的话

```javascript
/(?<=(\d+)(\d+)$/.exec('1053') // ["", "1", "053"]
```

todo...
再看个例子

```javascript
/(?<=(o)d\1)r/.exec('hodor')  // null
/(?<=\1d(o))r/.exec('hodor')  // ["r", "o"]
```

从上面的代码可以看出反斜杠引用的 \1 也要按照相反的顺序来放（\1 的作用是在正则表达式内部获取分组匹配）

## 9. * Unicode 属性类

有个提案是引入了一种新的类的写法 \p{...} 和 \P{...}，允许正则表达式匹配符合 Unicode 某种属性的所有字符


Unicode 属性类要指定属性名和属性值，因为这两种类只对 Unicode 有效，所以必须要加上前面学到的 u 修饰符

所以这两个类的正则表达式格式是这样子的

```javascript
const r = /\p{UnicodePropertyName=UnicodePropertyValue}/
const r = /\P{UnicodePropertyName=UnicodePropertyValue}/
```

其中 \P{...} 是 \p 的反向匹配，即匹配所有不满足条件的字符。

对于某些属性，可以只写属性名

```javascript
const r = /\p{UnicodePropertyName}/
```

各种应用

```javascript
// 指定匹配一个希腊文字母
const regexGreekSymbol = /\p{Script=Greek}/u

const regex = /^\p{Decimal_Number}+$/u

// 匹配所有数字
const regex = /^\p{Number}+$/u

// 匹配所有的箭头字符
const regexArrows = /^\p{Block=Arrows}+$/u
```

## 10. * 具名组匹配

先来看一个分组匹配的例子

```javascript
const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/

const matchObj = RE_DATE.exec('2017-10-18')
const year = matchObj[0]
const month = matchObj[1]
const day = matchObj[2]
```

上面代码每一组的匹配是通过序号来获取的，如果组的顺序变了，引用的时候，还要更改序号。另外，每一组的匹配含义也不容易看出来。

现在则有个“具名组匹配”（Named Capture Groups）的提案，允许为每一组匹配指定一个名字，既便于阅读，又便于引用。

“具名组匹配”在圆括号内部，模式的头部添加“问号 + 尖括号 + 组名”（?<year>）。然后就可以在exec方法返回结果的groups属性上引用该组名。同时，数字序号（matchObj[1]）依然有效。

```javascript
const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/

const matchObj = RE_DATE.exec('2017-10-18')
const year = matchObj.groups.year
const month = matchObj.groups.month
const day = matchObj.groups.day
```

如果具名组没有匹配，那么对应的 groups 对象属性会是 undefined。

** 解构赋值和替换 ** 
利用具名组匹配可以使用解构赋值从匹配结果中为变量赋值。

```javascript
const {group: {one, two}} = /^(?<one>.*):(?<two>.*)/.exec('bar:foo')

one // bar
two // foo
```

字符串替换时，可以使用 $<组名> 引用具名组。

```javascript
const r = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
'2017-10-18'.replace(r, '$<day>/$<month>/$<year>') // 18/10/2017
```

replace 方法的第二个参数可以是函数。

```javascript
const r = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/

'2017-10-18'.replace(r, (
  matched, // 整个匹配结果
  capture1, //第一个组匹配
  capture2, //第二个组匹配
  capture3, //第三个组匹配
  position, // 匹配开始的位置 0
  S, // 原字符串
  groups // 具名组构成的一个对象 {year, month, day}
) => {
  let groups = {day, month, year} = arg[args.length - 1]
  return `${day}/${month}/${year}`
}
```

具名组匹配在原来的基础上，新增了最后一个函数参数：具名组构成的一个对象，函数内部可以对其解构赋值。

** 引用　**
如果要在正则表达式内部引用某个“具名组匹配”，可以使用 \k<组名>　的写法

```javascript
const r = /(?<word>\w+)!\k<word>/
r.test('abc!abc') // true
r.test('abc!ab') // false
```

数字引用　\1　也依然有效

```javascript
const r = /(?<word>\w+)!\1/
r.test('abc!abc') // true
r.test('abc!ab') // false
```