---
title: browser-render
date: 2019-04-22 22:26:07
tags:
---

我们都知道

> JavaScript 是单线程的

既然 JavaScript 是单线程的，那么 setTimeout、setInterval 是如何执行的呢？

这些异步能执行的原因是，浏览器另开了一个线程用于管理它们

> 浏览器是多进程，并且是多线程的

其中，**浏览器渲染进程**是最重要的，是前端开发最需要密切关注的。

**浏览器渲染进程**是多线程的，包括

* **GUI渲染线程**：负责渲染浏览器界面，解析 `HTML` 生成 `DOM`，解析 `CSS` 生成 `CSSOM` 树，合并构建 `RenderObject` 树，布局和绘制等
* **JavaScript 引擎线程**：也称为JS内核，负责处理Javascript脚本程序
* **事件触发线程**：归属于浏览器而不是JS引擎，用来控制事件循环
* **定时触发器线程**：`setInterval` 与 `setTimeout` 所在线程
* **异步http请求线程**： 在 `XMLHttpRequest` 在连接后是通过浏览器新开一个线程请求

## 执行脚本带来的阻塞

虽然**GUI渲染线程**和**JavaScript 引擎线程**分属于不同的线程

但由于

> 脚本执行和渲染DOM的并发可能会引发严重的冲突

所以

> JavaScript引擎和渲染引擎所在的两个线程被设计为互斥的！

实际上 JavaScript 的执行是会阻塞 DOM 的解析和渲染的。