---
title: 理解JavaScript之数组去重
date: 2018-05-06 10:24:04
tags: ['JavaScript', '数组']
---

在大量的数据处理中，我们经常会使用到数据去重，得到唯一数组，以便展示。下面来看看常用的几种数组去重的方式。

#### 1、循环遍历

在这种方式中，通过源数组与目标数组进行比较，来判断源数组的元素在目标数组中是否唯一存在。上代码：

```javascript
const sourceArray = ['1', 1, 1, '1', 2, 2];
function unique(sourceArray) {
    // targetArray用来存储结果
    let targetArray = [];
    for (let i = 0, sourceArrayLen = sourceArray.length; i < sourceArrayLen; i++) {
        for (var j = 0, targetArrayLen = targetArray.length; j < targetArrayLen; j++ ) {
            if (sourceArray[i] === targetArray[j]) {
                break;
            }
        }

        // 如果sourceArray[i]是唯一的，那么执行完循环，j等于targetArrayLen
        if (j === targetArrayLen) {
            targetArray.push(sourceArray[i])
        }
    }
    return targetArray;
}
console.log(unique(sourceArray)); // ['1', 1, 2]
```

在内循环中，如果sourceArray[i] 与 targetArray[j] 的值不相等，说明元素是唯一的，这时候，j的值就会等于targetArray的长度。

在数组中检测某个元素是否存在，可以使用indexOf方法，这样便可以简化内循环。

```javascript
function unique(sourceArray) {
  let targetArray = [];
  for (let i = 0, len = sourceArray.length; i < len; i++) {
    const current = sourceArray[i];
    if (targetArray.indexOf(current) === -1) {
      targetArray.push(current)
    }
  }
  return targetArray;
}
const sourceArray = [1, 2, 1, 1, 1, 2];
console.log(unique(sourceArray)); // ['1', 1, 2]
```

#### 2、排序后去重

对于一个已经排序的数组，上面的方法显得效率有点低下，因为我们只需判断当前元素与上一个元素是否相同，不需要再进行循环。

```javascript
function unique(sourceArray) {
  let targetArray = [];
  const sortedSourceArray = sourceArray.concat().sort((a,b) => a-b); // 没有对源数组进行直接操作
  let pre;
  for (let i = 0, len = sortedSourceArray.length; i < len; i++) {
    let current = sortedSourceArray[i];
    if (!i || pre !== current) {
      targetArray.push(current);
    }
    pre = current;
  }
  return targetArray;
}
const sourceArray = [1, 2, 1, 1, 1, 2];
console.log(unique(sourceArray)); // [1, 2]
```

这种方式对于大量已经排序的数组效率有很大的提升，但是如果你的数据没有排序，又存在多种数据类型，那还是考虑其它方式会比较好。

#### 3、filter

若是依赖ES5提供的强大原生方法，可以大大的简化我们的代码：

```javascript
function unique(sourceArray) {
  return sourceArray.filter((item, index , arr) => sourceArray.indexOf(item) === index);
}
const sourceArray = [1, '2', 1, '1', 1, 2];
console.log(unique(sourceArray)); // [1, '2', '1', 2]
```

是不是超级简洁了。

#### 4、键值对

在用键值对判断另外一个值的时候，如果该属性存在，则说明该值是重复的。我们可以利用hasOwnProperty这个方法进行判断。

```javascript
function unique(sourceArray) {
  var obj = {};
  return sourceArray.filter(function (item, index, array) {
    return obj.hasOwnProperty(item) ? false : (obj[item] = true)
  })
}
const sourceArray = [1, '2', 1, '1', 1, 2];
console.log(unique(sourceArray)); // [1, '2']
```

只打印出来了1, '2'，字符串‘2’和数字2在对象里面表示的是同一个属性，我们其实可以用拼接字符串的形式来避免这个问题。

```javascript
function unique(sourceArray) {
  var obj = {};
  return sourceArray.filter(function (item, index, array) {
    return obj.hasOwnProperty(typeof item + item) ? false : (obj[typeof item + item] = true);
  })
}
const sourceArray = [1, '2', 1, '1', 1, 2];
console.log(unique(sourceArray)); // [1, '2', '1', 2]
```

最后如果遇到item是对象的情况，那么经过typeof item + item后将会是object[object Object]。如果要对象将转化为字符串的形式，我们可以使用JSON.stringify方法：

```javascript
function unique(sourceArray) {
  var obj = {};
  return sourceArray.filter(function (item, index, array) {
    console.log(typeof item + item);
    return obj.hasOwnProperty(typeof item + JSON.stringify(item)) ? false : (obj[typeof item + item] = true);
  })
}
const sourceArray = [1, '2', 1, '1', 1, 2];
console.log(unique(sourceArray)); // [1, '2', '1', 2]
```

#### 5、ES6

ES6的**Set**对象允许你存储任何类型的唯一值，无论是[原始值](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)或者是对象引用。我们可以利用这个对象来储存数组的唯一值。

```javascript
const unique = sourceArray => [...new Set(sourceArray)];
const sourceArray = [1, '2', 1, '1', 1, 2];
console.log(unique(sourceArray)); // [1, '2', '1', 2]
```

JavaScript竟然进化到了如此程度。