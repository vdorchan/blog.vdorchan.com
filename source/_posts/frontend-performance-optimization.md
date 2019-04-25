---
title: 前端性能优化与浏览器渲染
date: 2019-04-23 10:44:48
toc: true
tags:
  - 性能优化
  - 网站
  - 渲染
  - JavaScript
---

> 对性能优化的知识点做了些总结，如有纰漏，跪求批评指正。

在我们共同推动网页实现更多功能的过程中，将遇到一个常见的问题：性能。 如今，网站拥有比以往更多的功能，以至于许多网站都将精力用于在各种网络条件和设备上提供更高的性能。

不过，性能问题多种多样。轻微性能问题可能只会导致微弱的延迟，给您的用户带来短暂的不便。而严重的性能问题可能导致您的网站完全无法访问，无法对用户输入进行响应或两者同时发生。

## 内容压缩和优化

总体来说，我们要避免不必要的下载，首先要去评估每个资产的表现：其价值及其技术性能。然后根据这些资源是否提供了足够的价值来决定是否要移除它们。

比如一些 CSS 框架的开销可能导致渲染延迟严重，你可以视情况移除不必要的开销，以加速渲染。或者，移除不是必须的框架（使用更小的框架代替，例如使用 zepto 代替 jQuery，使用 Preact 代替 React）

而那些必要的资源，我们应该要对它进行压缩优化，根据资源(文本、图像、字体、源码等)的不同，我们使用不同技术压缩。

除了压缩，还可以对不同资源进行特定的优化：

