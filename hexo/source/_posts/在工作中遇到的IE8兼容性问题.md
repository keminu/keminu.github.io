---

title: 在工作遇到的IE8兼容性问题
date: 2016-07-02 20:33:30
categories: IE8兼容性
tags: [css,IE8兼容性]
keywords: IE8兼容性
description: 一些在工作过程中遇到的IE8兼容性问题，以及相应的解决方案。

---
<!-- toc -->
## 1、IE8不兼容ES5的一些方法，比如indexOf,lastIndexOf,filter等等
解决方法：添加es5-shim.js脚本，在github搜索相应关键字。
## 2、IE8不兼容opacity、rgba属性
解决方法：使用ie的过滤器filter
## 3、IE8不兼容border-radius属性
尝试使用border-radius.htc这个hack，但是还是没有效果，暂时还不知道是什么原因。
又尝试使用jquery.corner.js，还是没有解决。



