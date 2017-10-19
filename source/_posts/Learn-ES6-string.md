---
title: ES6学习笔记-字符串的扩展
date: 2017-10-11 14:49:31
tags: ES6 ES6学习笔记 学习 字符串
---
## 1. 字符的 Unicode 表示法
>unicode 是一个字符集，包含了世界上几乎所有的字符，并且为每个字符分配一个唯一的**码点**，unicode 的出现是为了能在计算机上更好的处理多国家的语言文字。unicode 每年都还在更新，每年都会加入很多新的字符。广义的 unicode 还包括了一系列的编码规则（UTF-8，UTF-16，UTF-32等等）。

JavaScript 有以下表示字符的方法

```javascript
'\z' === 'z'  // true
'\172' === 'z' // true
'\x7A' === 'z' // true
'\u007A' === 'z' // true
```

其中 JavaScript 允许采用 \uxxxx 形式表示一个字符，其中 xxxx 表示字符的 Unicode 码点

```javascript
'\u0061' \\ a
'\u2210' \\ ∐
```

但是当表示的字符的 Unicode 码点超过 0xFFFF 的时候，也就是从第 65537 （2的16次方） 个开始, 就没办法正常表示字符了

```javascript
'\u22104' \\ ∐4

// 采用这种方式可以正确表达字符
'\ud848\udd04' \\ 𢄄
```

而 ES6 中只要将码点放入大括号中，就能正确表示该字符

```javascript
'\u{22104}' \\ 𢄄
'\u{61}\u{62}\u{63}' \\ abc
```


## 2. codePointAt()

JavaScript内部使用 utf-16 的格式储存字符，每个字符固定长度为 2 字节。而字符码点大于 0xFFFF 的字符，需要 4 个字节的空间，JavaScript会认为它们是两个字符

```javascript 
var a = '𢄄'
a.length // 2
a.charAt(0) // ''
a.charAt(1) // ''
a.charCodeAt(0) // 55368 
a.charCodeAt(1) // 56580
```

'𢄄' 的码点为 0x22104（十进制为 139524），其 UTF-16 编码为 0xd848 0xdd04（十进制为 55368 56580）， 占用 4 个字节的储存空间，JavaScript 会误判这个字符长度为 2，charAt 方法无法读取整个字符，charCodeAt 也只能返回前两个字节和后两个字节的值。

ES6 提供了 codePointAt 方法，可以正确处理 4 个字节储存的字符

```javascript 
var a = '𢄄'
a.codePointAt(0) // 139524
a.codePointAt(0).toString(16) // 22104
a.codePointAt(1) // 56580

var b = '𢄄a'
a.codePointAt(0) // 139524
a.codePointAt(1) // 56580
a.codePointAt(2) // 97 a码点为 0x61 十进制为 97
```

通过上面的代码可以看到，字母 a 正确的序号位置应该是 1 才对的，然而，需要传给 codePointAt 方法的参数为 2 的时候才能获取到正确的十进制码点

解决这个问题可以使用 for...of 循环

```javascript 
var b = '𢄄a'
for (let ch of b) {
    console.log(ch.codePointAt(0).toString(16))
}

// 22104
// 61
```

codePointAt 方法还可以用来测试一个字符是由 2 个字符还是 4 个字符组成的

```javascript 
function is32Bit(str) {
    return str.codePointAt(0) > 0xFFFF
}

is32Bit('a') // false
is32Bit('𢄄') // false
```

## 3. String.fromCodePoint()

ES5 提供了 String.fromCharCode 方法用于从码点返回字符

```javascript 
String.fromCharCode(97) // a
String.fromCharCode(0x61) // a
String.fromCharCode(0x61, 0x62) // ab

String.fromCharCode(0x22104) // "℄"
String.fromCharCode(0x2104) // "℄"
```

但是当把这个方法用在 4 个字节长度的字符上的时候，就会出现错误了，返回的字符不是我们所期待的字符。这是因为 String.fromCharCode 方法不能识别大于 0xFFFF 的码点，所以 0x22104 就会发生溢出，最高位的 2 被舍弃了，所以最后返回的是码点 0x2104 的字符。

而 ES6 的 String.fromCodePoint 方法就是用来解决这个问题的

```javascript 
String.fromCodePoint(97) // a
String.fromCodePoint(0x22104) // 𢄄
String.fromCodePoint(0x22104, 97) // 𢄄a
```

## 4. 字符串的遍历器接口

ES6 为字符串添加了遍历器接口，使得字符串可以被 for...of 循环遍历

```javascript 
for (let codePoint of 'foo') {
    console.log(codePoint)
}

// 'f'
// 'o'
// 'o'
```

for...of 循环有个优点就是能正确识别 4 个字节长度的字符，而传统的 for 循环是没办法做到的