* 图像优化
  * 选择合适的尺寸
  * 使用 CSS3 效果和网页字体代替图像
  * 由于人眼的工作方式的缘故，可以适当进行有损压缩
  * 假如浏览器支持，可以使用 WebP 和 JPEG XR 等压缩率更高的新格式
  * 使用 `<picture>` 和 `<img srcset>` 实现[响应式图片](https://developer.mozilla.org/zh-CN/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)使用 `<picture>` 和 `<img srcset>` 来完成。给 `img` 或设置了 `background` 的 CSS 属性的元素，将其设置为 `display: none`，并不能其阻止加载图片。
  * 使用视频代替 GIF， 当使用视频代替动画 GIF 时，可以减小数据量，并可能减少系统资源的使用。

* 脚本优化
  * 减少重排（reflow）和重绘（repaint）操作
  * 缓存 DOM 元素、DOM 列表长度 length、属性值
  * 使用事件委托，避免批量绑定事件
  * 尽量使用 ID 选择器，因为它一经找到就停止查找，而使用类选择器的话将遍历整个dom
  * 移动端使用 touch 事件代替 click 事件，因为 click 有 300ms 延迟
  * 使用[节流（throttle）和防抖（debounce）函数](https://CSS-tricks.com/debouncing-throttling-explained-examples/)减少性能消耗
  
* HTML优化
  * CSS 文件写在头部，JavaScript 放在尾部
  * 避免层级深嵌套
  * 避免img、iframe、a等元素的空src（会以当前网页为地址进行加载）
  * 避免行内样式和事件绑定
  * 大图片避免使用 base64，否则会是 html 文件勾搭，阻塞 HTML 解析

* CSS优化
  * 移除空的CSS规则
  * 正确使用 `display` 的属性，因为 `display` 属性会影响页面的渲染
  * 不滥用 `float`，因为它在渲染时计算量较大
  * 不声明过多的 `font-size`
  * 值为 0 时不要使用单位
  * 标准化各种浏览器前缀
  * 避免使用多层标签选择器。使用 class 选择器替换，减少 CSS 查找

## GZIP 压缩

Gzip 是一种用于文件压缩与解压缩的文件格式。它基于 Deflate 算法，可将文件（译者注：快速地、流式地）压缩地更小，从而实现更快的网络传输。 Web服务器与现代浏览器普遍地支持 Gzip，这意味着服务器可以在发送文件之前自动使用 Gzip 压缩文件，而浏览器可以在接收文件时自行解压缩文件。

它可以进一步提高压缩率，但它需要浏览器支持和服务器配置。通过 GZIP 压缩文本会的到很好的效果，但压缩图片时效果可能不是很明显。

## HTTP 缓存

通过网络提取内容既速度缓慢又开销巨大。较大的响应需要在客户端与服务器之间进行多次往返通信，这会延迟浏览器获得和处理内容的时间，还会增加访问者的流量费用。因此，缓存并重复利用之前获取的资源的能力成为性能优化的一个关键方面。

1. 通过 ETag 验证缓存的响应

   * 服务器使用 ETag HTTP 标头传递验证令牌。
   * 验证令牌可实现高效的资源更新检查：资源未发生变化时不会传送任何数据。

2. 通过 Cache-Control HTTP 标头定义其缓存策略

   Cache-Control 指令控制谁在什么条件下可以缓存响应以及可以缓存多久。

   > Cache-Control 标头是在 HTTP/1.1 规范中定义的，取代了之前用来定义响应缓存策略的标头（例如 Expires）。 所有现代浏览器都支持 Cache-Control，因此，使用它就够了。

在制定缓存策略时，您需要牢记下面这些技巧和方法：

* **使用一致的网址**：如果您在不同的网址上提供相同的内容，将会多次提取和存储这些内容。
  
* **确保服务器提供验证令牌 (ETag)**：有了验证令牌，当服务器上的资源未发生变化时，就不需要传送相同的字节。

* **确定中间缓存可以缓存哪些资源**：对所有用户的响应完全相同的资源非常适合由 CDN 以及其他中间缓存进行缓存。

* **为每个资源确定最佳缓存周期**：不同的资源可能有不同的更新要求。 为每个资源审核并确定合适的 max-age。

* **确定最适合您的网站的缓存层次结构**：您可以通过为 HTML 文档组合使用包含内容指纹的资源网址和短时间或 no-cache 周期，来控制客户端获取更新的速度。

* **最大限度减少搅动**：某些资源的更新比其他资源频繁。如果资源的特定部分（例如 JavaScript 函数或 CSS 样式集）会经常更新，可以考虑将其代码作为单独的文件提供。这样一来，每次提取更新时，其余内容（例如变化不是很频繁的内容库代码）可以* 从缓存提取，从而最大限度减少下载的内容大小。

## 利用 loaclStorage 缓存

可以将部分请求的数据和结果存放在 LocalStorage 中，实现缓存，这样可以省去发送http请求所消耗的时间，从而提高网页的响应速度。

使用这种方式需要设计好一套更新机制，在资源需要更新的时候，可以替换 LocalStorage 存储的内容。

## 使用内容发布网络（CDN）

内容分发网络（Content delivery network或Content distribution network，缩写：CDN）是指一种透过互联网互相连接的计算机网络系统，利用最靠近每位用户的服务器，更快、更可靠地将音乐、图片、影片、应用程序及其他文件发送给用户，来提供高性能、可扩展性及低成本的网络内容传递给用户。

更多的可参考[CDN是什么？使用CDN有什么优势？ - 知乎](https://www.zhihu.com/question/36514327?rf=37353035)

## 减少http请求

请求是比较耗性能的，因此减少http请求可以提高网站性能。

具体方式：

* 图片可以使用 CSS Sprite 或者 Base 64
* JavaScript 脚本文件和样式表可以将它们合并

但是如果你使用了 http2，那么你就不需要进行这些操作了，使用 HTTP2 可以在单个连接中进行多次请求。合并成一个大的文件可能反而使加载的时间更长了。

## 避免重定向

重定向会触发额外的 HTTP 请求-响应周期，并会拖慢网页呈现速度。在最好的情况下，每个重定向都会添加一次往返（HTTP 请求-响应）；而在最坏的情况下，除了额外的 HTTP 请求-响应周期外，它还可能会让更多次的往返执行 DNS 查找、TCP 握手和 TLS 协商。因此，您应尽可能减少对重定向的使用以提升网站性能。

## 预加载（资源提示）

我们使用**资源提示与指令**，比如 preload、 prefetch、Preconnect 来实现性能优化。

> Preload 与 prefetch 不同的地方就是它专注于当前的页面，并以高优先级加载资源，Prefetch 专注于下一个页面将要加载的资源并以低优先级加载。同时也要注意 preload 并不会阻塞 `window` 的 `onload` 事件。

**Preload** 这个指令可以在 `<link>` 中使用，比如 `<link rel="preload">`。一般来说，最好使用 `preload` 来加载你最重要的资源，比如图像，CSS，JavaScript 和字体文件。这不要与浏览器预加载混淆，浏览器预加载只预先加载在HTML中声明的资源。`preload` 指令事实上克服了这个限制并且允许预加载在 CSS 和JavaScript 中定义的资源，并允许决定何时应用每个资源。

**Prefetch** 是一个低优先级的资源提示，允许浏览器在后台（空闲时）获取将来可能用得到的资源，并且将他们存储在浏览器的缓存中。一旦一个页面加载完毕就会开始下载其他的资源，然后当用户点击了一个带有 prefetched 的连接，它将可以立刻从缓存中加载内容。有以下三种不同的 prefetch 的类型

* link：假设用户将请求它们，所以允许浏览器获取资源并将他们存储在缓存中。浏览器会寻找 HTML `<link>` 元素中的 prefetch 或者 HTTP 头中如下的 Link
* DNS：允许浏览器在用户浏览页面时在后台运行 DNS 的解析。如此一来，DNS 的解析在用户点击一个链接时已经完成，所以可以减少延迟。可以利用 Pagespeed 的过滤器 [insert_dns_prefetch](https://link.juejin.im/?target=https%3A%2F%2Fmodpagespeed.com%2Fdoc%2Ffilter-insert-dns-prefetch%23objective) 来自动化的为所有域名插入`<link rel=“dns-prefetch”>`
* prerendering：和 link prefetch 类似，区别是 prerendering 在后台渲染了**整个页面**，整个页面所有的资源

**Preconnect** 允许浏览器在一个 HTTP 请求正式发给服务器前预先执行一些操作，这包括 DNS 解析，TLS 协商，TCP 握手，这消除了往返延迟并为用户节省了时间。

上面所提到的预加载技术，dns prefetch 得到了比较好的支持，其它的兼容性都比较一般。

## HTTP/2

HTTP/2 的主要目标是通过支持完整的请求与响应复用来减少延迟，通过有效压缩 HTTP 标头字段将协议开销降至最低，同时增加对请求优先级和服务器推送的支持。 为达成这些目标，HTTP/2 还给我们带来了大量其他协议层面的辅助实现，例如新的流控制、错误处理和升级机制。上述几种机制虽然不是全部，但却是最重要的，每一位网络开发者都应该理解并在自己的应用中加以利用。

HTTP/2 没有改动 HTTP 的应用语义。 HTTP 方法、状态代码、URI 和标头字段等核心概念一如往常。 不过，HTTP/2 修改了数据格式化（分帧）以及在客户端与服务器间传输的方式。这两点统帅全局，通过新的分帧层向我们的应用隐藏了所有复杂性。 因此，所有现有的应用都可以不必修改而在新协议下运行。

[HTTP/2 简介](https://developers.google.com/web/fundamentals/performance/http2/)

**HTTP/2** 可解决 HTTP/1.1 的许多固有性能问题，例如并发请求限制和缺乏标头压缩。

在 HTTP/1 环境中，常见的做法是将样式和脚本捆绑成较大软件包。 这么做是因为减少请求数。使用 HTTP/2 后就不需要再这么做，因为同时发送多个请求的成本更低。

## 延迟加载图片和视频

延迟加载解决方案可以减少初始页面负载和加载时间，它是一种在加载页面时，延迟加载非关键资源的方法，而这些非关键资源则在需要时才进行加载。就图像而言，“非关键”通常是指“屏幕外”。

延迟加载图片可以先使用占位图，当滚动到视口时，占位图才替换为最终图像。比起监听 `scroll` 和 `resize`，现代浏览器更倾向于使用 [`Intersection Observer API`](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)，不过要注意处理兼容性。

```html
<img class="lazy" src="placeholder-image.jpg" data-src="image-to-lazy-load-1x.jpg" data-srcset="image-to-lazy-load-2x.jpg 2x, image-to-lazy-load-1x.jpg 1x" alt="I'm an image!">
```

我们监听 DOMContentLoaded 事件，当它被触发是，它会查询 DOM，以获取类属性为 `lazy` 的所有 `<img>` 元素。 如果 `Intersection Observer` 可用，我们会创建一个新的 Observer，以在 `img.lazy` 元素进入视口时运行回调。

```JavaScript
document.addEventListener("DOMContentLoaded", function() {
  var lazyImages = [].slice.call(document.querySelectorAll("img.lazy"));

  if ("IntersectionObserver" in window) {
    let lazyImageObserver = new IntersectionObserver(function(entries, observer) {
      entries.forEach(function(entry) {
        if (entry.isIntersecting) {
          let lazyImage = entry.target;
          lazyImage.src = lazyImage.dataset.src;
          lazyImage.srcset = lazyImage.dataset.srcset;
          lazyImage.classList.remove("lazy");
          lazyImageObserver.unobserve(lazyImage);
        }
      });
    });

    lazyImages.forEach(function(lazyImage) {
      lazyImageObserver.observe(lazyImage);
    });
  } else {
    // 假如不支持 IntersectionObserver，回退到更有兼容性的方法
  }
});
```

CSS 中的图像可以使用类来控制

```CSS
.lazy-background {
  background-image: url("hero-placeholder.jpg"); /* 占位图 */
}

.lazy-background.visible {
  background-image: url("hero.jpg"); /* 最终显示的图片 */
}
```

你还可以利用这些库实现延迟加载：[lazysizes](https://github.com/aFarkas/lazysizes)、 [lozad.js](https://github.com/ApoorvSaxena/lozad.js)、[blazy](https://github.com/dinbror/blazy)、[yall.js](https://github.com/malchata/yall.js)

视频可以通过指定 `preload="none"`来阻止浏览器预加载**任何**视频数据

## 优化关键渲染路径

**优化关键渲染路径**是指优先显示与当前用户操作有关的内容

**关键渲染路径（Critical Rendering Path）** 主要是指从收到 HTML、CSS 和 JavaScript 字节到对其进行必需的处理，从而将它们转变成渲染的像素这一过程中有一些中间步骤。

在拿到 HTML 和 CSS 文件后，浏览器会进行下面的工作。

![](https://raw.githubusercontent.com/vdorchan/blog.vdorchan.com/master/static/frontend-performance-optimization/browser-render.jpg)

1. 解析处理 `HTML` 标记，并构建 `DOM` 树

2. 解析处理 `CSS` 标记，并构建 `CSSOM` 树

3. 将 `DOM` 树和 `CSSOM` 树合并生成渲染树(`Render Tree`)，这个时候计算了哪些节点应该是可见的以及它们的计算样式。

   > 一些不可见的节点（例如脚本标记、元标记等）或者通过 CSS 隐藏（`display: none`）的节点，在渲染树中都会被忽略

4. 布局（Layout）：根据生成的渲染树，进行布局，以计算每个节点在设备视口的几何信息（位置，大小）

5. 绘制（Painting）：根据渲染树以及布局知道的信息（哪些节点可见、它们的计算样式和几何信息），将渲染树中的每个节点转换成屏幕上的实际像素

因此**优化关键渲染路径就是指最大限度缩短执行上文第 1 步至第 5 步耗费的总时间**。这样一来，就能尽快将内容渲染到屏幕上，此外还能缩短首次渲染后屏幕刷新的时间，即为交互式内容实现更高的刷新率。

**优化关键渲染路径的常规步骤如下：**

   1. 对关键路径进行分析和特性描述：资源数、字节数、长度。
   2. 最大限度减少关键资源的数量：删除它们，延迟它们的下载，将它们标记为异步等。
   3. 优化关键字节数以缩短下载时间（往返次数）。
   4. 优化其余关键资源的加载顺序：您需要尽早下载所有关键资产，以缩短关键路径长度。

## CSS 加载带来的阻塞

在渲染树构建中，我们看到关键渲染路径要求我们同时具有 DOM 和 CSSOM 才能构建渲染树。所以，**HTML 和 CSS 都是阻塞渲染的资源**。HTML 显然是必需的，因为如果没有 DOM，我们就没有可渲染的内容，但 CSS 的必要性可能就不太明显。

即使有了 DOM，在 CSSOM 处理完之前，浏览器是不会渲染任何已处理的内容的，这时候会有一段时间的**白屏**。也就是说**CSS 加载是会阻塞渲染的**，所以我们需要将它尽早、尽快地下载到客户端，以便缩短首次渲染的时间。

**FOUC(Flash of Unstyled Content)**

假如你不把 `<link>` 放在 `<head>` 标签内，而是放在其它位置（比如底部），当页面进行到这个 `<link>` 标签的时候，它会构建 CSSOM，并重新触发渲染树的构建，进而重新渲染页面内容，这时候页面可能会闪烁。

**CSS 加载同样会阻塞 JavaScript 的执行**

JavaScript 不仅可以读取和修改 DOM 属性，还可以读取和修改 CSSOM 属性，所以浏览器会延迟脚本执行和 DOM 构建，直至其完成 CSSOM 的下载和构建。

**JavaScript 执行和加载带来的阻塞**

浏览器是多线程的（JavaScript 是单线程的）。并且分别分配了渲染线程和 JavaScript 引擎线程，但由于加载执行和渲染DOM的并发可能会引发严重的冲突。

所以，**JavaScript引擎和渲染引擎所在的两个线程被设计为互斥的！**。

JavaScript 会阻塞 DOM 的构建和延缓网页渲染，当遇到 `<script>` 时，它会暂停构建 DOM，将控制权移交给 JavaScript 引擎；等 JavaScript 引擎运行完毕，浏览器会从中断的地方恢复 DOM 构建。

所以一般把 `<script>` 标签放到 `<body/>` 前，先让 DOM 构建完。

如果你不想它阻止 HTMl 的解析，那么你可以将脚本设置为异步的（defer 和 async），具体看下面内容

当浏览器碰到 `script` 脚本的时候

> 下面的图中
> 蓝色代表 **html 解析（html parsing）**
> 灰色代表 **html 解析暂停（html parsing paused）**
> 紫色代表**脚本下载（script download）**
> 红色代表**脚本执行（script execution）**

1. `<script src="script.js"></script>`
  没有 `defer` 或 `async`，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 `script` 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行，并且它的加载和执行都会阻塞后面的 DOM 解析。
  ![script](https://raw.githubusercontent.com/vdorchan/blog.vdorchan.com/master/static/frontend-performance-optimization/script.png
  S4X7MV2)

2. `<script async src="script.js"></script>`
  有 `async`，加载和渲染后续文档元素的过程将和 `script.js` 的加载与执行并行进行（异步）
  ![script-async](https://raw.githubusercontent.com/vdorchan/blog.vdorchan.com/master/static/frontend-performance-optimization/script-async.png)

3. `<script defer src="script.js"></script>`
  有 `defer`，加载后续文档元素的过程将和 `script.js` 的加载并行进行（异步），但是 `script.js` 的执行要在所有元素解析完成之后，`DOMContentLoaded` 事件触发之前完成。
  ![script-defer](https://raw.githubusercontent.com/vdorchan/blog.vdorchan.com/master/static/frontend-performance-optimization/script-defer.png)

如何选择使用它们

* 如果该脚本是模块化并且不依赖任何其它脚本，则使用 `async`
* 如果该脚本依赖或者被依赖于其它脚本，则使用 `defer`
* 如果脚本很小并且依赖于其它的异步脚本，则使用内联脚本，并且不加 `async` 或 `defer`。

**DomContentLoaded 和 onload：**

* 当初始的 HTML 文档被完全加载和解析完成之后，`DOMContentLoaded` 事件被触发，而无需等待样式表、图像和子框架的完成加载。注意：`DOMContentLoaded` 事件必须等待其所属script之前的样式表加载解析完成才会触发。
* load 应该仅用于检测一个完全加载的页面。当一个资源及其依赖资源已完成加载时，将触发load事件。

## 渲染优化

对于页面的渲染，这是你拥有的最大控制权的部分

![](https://raw.githubusercontent.com/vdorchan/blog.vdorchan.com/master/static/frontend-performance-optimization/frame-full.jpg)

* **JavaScript**。一般来说，我们会使用 JavaScript 来实现一些视觉变化的效果。对一个数据集进行排序或者往页面里添加一些 DOM 元素等。除了 JavaScript，`CSS Animations、Transitions` 和 `Web Animation API` 也可以实现视觉变化效果。
* **样式计算**。此过程是根据匹配选择器（例如`.headline或.nav > .nav__item`）计算出哪些元素应用哪些 CSS 规则的过程。从中知道规则之后，将应用规则并计算每个元素的最终样式。
* **布局**。在知道对一个元素应用哪些规则之后，浏览器即可开始计算它要占据的空间大小及其在屏幕的位置。网页的布局模式意味着一个元素可能影响其他元素，例如 `<body>` 元素的宽度一般会影响其子元素的宽度以及树中各处的节点，因此对于浏览器来说，布局过程是经常发生的。
* **绘制**。绘制是填充像素的过程。它涉及绘出文本、颜色、图像、边框和阴影，基本上包括元素的每个可视部分。绘制一般是在多个表面（通常称为层）上完成的。
* **合成**。由于页面的各部分可能被绘制到多层，由此它们需要按正确顺序绘制到屏幕上，以便正确渲染页面。对于与另一元素重叠的元素来说，这点特别重要，因为一个错误可能使一个元素错误地出现在另一个元素的上层。

1. JS / CSS > 样式 > 布局 > 绘制 > 合成
  如果您修改元素的“layout”属性，也就是改变了元素的几何属性（例如宽度、高度、左侧或顶部位置等），那么浏览器将必须检查所有其他元素，然后“自动重排”页面。任何受影响的部分都需要重新绘制，而且最终绘制的元素需进行合成。

2. JS / CSS > 样式 > 绘制 > 合成
   如果您修改“paint only”属性（例如背景图片、文字颜色或阴影等），即不会影响页面布局的属性，则浏览器会跳过布局，但仍将执行绘制。

3. JS / CSS > 样式 > 合成
   如果您更改一个既不要布局也不要绘制的属性，则浏览器将跳到只执行合成，这个版本性能最好，最适合动画或滚动。
   只有在使用 `transform` 和 `opacity` 两个属性，并使用 `will-change` 或 `translateZ` 提升该元素到新层。
   此方法的优点是，定期重绘的或通过变形在屏幕上移动的元素，可以在不影响其他元素的情况下进行处理。
   但需要注意的是：不要创建太多层，因为每层都需要内存和管理开销。

以下是一些优化渲染的手段

* JavaScript 经常会触发视觉变化。有时是直接通过样式操作，有时是会产生视觉变化的计算，所以可以通过优化 JavaScript 执行来提升性能

  * 对于动画效果的实现，避免使用 `setTimeout` 或 `setInterval，请使用` `requestAnimationFrame。`
  * 将长时间运行的 JavaScript 从主线程移到 Web Worker。
  * 使用微任务来执行对多个帧的 DOM 更改。

* 计算样式的第一部分是创建一组匹配选择器，这实质上是浏览器计算出给指定元素应用哪些类、伪选择器和 ID。第二部分涉及从匹配选择器中获取所有样式规则，并计算出此元素的最终样式。优化的关键是缩小样式计算的范围并降低其复杂性
  
  * 降低选择器的复杂性；使用以类为中心的方法，例如 BEM。
  * 减少必须计算其样式的元素数量。

* 布局是浏览器计算各元素几何信息的过程：元素的大小以及在页面中的位置。根据所用的 CSS、元素的内容或父级元素，每个元素都将有显式或隐含的大小信息。为提升技能，性能的开销主要来自于需要布局的元素数量和这些布局的复杂性。所以我们应避免大型、复杂的布局和布局抖动
  
  * 布局的作用范围一般为整个文档。
  * DOM 元素的数量将影响性能；应尽可能避免触发布局。
  * 新版 Flexbox 一般比旧版 Flexbox 或基于浮动的布局模型更快。
  * 先读取样式值，然后进行样式更改。假如反过来，那么浏览器必须先进性布局才能回到问题

* 绘制是填充像素的过程，像素最终合成到用户的屏幕上。它往往是管道中运行时间最长的任务，应尽可能避免此任务。所以我们要简化绘制的复杂度、减小绘制区域
  
  * 除 `transform` 或 `opacity` 属性之外，更改任何属性始终都会触发绘制。
  * 绘制通常是像素管道中开销最大的部分；应尽可能避免绘制。
  * 通过层的提升和动画的编排来减少绘制区域。

参考：
https://developers.google.com/web/fundamentals/performance/why-performance-matters/
https://developers.google.com/speed/docs/insights/about
https://juejin.im/post/5b5984b851882561da216311
https://segmentfault.com/a/1190000014518067