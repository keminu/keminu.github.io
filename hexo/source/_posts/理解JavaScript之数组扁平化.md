---
title: 理解JavaScript之数组扁平化
date: 2018-03-06 11:12:57
tags: ['JavaScript']
---

### 什么是数组扁平化

就是将多维数组转化为一维数组，并且不能改变原来数组的值。比如说原来的数组是：

```javascript
var arr = [1, [2, [3, 4]]];
```

那么扁平化之后就是：

```javascript
[1, 2, 3, 4]
```

那么我们实现数组扁平化有什么用呢？可以用它来求几个数组的并集，也可以用它来找出差集。

### 如何实现

下面看看实现的方式。第一种很容易想到的是循环数组元素，如果还是一个数组，就递归调用该方法：

```javascript
// 方法 1
var arr = [1, [2, [3, 4]]];

function flatten(arr) {
    var result = [];
    for (var i = 0, len = arr.length; i < len; i++) {
        if (Array.isArray(arr[i])) {
            result = result.concat(flatten(arr[i]))
        }
        else {
            result.push(arr[i])
        }
    }
    return result;
}


console.log(flatten(arr))
```

这是一个简单的实现，并没有做什么优化。

我们知道[Array.prototype.toString](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/toString)可以返回一个字符串，表示指定的数组及其元素。对于数组对象，`toString` 方法返回一个字符串，该字符串由数组中的每个元素的 `toString()` 返回值经调用 `join()` 方法连接（由逗号隔开）组成。而[String.prototype.split](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/split)方法使用指定的分隔符字符串将一个[`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String)对象分割成字符串数组，以将字符串分隔为子字符串，以确定每个拆分的位置。 

```javascript
// 方法2
var arr = [1, [2, [3, 4]]];

function flatten(arr) {
    return arr.toString().split(',').map(function(item){
        return +item
    })
}

console.log(flatten(arr))
```

对于数组里面含有不同类型的数据时，这种方式就不适用了。

既然是对数组进行处理，最终返回一个值，我们就可以考虑使用 [Array.prototype.reduce](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) 来简化代码，这个方法对累加器和数组中的每个元素（从左到右）应用一个函数，将其减少为单个值。

```javascript
// 方法3
var arr = [1, [2, [3, 4]]];

function flatten(arr) {
    return arr.reduce(function(prev, next){
        return prev.concat(Array.isArray(next) ? flatten(next) : next)
    }, [])
}

console.log(flatten(arr))
```

ES6 增加了扩展运算符，用于取出参数对象的所有可遍历属性，拷贝到当前对象之中：

```javascript
var arr = [1, [2, [3, 4]]];
console.log([].concat(...arr)); // [1, 2, [3, 4]]
```

这种方法只可以扁平一层，但是可以稍微改改：

```javascript
// 方法4
var arr = [1, [2, [3, 4]]];

function flatten(arr) {

    while (arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr);
    }

    return arr;
}

console.log(flatten(arr))
```

最后放出终极大招，看看underscore是怎么实现的？

```javascript
/**
 * 数组扁平化
 * @param  {Array} input   要处理的数组
 * @param  {boolean} shallow 是否只扁平一层
 * @param  {boolean} strict  是否严格处理元素，下面有解释
 * @param  {Array} output  这是为了方便递归而传递的参数
 * 源码地址：https://github.com/jashkenas/underscore/blob/master/underscore.js#L528
 */
function flatten(input, shallow, strict, output) {

    // 递归使用的时候会用到output
    output = output || [];
    var idx = output.length;

    for (var i = 0, len = input.length; i < len; i++) {

        var value = input[i];
        // 如果是数组，就进行处理
        if (Array.isArray(value)) {
            // 如果是只扁平一层，遍历该数组，依此填入 output
            if (shallow) {
                var j = 0, len = value.length;
                while (j < len) output[idx++] = value[j++];
            }
            // 如果是全部扁平就递归，传入已经处理的 output，递归中接着处理 output
            else {
                flatten(value, shallow, strict, output);
                idx = output.length;
            }
        }
        // 不是数组，根据 strict 的值判断是跳过不处理还是放入 output
        else if (!strict){
            output[idx++] = value;
        }
    }

    return output;

}
```

来看看主要的参数解释：

- shallow true + strict false ：正常扁平一层
- shallow false + strict false ：正常扁平所有层
- shallow true + strict true ：去掉非数组元素
- shallow false + strict true ： 返回一个[]

Underscore确实有非常多值得学习的地方，更推荐Lodash。