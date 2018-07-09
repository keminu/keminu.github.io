---
title: 理解JavaScript之函数创建过程
date: 2018-06-09 16:32:54
tags: [JavaScript, 函数]
---

为了更好的了解JavaScript函数的执行，看了下ECMAScript标准有关函数的创建过程。

下面的伪代码展现了函数的创建算法。

```javascript
// 解释了原型链最顶部的类型
F = new NativeObject();
  
// property [[Class]] is "Function"
F.[[Class]] = "Function"
  
// a prototype of a function object
F.[[Prototype]] = Function.prototype
  
// reference to function itself
// [[Call]] is activated by call expression F()
// and creates a new execution context
// 函数调用时执行[[Call]]，并创建执行上下文
F.[[Call]] = <reference to function>
  
// built in general constructor of objects
// [[Construct]] is activated via "new" keyword
// and it is the one who allocates memory for new
// objects; then it calls F.[[Call]]
// to initialize created objects passing as
// "this" value newly created object 
// 当使用 new 操作符时会调用[[Construct]]，并为新的对象分配内存，再调用[[Call]]
F.[[Construct]] = internalConstructor
  
// scope chain of the current context
// i.e. context which creates function F
// 这里说明了使用new操作符与直接调用的不同，
// 不同的原型链，不同的this指向
F.[[Scope]] = activeContext.Scope
// if this functions is created 
// via new Function(...), then
F.[[Scope]] = globalContext.Scope
  
// number of formal parameters
// 函数参数
F.length = countParameters
  
// a prototype of created by F objects
__objectPrototype = new Object();
__objectPrototype.constructor = F // {DontEnum}, is not enumerable in loops
F.prototype = __objectPrototype
  
return F
```

需要注意的是，*F.[[Prototype]]*是函数(构造器)的原型，也被这个函数所创建的对象的原型。怎么翻译都感觉不对，自己体会吧。(*F.[[Prototype]]* is a prototype of the *function (constructor)* and *F.prototype* is a prototype *of objects created by this function*)