```javascript 
var b = '𢄄a'
for (let ch of b) {
    console.log(ch.codePointAt(0).toString(16))
}

// 22104
// 61
```

## 5. * at()

ES5 有个方法用于返回给定位置的字符，该方法同样不能识别大于 0xFFFF 的字符

```javascript 
'abc'.charAt(0) // a
'𢄄'.charAt(0) // ''
```

提案提出一个字符串实例的 at 方法，可以识别大于 0xFFFF 的字符

```javascript     
'abc'.at(0) // a
'𢄄'.at(0) // ''
```

## 6. normalize()

许多欧洲语言有语调符号和重音符号。为了表示它们，Unicode 提供了两种方法。一种是直接提供带重音符号的字符，比如Ǒ（\u01D1）。另一种是提供合成符号（combining character），即原字符与重音符号的合成，两个字符合成一个字符，比如O（\u004F）和ˇ（\u030C）合成Ǒ（\u004F\u030C）

这两种表示方法，在视觉和语义上都等价，但是 JavaScript 不能识别。

```javascript
'\u01D1'==='\u004F\u030C' //false

'\u01D1'.length // 1
'\u004F\u030C'.length // 2
```

ES6 提供字符串实例的normalize()方法，用来将字符的不同表示方法统一为同样的形式，这称为 Unicode 正规化。

```javascript
'\u01D1'.normalize() === '\u004F\u030C'.normalize() // true
```
normalize方法可以接受一个参数来指定normalize的方式，参数的四个可选值如下。

* NFC，默认参数，表示“标准等价合成”（Normalization Form Canonical Composition），返回多个简单字符的合成字符。所谓“标准等价”指的是视觉和语义上的等价。
* NFD，表示“标准等价分解”（Normalization Form Canonical Decomposition），即在标准等价的前提下，返回合成字符分解的多个简单字符。
* NFKC，表示“兼容等价合成”（Normalization Form Compatibility Composition），返回合成字符。所谓“兼容等价”指的是语义上存在等价，但视觉上不等价，比如“囍”和“喜喜”。（这只是用来举例，normalize方法不能识别中文。）
* NFKD，表示“兼容等价分解”（Normalization Form Compatibility Decomposition），即在兼容等价的前提下，返回合成字符分解的多个简单字符。

```javascript
'\u004F\u030C'.normalize('NFC').length // 1
'\u004F\u030C'.normalize('NFD').length // 2
```

## 7.includes(), startsWith(), endsWith()
ES6 之前有个 indexOf 方法，用于确定一个字符串是否包含在另一个字符串中。

ES6 提供了三种新的方法
* includes()：返回布尔值，表示是否找到了参数字符串
* startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。
* endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部。
```javascript
const str = 'Hello World!'

str.includes('o') // true
str.startsWith('Hello') // true
str.endsWidth('!') // true
```
三种方法都支持第二个参数，表示开始搜索的位置
```javascript
str.startsWith('W', 6) // true
str.startsWith('W', 7) // false
str.endWith('!', 1) // true
str.includes('W', 7) // false
```

## 8. repeat()
repeat 将原字符串重复 n 次以后，返回一个新的字符串
```javascript
'x'.repeat(3) // xxx
```
参数如果是小数，将取证
```javascript
'x'.repeat(2.5) // xxx
```
如果参数是负数或者是 Infinity，会报错
```javascript
'x'.repeat(Infinity) // RangeError
'x'.repeat(-1) // RangeError
```
如果参数是0到-1之间的话，取整为 0, 如果参数为 NaN，也等同于 0
```javascript
'x'.repeat(-0.9) // ''
'x'.repeat(NaN) // ''
'x'.repeat(0) // ''
```
如果参数是字符串，则先转换成数字
```javascript
'x'.repeat('x') // ''
'x'.repeat('3') // 333
```

## 9. padStart()，padEnd()
ES2017 引入了字符串补全的功能，如果某个字符不够指定长度，会在头部或者尾部不全。padStart 方法用于头部补全，padEnd 用于尾部补全
```javascript
'x'.padStart(5, 'ab') // ababx
'x'.padEnd(5, 'ab') // xabab
```
上面代码中，第一个参数用来用来指定输出字符串的最小长度，如果长度大于或者等于指定的最小字符串，则返回原字符串
```javascript
'xxxxx'.padStart(3, 'ab') // xxxxx
'xxxxx'.padEnd(3, 'ab') // xxxxx
```
如果用来补全的字符串和原字符串的和大于指定的最小字符串，则截去超出位数的补全字符串
```javascript
'x'.padStart(6, 'ab') // ababax
'x'.padEnd(6, 'abcdef') // xabcde
```
第二参数用来指定补全的字符串，如果省略，则用空格补全
```javascript
'x'.padStart(3) // '  x'
'x'.padEnd(3) // 'x  '
```
padStart 方法常见的有两个用途
一个是用来为数值补全制定位数
```javascript
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"
```
另一个用途是用于提示字符串格式
```javascript
'11'.padStart('YYYY-MM-DD') // YYYY-MM-11
'10-11'.padStart('YYYY-MM-DD') // YYYY-10-11
```

