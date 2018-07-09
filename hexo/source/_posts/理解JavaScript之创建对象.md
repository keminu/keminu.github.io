---
title: 理解JavaScript之创建对象
date: 2018-06-03 10:09:49
tags: [JavaScript]
---

翻看了之前的文章，有说到，如果有一个对象的访问链是这样的*a.b.c*，那么其实它可以用数组来表示，写为[a, b, c]。要达到*a.b.c*转为*[a, b, c]*是比较简单的，只需要遍历*a*的所有可枚举的属性，再push到一个数组里面。

好了，现在考虑这样一个场景，在一颗树中，我搜索出了*c*的路径为*[a, b, c]*，如何把它转化为一个对象*a.b.c*？

为了解决这个问题，去复习了下对象的创建方式，这篇文章更像是笔记。

说了那么多的废话，现在开始吧。下面都是参考《JavaScript高级程序设计》。

### 1、工厂模式

```javascript
function createPerson(name) {
    var o = new Object();
    o.name = name;
    o.getName = function () {
        console.log(this.name);
    };
    return o;
}

var person1 = createPerson('kevin');
```

这种方式所有的对象都是从一个函数创建的。所有的实例都指向一个原型，对象无法识别。

### 2、构造函数模式

```javascript
function Person(name) {
    this.name = name;
    this.getName = function () {
        console.log(this.name);
    };
}

var person1 = new Person('kevin');
```

这种方式的实例可以识别为一个特定的类型，但是每次创建实例时，每个方法都要被创建一次。

### 2.1、构造函数模式优化

```javascript
function Person(name) {
    this.name = name;
    this.getName = getName;
}

function getName() {
    console.log(this.name);
}

var person1 = new Person('kevin');
```

这种方式解决了创建实例时，每个方法都要被创建一次的问题，但是，这代码。。

### 3、原型模式

```javascript
function Person(name) {

}

Person.prototype.name = 'keivn';
Person.prototype.getName = function () {
    console.log(this.name);
};

var person1 = new Person();
```

这样的话，原型方法不会被重新创建，但是所有的属性和方法都是被共享的，很多自定义参数也不能初始化。

### 3.1、原型模式优化1

```javascript
function Person(name) {

}

Person.prototype = {
    name: 'kevin',
    getName: function () {
        console.log(this.name);
    }
};

var person1 = new Person();
```

封装性好了一点，但是重写了原型，丢失了constructor属性。

### 3.2、原型模式优化2

```javascript
function Person(name) {

}

Person.prototype = {
    constructor: Person,
    name: 'kevin',
    getName: function () {
        console.log(this.name);
    }
};

var person1 = new Person();
```

保留了构造函数，但是所有的属性和参数还是被共享。

### 4、组合模式

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype = {
    constructor: Person,
    getName: function () {
        console.log(this.name);
    }
};

var person1 = new Person();
```

构造函数模式与原型模式的组合，该共享的共享，该私有的私有，使用最广泛的方式。但是有人就喜欢有更好的封装，所有的东西写在一块。

### 4.1、动态原型模式

```javascript
function Person(name) {
    this.name = name;
    if (typeof this.getName != "function") {
        Person.prototype.getName = function () {
            console.log(this.name);
        }
    }
}

var person1 = new Person();
```

使用这种方式时，不能用对象字面量重写原型。

```javascript
function Person(name) {
    this.name = name;
    if (typeof this.getName != "function") {
        Person.prototype = {
            constructor: Person,
            getName: function () {
                console.log(this.name);
            }
        }
    }
}

var person1 = new Person('kevin');
var person2 = new Person('daisy');

// 报错 并没有该方法
person1.getName();

// 注释掉上面的代码，这句是可以执行的。
person2.getName();
```

使用字面量方式直接覆盖 Person.prototype，并不会更改原来实例的原型的值，person1 依然是指向了以前的原型，而不是 Person.prototype。而之前的原型是没有 getName 方法的，所以就报错了！

### 5.1、寄生构造函数模式

```javascript
function Person(name) {

    var o = new Object();
    o.name = name;
    o.getName = function () {
        console.log(this.name);
    };

    return o;

}

var person1 = new Person('kevin');
console.log(person1 instanceof Person) // false
console.log(person1 instanceof Object)  // true
```

就是寄生在构造函数中的一种模式，应用场景在于比如我们想创建一个具有额外方法的特殊数组，但是又不想直接修改Array构造函数。

```javascript
function SpecialArray() {
    var array = new Array();

    array.selfFn = function () {
    };
    return array;
}
```

### 5.2、稳妥构造函数模式

```javascript
function person(name){
    var o = new Object();
    o.sayName = function(){
        console.log(name);
    };
    return o;
}

var person1 = person('kevin');

person1.sayName(); // kevin

person1.name = "daisy";

person1.sayName(); // kevin

console.log(person1.name); // daisy
```

所谓稳妥，指的是没有公共属性，而且其方法也不引用 this 的对象。适合在一些安全的环境中。

每一种模式都对应着不同的应用场景，我们使用的最多的应该是组合模式。