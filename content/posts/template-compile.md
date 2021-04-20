---
title: template-compile
date: 2017-10-12 10:59:42
tags: 
  - 学习
---
现在经常能看到“模板编译”、“模板引擎”的字眼。对这些概念并没有进行过深入的了解，基本处于貌似知道是什么，但又说不出来是什么的状态。带着一堆的疑惑，去看了几篇文章，也算是搞懂了一部分。

## 模板（Template）和模板编译
什么是模板呢？folderc 上面的解释是
> “一个包含了各种参数，并能够由模版处理系统通过识别某些特定语法来替换这些参数的文档。”

一个最基本的模板
```
My name is {{ name }}, I am {{ age }} years old.
```

上面所示的模板，它就包括了 name、age 参数，它将由**模板处理系统**通过识别某些特定的语法，用数据将 name、age 参数替换掉。

比如将模板中的 name、age 使用下面的数据对象给替换掉
```javascript
const data = {
  name: 'jack',
  age: 20
}
```

期待的结果应该是
```
My name is jack, I am 20 years old.
```

这个从模板到上面结果的之间的过程就称之为**模板编译**

我们将用正则替换来简单实现下这个过程
```javascript
const template = (tpl, data) => {
  let ret = tpl

  for (let item in data) {
    if (data.hasOwnProperty(item)) {
      const reg = new RegExp('{{' + item + '}}', 'g')
      ret = ret.replace(reg, data[item])
    }
  }
}

const tpl = 'My name is {{ name }}, I am {{ age }} years old.'

const data = {
  name: 'jack',
  age: 20
}

const result = template(tpl, data)
```

上面的实现方法效率是很差的，每个字段都要执行一次正则替换，当字段多的时候，那性能会是个大问题。

其实假设模板是这样子的，那就会方便多了，只要将数据传入函数，就能得到想要的结果了
```javascript
const tpl = (data) => {
  return 'My name is ' + data.name +', I am '+ data.age +' years old.'
}
```
为了得到这个函数，我们可以使用 new Function 构造函数
```javascript

```
