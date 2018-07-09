---
title: vue单页应用中设置网页标题
date: 2017-12-06 22:52:19
tags: [vue]
---

<!-- toc -->

在vue-router的单页应用中，你可能需要在不同的路由设置不同的网页标题，如果你在单文件组件里直接使用document.title来设置页面标题，是不生效的。像这样：

```
mounted() {
    document.titile = 'title';
}
```
这样是不生效的。

这里有两种方式可以做到这样的功能。一是在路由里定义对应的组件标题，然后在路由钩子里动态设置，但是这种方式在iOS app下不能被支持，因为它不允许动态改变网页标题。二是通过vue指令，在路由切换完成后，静默加载一个空的iframe来设置title。

第一种方式实现如下：

```
router.beforeEach((to, from, next) => {
  document.title = to.extra.title;
  next();
});
```
对应部分路由定义：

```
const routers = [{
    path: '/',
    extra: {
        title: '首页'
    }
}]
```

第二种方式实现：

先定义一个函数用来动态修改标题。

```
export const setMetaTitle = title => {
  document.title = title;
  const mobile = navigator.userAgent.toLowerCase();
  if (/iphone|ipad|ipod/.test(mobile)) {
    let iframe = document.createElement('iframe');
    iframe.style.visibility = 'hidden';
    iframe.setAttribute('src', 'https://www.baidu.com/');
    const iframeCallback = () => {
      setTimeout(() => {
        iframe.removeEventListener('load', iframeCallback);
        document.body.removeChild(iframe);
      }, 0);
    }
    iframe.addEventListener('load', iframeCallback);
    document.body.appendChild(iframe);
  }
}
```
再在vue的入口点比如app.js里，定义一个vue指令：

```
import { setMetaTitle } from '@/common/utils'; 

Vue.directive('title', {
  inserted(el, binding) {
    setMetaTitle(binding.value);
  }
});
```
最后在你的组件里添加这个指令：
```
<template>
    <div v-title="这里是你的标题"></div>
</template>
<script>
    export default {}
</script>
```
推荐第二种方式实现，没毛病。