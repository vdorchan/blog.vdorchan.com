---
title: React项目优化的最佳实践
date: 2021-05-16 15:22:13
tags: 
  - 实践
  - React
  - Webpack
---

## 构建优化及开发体验
旧项目使用了 antd-design-pro，基于 umi 框架，umi 是一个整合了多个技术栈的框架，如状态管理、路由、webpack打包等。对于开发者来说，这些功能的初始化和具体配置被封装起来，开箱即用确实方便。但这么一个黑箱，除了通过官方提供的配置，我几乎无法有更进一步调整。

于是我移除了 umi，转而使用 webpack 5 从零搭起一个开发环境，过程花了不少时间去反复读文档，了解最佳实践，踩了不少坑。然而结果还是不错的，最新的技术可以带来不错的效益，实践过后也沉淀了一些东西。
### 依赖安装 - pnpm 与 yarn pnp
npm3 维护了一个扁平的依赖结构，虽然解决 npm2 的嵌套关系过深的问题，这也使得 `Node_modules`文件夹变得混乱。想必身为前端基本都苦 npm 久矣，每次安装，除了网络问题，数量繁多的小文件读写，也使得这个过程艰难险阻。

后来 Facebook 带来了 [Yarn](https://yarnpkg.com/)，它提供了一些更好的体验，比如 lock 文件、更好的缓存控制等。但其并没有改变巨大的`NODE_MODULES`目录。你的电脑依然要消耗空间去存储很多重复的模块。


在之后，yarn 推出了一个被称为即插即用（[pnp](https://yarnpkg.com/features/pnp/)）的功能，它直接移除了`NODE_MODULES`，而是使用 一个 pnp.js 的文件，将文件指向缓存中的映射。因为少了`NODE_MODULES`，在 webpack 5 之前，你就需要去配置 webpack 的 [resovle](https://webpack.docschina.org/configuration/resolve/) 模块的模块解析规则，或者使用[PnpWebpackPlugin](https://github.com/arcanis/pnp-webpack-plugin)插件。而 webpack 5 则是[原生支持](https://webpack.docschina.org/blog/2020-10-10-webpack-5-release/#resolving)，不需要额外的配置。


[pnpm](https://github.com/pnpm/pnpm) 同样是用来作为 npm 的替代，它不会去计算包的嵌套关系，并且通过硬连接将 `NODE_MODULES`内的文件指向机器内的一个全局存储目录，这样同一个模块只需要安装一次。[安装方式](https://pnpm.io/installation)很简单，比如通过脚本安装：


macOS, Linux 等系统
```bash
curl -f https://get.pnpm.io/v6.js | node - add --global pnpm
```


Windows (使用 PowerShell):
```
(Invoke-WebRequest 'https://get.pnpm.io/v6.js' -UseBasicParsing).Content | node - add --global pnpm
```


安装后，就像`npm install`一样，使用`pnpm install`即可。


下图是安装速度对比的benchmark。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/376315/1619342075007-c3c81bd7-ca87-4ace-b46d-02e7cdb80548.png#align=left&display=inline&height=506&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1012&originWidth=945&size=79658&status=done&style=none&width=472.5)
### Webpack 5
[Webpack 5](https://webpack.docschina.org/blog/2020-10-10-webpack-5-release/) 发布了很多用于改进编译性能的新特性，以下是一些简单的介绍，后面会针对 webpack5 + react + ts 配置肝一篇文章出来。
#### 持久缓存
Webpack5之前在构建时，会以配置的 entry 为入口，递归解析模块依赖，构建出一个依赖图（graph），该依赖图记录代码中各个 module 之间的关系。

每当有文件内容更新的时候,会重新递归生成依赖图，如果简单粗暴地重建依赖图再编译，会有很大的性能开销。在webpack5中，利用缓存实现增量编译，从而提升构建性能。每当代码变化、模块之间依赖关系改变导致依赖图改变时， Webpack 会读取记录做增量编译。

在之前，我们通常会通过`cache-loader`将编译结果写入缓存，Webpack再次构建时如果文件没有发生变化，则会直接拉取缓存。还有像 babel-loader 这样的 loader 自带了缓存配置。


webpack 5 缓存默认是 memory，设置缓存类型为文件系统缓存，将缓存写如本地目录（默认为`node_modules/.cache/webpack`）。


下面截图分别是第一次编译，以及无修改后的第二次编译。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/376315/1619343092384-70c56d0d-73ce-450d-8b77-6620413c6b93.png#align=left&display=inline&height=205&margin=%5Bobject%20Object%5D&name=image.png&originHeight=410&originWidth=770&size=56837&status=done&style=none&width=385)![image.png](https://cdn.nlark.com/yuque/0/2021/png/376315/1619343123069-ab1eeaa9-457d-4ef8-9b0e-8db29bdb9aa2.png#align=left&display=inline&height=209&margin=%5Bobject%20Object%5D&name=image.png&originHeight=418&originWidth=796&size=57932&status=done&style=none&width=398)
增量编译在代码量大，模块多的情况下，会有更大的优势。它会让 cpu 和内存的使用率大大降低。
#### 资源模块 asset/resource
在 webpack 5 之前，我们需要借助`[raw-loader](https://webpack.docschina.org/loaders/raw-loader/)`、`[url-loader](https://webpack.docschina.org/loaders/url-loader/)`、[`file-loader`](https://webpack.docschina.org/loaders/file-loader/)来允许我们使用字体、图标等资源文件。


比如使用`file-loader`将图片文件发送到输出目录：
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/i,
        use: [
          {
            loader: 'file-loader',
          },
        ],
      },
    ],
  },
};
```
#### 
而 webpack 5 则是原生提供了[资源模块](https://webpack.docschina.org/guides/asset-modules/)(asset module)，无需在额外配置 loader。


资源模块类型(asset module type)，通过添加 4 种新的模块类型，来替换所有这些 loader：

- `asset/resource` 发送一个单独的文件并导出 URL。之前通过使用 `file-loader` 实现。
- `asset/inline` 导出一个资源的 data URI。之前通过使用 `url-loader` 实现。
- `asset/source` 导出资源的源代码。之前通过使用 `raw-loader` 实现。
- `asset` 在导出一个 data URI 和发送一个单独的文件之间自动选择。之前通过使用 `url-loader`，并且配置资源体积限制实现。



下面代码我们使用了`asset`类型的资源模块，当命中的资源小于 4k 的时候，资源模块将其导出为一个 data URI。反之，则是发送一个一个单独的文件。
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: [/\.bmp$/, /\.gif$/, /\.jpe?g$/, /\.png$/],
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 4 * 1024, // 4kb
          },
        },
        generator: {
          filename: 'static/assets/[hash][ext][query]',
        },
      },
    ]
  }
}
```
#### 一些坑一个坑 - 热刷新失效
在使用 webpack 5 的时候，也遇到了一些坑。还好也都一一解决了。
#####  热刷新失效
在使用 `webpack-dev-server` 热更新的功能时候，代码正常编译，但页面并未刷新。后来找到对应的 issue。[https://github.com/webpack/webpack-dev-server/issues/2758](https://github.com/webpack/webpack-dev-server/issues/2758)。这是 `webpack-dev-server`在我们使用 webpack 5 + browserlist 时出现的一个 bug。官方后面会更新修复，在这之前，我们可以使用`target: web`来临时解决这个问题。
##### less-loader 警告语句
在使用 `less-loader`，会报`less.webpackLoaderContext deprecated`这样的警告语句，同样在官方 issue [https://github.com/webpack-contrib/less-loader/issues/413](https://github.com/webpack-contrib/less-loader/issues/413) 发现错误。官方已在版本 8.1.1 修复错误。
#### 编译提速 - thread loader
在 webpack，有些处理可能会相当耗时，这时我们其实可以利用计算机多核的优势，将一些任务放在单独的线程并行去处理任务。虽然 happypack 流行过一阵，但作者逐渐失去兴趣，已经停止维护，并且向我们推荐了[thread-loader](https://webpack.docschina.org/loaders/thread-loader/#root)。


thread-loader 是官方维护的 loader。该 loader 可以将一些 loader 放在独立的 worker 池中运行。但每个 worker 都是一个独立 node.js，这同样会有一定的开销。所以只在耗时的操作使用它，否则可能会导致编译的速度更慢。


下面代码我们将使用 `thread-loader`来开启单独的进程处理babel-loader的任务。可以通过`wamup`来防止启动 worker 时的高延时。
```javascript
const threadLoader = require('thread-loader')
const workerPoolBabel = {
  workers: +webpackEnv.threads, // 产生的 worker 的数量
  workerParallelJobs: 2, // 一个 worker 进程中并行执行工作的数量
  poolTimeout: webpackEnv.watch ? Infinity : 2000,
};

threadLoader.warmup(workerPoolBabel, ['babel-loader']);

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: 'thread-loader',
            options: workerPoolBabel,
          },
          {
            loader: require.resolve('babel-loader'), 
          }
        ]
      }
    ]
  }
}
```


### React 开发体验 - React Fast refresh
React Fast Refresh 是 React 官方为 React Native 开发的**模块热替换（HMR）方案**，由于其核心实现与平台无关，所以官方将它作为纯用户解决方案，web 也能使用。同时 react-hot-loader 也随之被取代。

React Fast Refresh 具有更低的侵入性，它不需要在代码中加入`hot(App)`，同时官方支持的光环之外，还带来性能与稳定性保障。使用过程中，除了提供编辑后的及时反馈。还有个好处就是，对 hook 更完善的支持，在刷新的时候，组件状态可以得以保留。

借助 webpack 配合[react-refresh-webpack-plugin](https://github.com/pmmmwh/react-refresh-webpack-plugin)，我们可以很容易用上它。


使用 pnpm 安装所需依赖
```bash
pnpm add -D @pmmmwh/react-refresh-webpack-plugin react-refresh
```
编写配置
```javascript
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');

const isDevelopment = process.env.NODE_ENV !== 'production';

module.exports = (webpackEnv, argv) => {
  return {
    mode: isDevelopment ? 'development' : 'production',
    module: {
      rules: [
        {
          test: /\.js$/,
          use: [
            {
              loader: require.resolve('babel-loader'),
              exclude: /node_modules/,
              options: {
                plugins: [
                  isDevelopment && require.resolve('react-refresh/babel'),
                ].filter(Boolean),
              },
            },
          ],
        },
      ],
    },
    plugins: [
      isDevelopment && new ReactRefreshWebpackPlugin(),
    ].filter(Boolean),
  };
};
```
然后通过`webpack-dev-server`启动，hot选项是必须的。
```bash
webpack-dev-server --hot
```
#### 总结
除了上面介绍的，webpack 配置还包括：

- 使用 TypeScript 开发时，类型检查会占用很大的机器性能，使用 [fork-ts-checker](https://github.com/TypeStrong/fork-ts-checker-webpack-plugin) 可以在一个单独的进程上运行类型检查器。
- 资源分割
### 无限滚动优化
这次负责项目的页面，有一个无限滚动的长列表。
> 无限滚动的作用是骗过用户，让用户在不断往下滚动的时候，新内容同时也不断出现。对于用户来说，这会带来不错的体验和吸引力。



最简单朴素的滚动加载，则是监听`onscroll`，通过滚动高度和内容高度差，去发现用户滚动到底部，然后触发加载，我们可以加个 loading 的字样或 icon 起到提示作用。


项目一开始使用[react-infinite-scroller](https://github.com/danbovey/react-infinite-scroller)来简单实现了一个无限滚动的效果，效果其实类似官方[demo](https://danbovey.uk/react-infinite-scroller/demo/)。


在开发这种类型的页面的时候，可以使用一些提升用户提升的手段。比如保持导航栏可见、置顶按钮、墓碑（Tombstones）。其中，使用虚拟滚动可以对性能有比较大的提升。
#### 墓碑（Tombstones）
当出现网络延迟或者接口读取慢的情况，用户飞快的滚动页面，可以很轻松的到达最后一个元素。这时候使用一些占位符，直到接口返回数据，再用实际的内容替换。这样的过渡会相对和谐很多，而不至于导致用户失去对页面的关注。

下面截图是 facebook 滚动加载时的截图。
### ![image.png](https://cdn.nlark.com/yuque/0/2021/png/376315/1619366383189-850e4285-d64a-4760-b8b3-735ea47fd2a3.png#align=left&display=inline&height=606&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1212&originWidth=1110&size=57971&status=done&style=none&width=555)
#### 虚拟滚动
滚动加载是内容是前后端共同优化的一种方式，一次不会渲染太多的内容而导致页面卡顿。但如果有个长列表需要一次渲染很多条数据，比如1万条，如果前端一次性把它渲染出来，渲染会阻塞主线程，导致页面卡顿，无法响应用户的行为。


创建 DOM 并添加到页面是很昂贵的操作，这些 DOM 节点会不断的增加内存、布局、样式的成本，进而影响性能。一种可行的优化方式就是只渲染可视区域内的。

- 滚动容器元素：一般情况下，滚动容器元素是 window 对象。然而，我们可以通过布局的方式，在某个页面中任意指定一个或者多个滚动容器元素。只要某个元素能在内部产生横向或者纵向的滚动，那这个元素就是滚动容器元素考虑每个列表项只是渲染一些纯文本。在本文中，只讨论元素的纵向滚动。
- 可滚动区域：滚动容器元素的内部内容区域。假设有 100 条数据，每个列表项的高度是 50，那么可滚动的区域的高度就是 100 * 50。可滚动区域当前的具体高度值一般可以通过(滚动容器)元素的 scrollHeight 属性获取。用户可以通过滚动来改变列表在可视区域的显示部分。
- 可视区域：滚动容器元素的视觉可见区域。如果容器元素是 window 对象，可视区域就是浏览器的视口大小(即视觉视口)；如果容器元素是某个 div 元素，其高度是 300，右侧有纵向滚动条可以滚动，那么视觉可见的区域就是可视区域。



以下是 [https://github.com/bvaughn/react-window](https://github.com/bvaughn/react-window) 的例子，我们可以看到 DOM 被重复利用，并使用绝对移动模拟滚动的效果。
![1.gif](https://cdn.nlark.com/yuque/0/2021/gif/376315/1620661605336-2ba8e9f1-216e-4aa9-b5be-ebf85eea94c0.gif#align=left&display=inline&height=499&margin=%5Bobject%20Object%5D&name=1.gif&originHeight=499&originWidth=800&size=3494664&status=done&style=none&width=800)


## 使用 Worker 优化项目
上一节我们讲到渲染会阻断主线程，同样的，当使用 JavaScript 进行大量复杂运算时也会独占主线程，其它页面的事件将无法得到及时的响应，造成页面假死的现象。虽然 JavaScript 为了在浏览器能准确运行而被设计成单线程，但计算机是多线程的，[Web Workers API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API) 给了我们在一个独立线程运行代码的能力，这样就不会影响主线程的进行。


我们依然不能在 worker 线程中操纵 DOM 元素，或使用`window`对象中的某些方法和属性。通过`postMessage()`，我们可以在主线程和 worker 线程之间传输信息。


项目内有个需求是导出大数量数据的 excel，使用 `Web Worker` 保证了导出时的界面的正常运行。具体的效果可以看这个 demo（TODO）。


除了 `Web Worker`，我们还可以使用 [Service Worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers) 来实现离线应用和优化缓存。
## React 最佳实践
### 函数式组件和 hooks
这次项目，函数式组件得到更广泛的使用，在 hooks 加持之下，函数式组件功能在不断丰富。虽然目前用 Class Component 还是 Function Component 在社区还存在争论。Hooks 拥抱了函数，实际上更加符合声明式和函数式的概念，和 React 组件数据到视图映射的函数 UI = F（data）也更加匹配。
两种组件形式在性能并没有哪个比另外一个拥有很大的优势，hooks 涉及的闭包，对比类的原始性能在大多数情况下的差异基本可以忽略不计。与之相对，我们更应该清楚，两者存在截然不同的心智模型。

之前我们在使用类组件的时候，因为可变（immutable）的`this`，让我们能随时都能访问到最新的状态（`props` 和 `state`）。而函数式组件则是捕获了每一次渲染的状态，不同的渲染帧之间，具有各自独立的状态。

这种不同可以用[一个例子](https://codesandbox.io/s/pjqnl16lm7)来说明，先打开它，尝试点击按钮并观察现象。


使用这个函数式组件，当你点击按钮之后的 3 秒内，修改`user`，你会发现弹窗上显示的依然是你点击按钮时的那个值。我们可以明确的是，当我们点击按钮的时候，组件的`props.user`变量捕获了当点击事件被触发的那一次渲染。所以当你点击的时候，弹出的内容就是那一刻的`props.user`变量。


```javascript
function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };
  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };
  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```


### 使用 hooks

- 状态逻辑复用

在使用 Class Component 的时候，通常是利用 HOC（高阶组件）或 render props 的方案来实现一部分逻辑服用，它通常需要我们重新组织组件结构，同时会存在过多的嵌套抽象层组件从而导致“嵌套地狱”。而 hooks 让状态逻辑复用变得更加简单。


比如在项目中，我们有非常多的页面需要请求接口，为了良好的用户体验，我们通常需要一个 loading 的效果，与此之外，我们还需要在接口请求失败的时候在页面上告知用户，于是我们在项目的不同页面写下这些代码。


```jsx
function Page() {
	const [loading, setLoading] = useState(false);
	const [errorMsg, setErrorMsg] = useState('');
	const [data, setData] = useState('');
  
  useEffect(() => {
  	fetchData()
      .then(res => setData(res.data))
    	.catch(err => setErrorMsg(err.message))
  }, [])
  
  if (loading) return <div>loading...</div>
  if (errMsg) return <div>error: {errMsg}</div>
  
  return <div>{data}</div>
}
```
上面的代码，我们在不同的页面都要重新去写，我们完全可以使用 hooks 将这些逻辑封装起来，实现一个 `useRequest`。
```jsx
function useRequest(service) {
  const [loading, setLoading] = useState(false);
	const [errorMsg, setErrorMsg] = useState('');
	const [data, setData] = useState('');
  
  useEffect(() => {
  	service()
      .then(res => setData(res.data))
    	.catch(err => setErrorMsg(err.message))
  }, [])
  
  return {
  	data,
    loading,
    errMsg
  }
}
```
然后，在需要使用的页面无需改动界面代码直接引入使用即可。
```jsx
import useRequest from 'useRequst'

function Page() {
	const { data, loading, errMsg }	= useRequst(fetchData);
  
  if (loading) return <div>loading...</div>
  if (errMsg) return <div>error: {errMsg}</div>
  
  return <div>{data}</div>
}
```


在项目中合理抽象出可复用的逻辑，可以减少重复写一样的代码，并可以集中维护相关逻辑。上面的例子只是简单示范，实际项目可以使用阿里开源的 [ahooks](https://ahooks.js.org/) 库实现的 [useRequest](https://ahooks.js.org/hooks/async)。
### 恰当的组件设计
说到组件设计，我们通常会说要遵循单一职责化，拆分成很多个可复用组件，这样子的目的在实际复杂业务的项目中并不容易达到。在划分组件的过程，实际上是会有不断的调整过程，因为业务会变，它不具备规律性。


根据划分的维度不同，组件通常有木偶组件（Dumb Component）、智能组件（Smart Component）、业务组件、路由组件这些。


现在的中后台项目通常都会使用 ant design 这类 UI 库，这些库提供的 ui 组件是我们项目中用到的粒度最小的组件，它完全和业务无关。我们经常也需要自己去实现一些 UI 组件，设计这类组件对可复用的要求最高，要具备比较高的通用性，在设计 `props`要尽量严谨规范。


顶层组件通常是按照路由来划分的，这些组件是不能复用的，它通常包含了比较复杂的业务逻辑。其实，除了顶层组件，我们在划分组件的时候，不止是因为可复用，它可以给我们带来的另一个好处是**分治**，如果你一个页面写了上千行的代码，维护起来会相当困难。但是通过拆分组件就可以很容易定边界，不仅结构更加清晰，同时利于排查错误。


假如我们现在有两个类似的列表页面，它们都由筛选表单+列表组成。那么我们立刻想到的是不是，为这两个类似的页面组件去设计一个可复用的组件`List`，然后分别传入`listA`和`listB`。但实际情况是，两个页面类似，但实际上两个页面有不同的业务处理规则，当然我们可以选择在父组件处理好业务规则再往`List`传数据（组件之间过多的`props`传递也会降低维护性），但两个页面的列表部分也存在一些专属的业务逻辑，我们免不了要在 `List` 组件写各种判断，导致组件逻辑变得混乱。当业务需求有改动时，扩展会变得很困难。

上面说的这种情况，更合适的做法是，首先部分抽象，比如这两页面的列表都使用表格展示，它们都有无限滚动功能和排序功能，那么我们就可以抽象出`Table`、`InfiniteLoad`、`Sorter`组件。然后分别开发两个业务模型下的列表组件`ListA`和`ListB`组件，业务逻辑则分别在两个组件下单独维护。


我们可以看到，组件的粒度控制是十分重要的，粒度太粗可能会存在太多的重复代码，粒度太细则会影响后续可扩展性。大多数情况下，我们还是要根据实际的业务情况来评估，然后进行一定的耦合度标准的取舍。


#### 总结
一般项目可以通过这些规则来划分组件：

- 路由划分顶层组件。
- 为顶层组件合理划分业务子组件实现分治。
   - 当涉及到不同子组件共同的业务逻辑，可以写在父组件，通过传递`props`来协调各个子组件。
   - 大多数时候，业务逻辑直接写在子组件。
   - 注意抽象出一些可以解偶业务的可复用组件
- UI 组件着重可复用和可靠性，`props`设计要规范。
- 组件要将信息隐藏，封装在组件内。在使用这些组件的时候，其它组件不需要知道或依赖组件的内部结构和细节。
- 业务组件的命名应该尽量详细有意义，冗长也比信息表达不清晰要强。



### 注意性能问题
当项目复杂到一定程度（在项目刚开始的时候，我们切勿想太多，过早优化容易让你寸步难行），我们就要开始留意性能问题了。在优化之前，我们首先需要进行分析和找出问题，React 官方提供了 chrome 扩展用于发现项目中的渲染问题，之后我们就可以进行有针对优化。除此之外，我们可以借助 chrome 自带的 [performance](https://developer.chrome.com/docs/devtools/evaluate-performance/) 模块帮助分析。


react 使用了一个启发式算法来进行 diff 操作，当某个组件节点的 props 和 state 改变时，这个组件下的所有节点将会直接重新渲染。这样子会产生什么问题呢，前面我们谈到组件化的时候讲到复用和分治，假如你将代码都写在一个组件还会有性能问题，因为任何一个状态改变都会使这整个庞大的组件重新渲染。


#### React.memo()
将一个大的组件分成多个小的组件之后，我们要让这些组件避免在不必要的时候更新，我们要把组件 memorize 起来。

React 提供了 `React.memo`，通过 HOC 的方式，在需要减少渲染的组件外包裹一层`React.memo`。这可以让组件记住原本的 props，然后对 props 进行浅比较， 只在其变化的时候重新渲染。


这个例子，父组件存储了一个 `msg` 变量，并且监听输入框改变变量，以及使用一个组件 `ExpensiveComponent` 的两个不同版本，数字是 `ExpensiveComponent` 重新渲染的次数。
```jsx
const ExpensiveComponent = React.memo(() => {
	// ...
})
```
#### React.useMemo() 与 React.useCallback()
还是上面的例子，我们将一个需要经过复杂过滤函数 `complicateFilter` 得到的 `filteredList` 传给 `ExpensiveComponent` 组件。通常情况下，父组件每次渲染，都要重新执行`complicateFilter`，并把重新获得的列表传给组件。这个时候 `ExpensiveComponent` 检测到有 `prop` 的内存地址变了，便会重新渲染。


所以memo通常要和 `React.useMemo` 配合使用，`React.useMemo`可以将计算的结果缓存起来，避免重复计算新的结果。
```jsx
const memoizedFilteredList = useMemo(() => complicateFilter(list), [list]);
```


除了`useCallMemo()`，React 还提供了`useCallback` 用于将函数缓存。
```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```
#### 它不是万能的
这些方法只是 React 提供的一些用于性能优化的小窍门，它们并不能在状态变化的时候去阻止渲染。并且，缓存组件或对象是会需要额外的成本的。比如一个`prop`变化频繁的组件，因为我们可以预料到，在每次的`prop`比较它总会返回`false`，然后组件重新渲染。


所以，`memorization`通常适用于这些情况

1. 纯函数组件，相同的`prop`，总是输出一样的渲染。
1. `prop`相同，但重新渲染很频繁的组件。
1. 有意外渲染情况的中大型组件



#### 总结
在进行性能优化的时候，我们要不能忘记下面这些思考：

1. 任何优化都会增加复杂性，任何过早添加的优化都是有风险的，因为优化的代码可能会多次更改。
1. 先有分析测量和找出问题，在根据问题去确定优化方案。
1. 比起使用`React.memo()`等 api 增加的复杂度，增加的性能是否值得。
### 总结
篇幅有限，本文只是相对概括但不算仔细的概括这一次实践。其实写下来，总感觉还有很多东西没有说。比如像 React 的 `context` 和 `reducer` 能否代替全局状态管理器这种问题，在做项目的过程是觉得它非常值得拿出来讨论的，但讲起来会很复杂，为了避免文章过于冗长，也就只能放弃在这篇文章讲了。


经过一轮折腾，代码的质量，以及产品的性能，是有得到一定的提升的。比如首屏文件加载大小减少了三成，一定幅度缩短了白屏的时间。


相比于后端去考虑高性能、可扩展之类的。前端通常更多考虑的是高内聚低耦合的分层设计。现在的前端项目越来越庞大，架构的设计会很大程度影响项目的质量。良好的设计可以降低开发人员的心智负担，让开发人员维护起来更舒服，这样是可以带来很大的开发效率提升的。因此作为前端，应该时刻关注这些问题，及时总结。


参考：
[https://developers.google.com/web/updates/2016/07/infinite-scroller](https://developers.google.com/web/updates/2016/07/infinite-scroller)
[https://uxplanet.org/infinite-scrolling-best-practices-c7f24c9af1d#.6vfij8d11](https://uxplanet.org/infinite-scrolling-best-practices-c7f24c9af1d#.6vfij8d11)
[https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/](https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/)
[https://cloud.tencent.com/developer/article/1504653](https://cloud.tencent.com/developer/article/1504653)
[https://developer.chrome.com/docs/devtools/evaluate-performance/](https://developer.chrome.com/docs/devtools/evaluate-performance/)
[https://kentcdodds.com/blog/usememo-and-usecallback/](https://kentcdodds.com/blog/usememo-and-usecallback/)


## 