## 10. 模版字符串
传统的方法，输出模版通常是这么写的
```javascript
$('#article').html(
  '<div>' + 
    '<h1>' + post.title + '</h1>' + 
    '<p>' + post.content + '</p>' +
  '</div>'
)
```
这种写法真的太不方便了，看起来又杂乱无比。ES6 引入**模版字符串**，这个真的很好用
```javascript
$('#article').html(`
  <div> 
    <h1>${post.title}</h1> 
    <p>${post.content}</p>
  </div>
`)
```
模版字符串其实就是增强版的字符串，用反引号（`）来标识
它可以当作普通字符串来使用
```javascript
`this is text line 1`
```
也可以定义多行字符串
```javascript
`this is text line 1
 this is text line 2`
```
最重要的是在字符串嵌入变量的方式
```javascript
const name = 'kobe' 
`my name is ${name}`
```
如果在模版字符串里想要使用反引号，则需要用反斜杠定义
```javascript
let str = `my name is \`kobe\``
```
模版字符串中的空格和换行都将保留，这里 <ul> 标签前有个换行，可以使用 trim 方法去掉
```javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim())
```
模版字符串变量名写在 ${} 之中
```javascript
const name = 'kobe' 
`my name is ${name}`
```
大括号内可以进行运算，可以引用对象属性
```javascript
let x = 1
let y = 2

`${x} + ${y} = ${x + y}` 

let obj = {x: 1, y: 2}
`${obj.x} + ${obj.y} * 2 = ${obj.x + obj.y * 2}`
```
模版字符串中还能调用函数
```javascript
function f() {
  return 'Hello World!'
}
`foo ${f()} bar`
```
如果大括号中的值不是字符串，将按照一般的规则，使用 toString 方法转换为字符串

模版字符串可以嵌套
```javascript
const tmpl = addrs => `
  <table>
    ${addrs.map(addr => `
      <tr><td>${addr.first}</td></tr>
      <tr><td>${addr.last}</td></tr>
    `).join('')}
  </table>
`
const data = [
  { first: '<Jane>', last: 'Bond' },
  { first: 'Lars', last: '<Croft>' },
]
tmpl(data)
// <table>
//   <tr><td><Jane></td></tr>
//   <tr><td>Bond</td></tr>

//   <tr><td>Lars</td></tr>
//   <tr><td><Croft></td></tr>

// </table>
```
如果需要引用模版字符串本身，在需要时执行，可以这样写
```javascript
// 写法一 
let str = 'return `Hello ${name}!`'
let f = new Function('name', str)

// 写法二 
let str = 'name => `Hello ${name}`'
let f = eval.call(null, str)
```

## 11. 实例：模版编译
这里将演示一个通过模板字符串，生成正式模板的实例
```javascript
let template = `
<ul>
  <% for (let i = 0; i < data.suppies.length; i++) { %>
    <li><%= data.suplies[i] %></li>
  <%} %>
</ul>
`
```
编译这个模板字符串的思路就是，将其先转换为如下的 JavaScript 表达式
```javascript
echo('<ul>')
for (let i = 0; i < data.suppies.length; i++>) {
  echo('<li>')
  echo(data.suplies[i])
  echo('<li>')
}
echo('<ul>')
```
转换方法使用正则替换
```javascript
let evalExpr = /<%=(.+?)%>/g
let expr = /<%([\s\S]+?)%>/g

template = template
  .replace(evalExpr, '`); \n echo( $1 ); \n echo(`')
  .replace(expr, '`); \n $1 \n echo(`')

template = 'echo(`' + template + '`);' 
```
然后，将 template 封装在函数里，然后返回这个函数即可
```javascript
let script = `
(function parse(data) {
  let output = ''

  function echo(html) {
    output += html
  }

  ${ template }

  return output
})
`
return script
```
将上面的内容拼装起来，就是一个模板编译函数了，这个函数命名为 compile
```javascript
function compile() {
  let evalExpr = /<%=(.+?)%>/g
  let expr = /<%([\s\S]+?)%>/g

  template = template
    .replace(evalExpr, '`); \n echo( $1 ); \n echo(`')
    .replace(expr, '`); \n $1 \n echo(`')

  template = 'echo(`' + template + '`);' 

  let script = `
  (function parse(data) {
    let output = ''

    function echo(html) {
      output += html
    }

    ${ template }

    return output
  })
  `
  return script
}
```
用法如下
```javascript
let parse = eval(compile(template))
parse({ supplies: [ "broom", "mop", "cleaner" ] })
//   <ul>
//     <li>broom</li>
//     <li>mop</li>
//     <li>cleaner</li>
//   </ul>
```

## 12. 标签模板
模版字符串还可以紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串，这被称为“标签模板”功能（tagged template）
```javascript
alert`123`
// 相当于
alert(123)
```
标签模板其实并不是模板，而是函数调用的一种特殊形式。“标签”指的是那个函数，紧跟在函数后面的模板字符串就是参数。

当模板字符串里有变量的时候，就不再只是简单的调用了，而是会将模板字符串处理成多个参数，然后在调用
```javascript
let a = 1
let b = 3

