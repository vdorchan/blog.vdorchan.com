---
title: 理解 CSS 布局和 BFC
date: 2019-04-24 23:01:03
tags:
  - 学习
  - CSS
  - BFC
---

## 正常布局流 (normal flow)

正常布局流是指在不对页面进行任何布局控制时，浏览器默认的HTML布局方式。

在 normal flow 中，元素按照其在 HTML 源码中出现的先后位置至上而下布局。在这个过程中，行内元素水平排列，直到当行被占满然后换行。块级元素则会被渲染为完整的一个新行，除非另外指定，否则所有元素默认都是 normal flow 定位，也可以说，普通流中元素的位置由该元素在 HTML 源码中的位置决定。

以下这些布局技术可能会覆盖默认的流式布局

* __[display](https://developer.mozilla.org/en-US/docs/Web/CSS/display) 属性__： 像 block、inline 或者 inline-block 这样的标准值可以改变元素在 normal flow 中的行为。而使用像 CSS Grid 和 Flexbox 的值允许我们使用完全不同的方式来布局。
* __浮动（Floats）__： 应用 float 值，诸如 left 能够让块级元素互相并排成一行。
* __[position](https://developer.mozilla.org/en-US/docs/Web/CSS/position) 属性__： 正常布局流中，默认为 static ，可以使用其它值会来为元素使用不同的布局方案。
* __表格布局__：用于布置表格
* __[多列布局（Multi-column layout）](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Columns)__：可以使块的内容按列布局

## 脱离文档流

  > An element is called out of flow if it is floated, absolutely positioned, or is the root element. An element is called in-flow if it is not out-of-flow. The flow of an element A is the set consisting of A and all in-flow elements whose nearest out-of-flow ancestor is A.

根据 W3C 文档。我们可以简单理解为。__被float和被绝对定位的元素会脱离文档流__。除了这两种情况(其实还有fixed定位)以外的情况下，布局都是按照normal flow(文档流)的过程来布局的。

## BFC

**块格式化上下文（Block Formatting Context，BFC）**是Web页面的可视化CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。

按照 CSS 规范，下列方式会创建**块格式化上下文**：

* 根元素或包含根元素的元素
* 浮动元素（元素的 [float](https://developer.mozilla.org/zh-CN/docs/Web/CSS/float) 不是none）
* 绝对定位元素（元素的 [position](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position) 为absolute或fixed）
* 行内块元素（元素的 [display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为inline-block）
* 表格单元格（元素的 [display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为table-cell，HTML表格单元格默认为该值）
* 表格标题（元素的 [display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为table-caption，HTML表格标题默认为该值）
* 匿名表格单元格元素（元素的 [display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为table、table-row、table-row-group、table-header-group、table-footer-group（分别是HTML table、row、tbody、thead、tfoot的默认属性）或inline-table）
* [overflow](https://developer.mozilla.org/zh-CN/docs/Web/CSS/overflow) 值不为visible的块元素
* [display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 值为 [flow-root](https://drafts.csswg.org/css-display/#valdef-display-flow-root) 的元素
* [contain](https://developer.mozilla.org/zh-CN/docs/Web/CSS/contain) 值为layout、content或strict的元素
* 弹性元素（ [display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为flex或inline-flex元素的直接子元素）
* 网格元素（ [display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 为grid或inline-grid元素的直接子元素）
* 多列容器（元素的 [column-count](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-count) 或 [column-width](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-width) 不为auto，包括column-count为1）
* column-span为all的元素始终会创建一个新的BFC，即使该元素没有包裹在一个多列容器中（ [标准变更](https://github.com/w3c/csswg-drafts/commit/a8634b96900279916bd6c505fda88dda71d8ec51) ， [Chromebug](https://bugs.chromium.org/p/chromium/issues/detail?id=709362) ）。

## BFC的一些规则和作用

* BFC 内部的 Box 会在垂直方向一个接一个地放置，和普通流一样，一行一行的放置。

* Box垂直方向的距离由margin决定，属于同一个 BFC 下的两个相邻 Box 的 margin 会发生重叠。
可以通过让这两个 Box **分属于不同的BFC**时就可以**阻止margin重叠**

[去 codepen 编辑下面代码](https://codepen.io/vdorchan/pen/WWamBE)

```html
<div class="box"></div>
<!-- 使用一个wrapper包裹box，并触发这个wrapper的BFC，那么这个box就属于wrapper这个BFC下面了 -->
<div class="wrapper">
  <div class="box"></div>
</div>
<style>
  .box {
    display: block;
    width: 100px;
    height: 100px;
    background: lightblue;
    margin: 100px;
  }
  .wrapper {
    /* 使用 overflow: hidden; 触发BFC */
    overflow: hidden;
  }
  </style>
```

* 当一个 wrapper 包含两个浮动的 float-box 的时候，wrapper 将无法被撑开。这个时候就需要清除浮动了。触发这个 wrapper 的 BFC，使两个 float-box **处在同一个 BFC 区域内**，就能成功清除浮动了。

[去 codepen 编辑下面代码](https://codepen.io/vdorchan/pen/wZYZqp)

```html
<div class="wrapper">
  <div class="float-box"></div>
  <div class="float-box"></div>
</div>
```

```css
.float-box {
  float: left; /* 使其浮动 */
  width: 100px;
  height: 100px;
}
.wrapper {
  /* 触发BFC，任意一个都可以 */
  /* position: absolute; */
  /* overflow: hidden; */
  display: inline-block;
}
```

* 当一个 float-box 后面跟随一个 box 的时候，两者会重叠。这个时候只要给 box 触发 BFC ，两者就不会重叠了。结论：**BFC的区域不会与float box重叠**。这种用法，可以用来实现“固定 float-box宽度，box 根据包含块自适应宽度”的**自适应两栏**。

[去 codepen 编辑下面代码](https://codepen.io/vdorchan/pen/KYGEGX)

```html
<div class="float-box"></div>
<div class="box"></div>
```

```css
.float-box {
  float: left; /* 使其浮动 */
  width: 100px;
  height: 100px;
  background: lightpink;
}
.box {
  height: 200px;
  background: lightblue;
  overflow: hidden; /* 触发 BFC，将不会和 float-box 重叠 */ 
}
```

## Layout 和 hasLayout

“Layout”是一个 IE/Win 的私有概念，它决定了一个元素如何显示以及约束其包含的内容、如何与其他元素交互和建立联系、如何响应和传递应用程序事件/用户事件等。
当我们说一个元素“得到 layout”，或者说一个元素“拥有 layout” 的时候，我们的意思是指它的微软专有属性 hasLayout 为此被设为了 true 。

为了处理 IE 的兼容性，在需要触发 BFC 的时候，还需要针对 IE 浏览器通过一些动作将一个元素的 hasLayout 设置为 true。

下列 CSS 属性和取值将会让一个元素获得 layout：

* display: inline-block
* height: (除 auto 外任何值)
* width: (除 auto 外任何值)
* float: (left 或 right)
* position: absolute
* zoom: (除 normal 外任意值)
* writing-mode: tb-rl

## 文字环绕

预览下面的代码可以看到，虽然 box 被覆盖，但文字并没有被 float-box 覆盖，这是因为 float 当初设计的时候就是为了使文本围绕在浮动对象的周围。

```html
<div class="float-box"></div>
<div class="box">哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈
</div>
```

```css
.float-box {
  float: left;/* 使其浮动 */
  width: 100px;
  height: 100px;
  background: lightpink;
}
.box {
  width: 300px;
  height: 200px;
  background: lightblue;
}
```

参考：
[深入理解BFC - 小火柴的蓝色理想 - 博客园](http://www.cnblogs.com/xiaohuochai/p/5248536.html)
[块格式化上下文 - Web 开发者指南 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)