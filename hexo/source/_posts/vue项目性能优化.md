---
title: vue项目性能优化
date: 2018-02-04 20:31:11
tags: [vue]
---

这篇文章主要是对之前项目的一点总结，提出一些在开发中可以进行性能优化的点，针对vue-cli初始化或webpack打包的项目。如果需要了解web性能方面的知识，请参考[web性能](https://maizsss.github.io/2017/12/12/Web%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/)。

### 代码分割
默认的情况下，webpack会把你应用的代码打包到一个bundle中，如果你的项目有很多的页面，这样打包出来的文件会非常大，导致加载慢。最好的方式是按照页面划分，每个页面单独分割出一个文件，然后按需加载。比如，你的页面中有一个首页和一个用户信息页，那么可以把它们分成两个文件index.vue和info.vue。当然，如果首页还是很大，还可以按照页面组件来划分，比如menu.vue、tab.vue、modal.vue等。

### 组件动态加载
通过上面的分割代码之后，bundle被分成了很多小部分，这样我们可以动态加载需要的部分。来看看是怎么实现的？

利用import()作为动态模块加载器。在webpack中已经把import()作为一个代码分割点，它把import来的模块打包成一个单独的文件，并在需要的时候去加载。import接收一个文件路径作为参数，然后返回一个Promise。这里有一个异步单文件组件：

AsyncComponent.vue

```
<template>
  <div>Async Component</div>
</template>
<script>
  export default { }
</script>
```
使用import就像这样如此简单整洁的完成异步加载：

```
new Vue({ 
  el: '#app',
  components: {
    AsyncComponent: () => import('./AsyncComponent.vue')
  }
});
```
### vue-router路由懒加载
在vue的项目中，经常会使用vue-router去管理SPA应用。当页面非常多时，打包的文件会变得很大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。使用import，再结合webpack，很容易就能做到。

首先，定义一个能够被 Webpack 自动代码分割的异步组件：

```
const AsyncComponent = () => import('./AsyncComponent.vue')
```
然后在路由定义中去引用它：

```
const router = new VueRouter({
  routes: [
    { path: '/asyncComponent', component: AsyncComponent }
  ]
})
```
当然，如果你想对你的组件使用不同的名字，也很容易做到：

```
const asyncComponent = () => import(/* webpackChunkName: "async-component" */ './asyncComponent.vue')
```
这样打包出来的分块名就是async-component.[hash].js。

### 运行时构建
如果在应用中你只使用了渲染函数(render functions)（在单文件组件中，会把组件编译成render function），而不需要html 模板(templates)，那么其实你不需要vue的模板编译器，这样在webpack打包的时候大约会减少25%的体积。当使用*import vue from 'vue*'时，默认使用运行时构建，你也可以改变webpack的配置：

```
resolve: {
  alias: {
    'vue$': 'vue/dist/vue.esm.js' // Use the full build
  }
},
```
### 在生产环境中去掉warnings and error信息
把*process.env.NODE_ENV*设置成*production*，就可以去掉这些不需要的信息代码。

```
if (process.env.NODE_ENV === 'production') {
  module.exports.plugins = (module.exports.plugins || []).concat([
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    }),
    new webpack.optimize.UglifyJsPlugin()
  ])
}
```
### CDN化
如果使用vue-cli，配置CDN路径很简单：

```
module.exports = {
  build: {
    assetsSubDirectory: 'static',
    assetsPublicPath: './',  // 这里换成CDN地址
  }
```
这样对于打包后的js和css路径引用是没有问题的，但是js里使用绝对路径或相对路径引用的图片却没有正常引用CDN地址。

```
export default {
    data() {
        return {
            img: './logo.png'
        }
    }
}
```
这里换一种方式就能正确引用了：

```
import logo from './logo.png'

export default {
    data() {
        return {
            img: logo
        }
    }
}
```


### 代码层面
v-show or v-if：如果不是频繁的切换显示，我更倾向于使用v-if，它可以减少页面中dom总数，并且它的切换时间肉眼基本没有什么感觉的。


### 参考资料
1、[潜谈vue项目优化](https://segmentfault.com/a/1190000009443366)

2、[vue路由懒加载](https://router.vuejs.org/zh-cn/advanced/lazy-loading.html)

3、[vue-js-code-splitting-webpack](https://vuejsdevelopers.com/2017/07/03/vue-js-code-splitting-webpack/)

4、[vue-js-boost-your-app-with-webpack](https://vuejsdevelopers.com/2017/06/18/vue-js-boost-your-app-with-webpack/)