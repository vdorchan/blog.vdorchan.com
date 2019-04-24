---
title: 使用 vue 开发项目遇到的问题总结
date: 2019-04-19 11:14:24
tags:
  - Vue.js
  - 问题
---

开发的项目为：[https://github.com/vdorchan/vue-movie](https://github.com/vdorchan/vue-movie)，最初使用 vue-cli 2 作为脚手架工具，后又使用 vue-cli 3 重构。

开发过程还是遇到了一些问题，现在试着回想并记录下来。

## 1、请求接口跨域

接口跨域可以通过 webpack 配置 API 代理解决

webpack 是借助 [webpack-dev-server](https://github.com/webpack/webpack-dev-server) 插件提供开发服务器的，而 webpack-dev-server 使用 [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware) 实现跨域代理。

```javascript
const devWebpackConfig = merge(baseWebpackConfig, {
  // ...
  devServer: {
    // ...
    proxy: {
      '/api': {
        target: 'http://api.douban.com/v2', // 代理的API地址，就是需要跨域的API地址
        changeOrigin: true, // 代理的API地址如果是域名就要加这个
        pathRewrite: {
          '^/api': '',
        },
      }
    }
  }
})
```

上面代码涉及到的参数说明：

* target 为代理的API地址，就是需要跨域的API地址
* 代理的API地址如果是域名就要加多个参数 `changeOrigin: true`
* pathRewrite 是路径重写，也就是说会修改最终请求的API路径，原本访问的是 `http://api.douban.com/v2/api/xx`，上面代码重写路径后最终访问 `http://api.douban.com/v2/xx`

## 2、第二次进入页面不刷新

应用使用了 vue-router，为了避免每次路由变化的时候都重新渲染组件，便配合用上了 `keep-alive` 组件。

```html
<keep-alive>
  <router-view></router-view>
</keep-alive>
```

上面的 `router-view` 也是组件，用来渲染匹配路由的组件。

`keep-alive` 是抽象组件，自身不会渲染 DOM 元素，也不会出现在父组件链中。

加入上面代码之后，出现了下面两种情况

1. 当我第二次选中一部电影并进入电影详情页（/movie）的时候它还是显示第一次进入页面时的电影
2. 从当前电影详情页进入到另一部电影的详情页，页面依然没刷新

> 电影详情页会有一些相似电影推荐，通过它们也能进入电影详情页（/movie）

这是因为刷新数据的操作是放在 `created` 钩子上执行的，如下：

```javascript
created () {
  this.loadMovie()
}
```

后来查到有几个可用的钩子函数

* `[activated](https://cn.vuejs.org/v2/api/#activated)` 在 `keep-alive` 组件激活时调用，适用于情况 1，但上面的情况 2 不会调用
* `[beforeRouteEnter](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E8%B7%AF%E7%94%B1%E7%8B%AC%E4%BA%AB%E7%9A%84%E5%AE%88%E5%8D%AB)` 仅适用于情况 1
* `[beforeRouteUpdate](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E8%B7%AF%E7%94%B1%E7%8B%AC%E4%BA%AB%E7%9A%84%E5%AE%88%E5%8D%AB)` 近适用于情况 1
* 使用 `watch` 监听路由变化

我的项目最终选择了使用 `watch` 监听路由对象变化，1、2都能得到解决

```javascript
export default {
  created () {
    this.loadMovie()
    this.routeName = this.$route.name
  },
  watch: {
    '$route' (to, from) {
      if (to.name === this.routeName) {
        this.loadMovie()
      }
    }
  },
}
```

## 3、定时器在页面切换时没有及时销毁

因为 spa 在页面切换的时候并没有刷新页面，定时器也就不会被自动销毁。

通常情况，可以借助 `beforeDestroy` 钩子函数

```javascript
beforeDestroy () {
  clearInterval(this.timer)
}
```

但使用了 `keep-alive` 组件后，通过路由切换页面，组件并不会被销毁，那么这个函数也就不会被调用，这时可以使用 vue-router 提供的 `beforeRouteLeave`，同样可以在组件内定义

```javascript
beforeRouteLeave () {
  clearInterval(this.timer)
}
```