tag`Hello ${a}  World! ${a+b}`
// 等同于
tag(['Hello ', ' World', ''], 1, 4)
```
函数 tag 会接收到多个参数
```javascript
function tag(stringArr, value1, value2) {
  // ...
}

// 等同于

function tag(stringArr, ...value) {
  // ...
}
```
tag 函数的第一个参数是一个数组，成员是没有变量替换的部分，成员与成员之间是变量替换的部分

其它参数分别都是变量替换之后的值

所以， tag 函数所有参数实际的值如下
* 第一个参数： ['Hello ', ' World', '']
* 第一个参数： 1
* 第一个参数： 4

也就是说，tag 函数实际上以下面的形式调用
```javascript
tag(['Hello ', ' World', ''], 1, 4)
```
模板处理函数的第一个参数（模板字符串数组），还有一个 raw 属性
```javascript
console.log`123`;
// ['123', raw: Array[1]]
```
上面的代码中，console.log 实际接受的参数实际上是一个数组。该数组有一个 raw 属性，保存的是转义后的原字符串。

再看一个例子加深了解
```javascript
tag`First line\nSecond line`

function tag(strings) {
  console.log(strings.raw[0]);
  // strings.raw[0] 为 "First line\\nSecond line"
  // 打印输出 "First line\nSecond line"
}
```
上面代码中，tag 函数的第一个参数 strings，有一个 raw 属性，也指向一个数组。该数组的成员和 strings 数组是完全一致的。比如 strings 数组是["First line\nSecond line"]，那么 strings.raw 数组就是["First line\\nSecond line"]。两者唯一的区别是，后者的字符串里面的斜杠都被转义了。这个属性就是为了取得转义之前的原始模板而设计的。

下面是另一个复杂点的例子，将展示如何
```javascript
let total = 30
let msg = passthru`The total is ${total}(${total * 1.05} with tax)`

function passthru(literals) {
  let result = ''
  let i = 0

  while (i < literals.length) {
    result += literals[i++]
    if (i < arguments.length) {
      result += arguments[i]
    }
  }

  return result
}

msg // "The total is 30 (31.5 with tax)"
``` 
上面的例子采用 rest 参数的写法如下
```javascript
function passthru(literals, ...values) {
  let result = ''
  let index = 0

  for (index = 0; index < values.length; index++) {
    result += literals[index] + values[index]
  }

  result += literals[index]

  return result
}
```
标签模板的一个重要应用，就是过滤 HTML 字符串，防止用户输入恶意内容。这个例子中函数只过滤变量中的字符串，因为变量往往就是用户提供的。
```javascript
let message = SaferHTML`<p>${sender} has sent you a message.</p>`

function SaferHTML(templateData) {
  let s = templateData[0]

  for (let i = 1; i < arguments.length; i++) {
    let arg = String(arguments[i])

    s += arg
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')

    s += templateData[i]
  }
  return s
}

let sender = '<script>alert("abc")</script>'; // 恶意代码
let message = SaferHTML`<p>${sender} has sent you a message.</p>`;

message
// <p>&lt;script&gt;alert("abc")&lt;/script&gt; has sent you a message.</p>
```

## 13. String.raw()

ES6 为原生的 String 对象提供了一个 raw 方法

String.raw 往往作为模板字符串的处理函数，用于返回一个斜杠都被转义（即斜杠前再加一个斜杠）的字符串，对应于替换变量后的模板字符串
```javascript
String.raw`Hi\n${2+3}`
// "Hi\\n5!"

String.raw`Hi\u000A!`
// 'Hi\\u000A!'
```
如果原字符串的斜杠已经被转义，那么 String.raw 方法将不做任何处理
```javascript
String.raw`Hi\\n`
// "Hi\\n"
```
String.raw 也可以作为正常的函数使用。它的第一个参数应该有个 raw 属性，该属性的值应该是一个数组
```javascript
String.raw({ raw: 'test' }, 0, 1, 2);
// 't0e1s2t'

// 等同于
String.raw({ raw: ['t','e','s','t'] }, 0, 1, 2);
```