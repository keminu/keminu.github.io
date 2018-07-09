---
title: 精通JavaScript正则之正则基础
date: 2017-11-05 21:23:08
tags: [正则表达式]
---

### 1.正则结构定义

有两种方式可以定义一个正则表达式

```
var reg = /pattern/;
```
或者

```
var reg = new RegExp('pattern');  // 动态匹配
```
### 2.正则标识

| 标识   | 含义                      |
| ---- | ----------------------- |
| g    | 全局（匹配多次；不同的方法对g标识的处理不同） |
| i    | 忽略字符的大小写                |
| m    | 匹配多行（在此标识下，^和$能匹配行结束符）  |
### 3.正则对象属性

| 属性         | 用法                   |
| ---------- | -------------------- |
| global     | 如果标识 g 被使用，值为true    |
| source     | 正则表达式源码文本            |
| ignoreCase | 如果标识 i 被使用，值为true    |
| lastIndex  | 下一次exec匹配开始的索引。初始值为0 |
| multiline  | 如果标识 m 被使用，值为true    |

### 4.元字符

字符的类集，比如不确定的数字、任意字符、空白符、结束符
```
// 匹配数字:  \d
"ad3ad2ad".match(/\d/g);  // ["3", "2"]

// 匹配除换行符以外的任意字符:  .
"a\nb\rc".match(/./g);  // ["a", "b", "c"]

// 匹配字母或数字或下划线 ： \w
"a5_  汉字@!-=".match(/\w/g);  // ["a", "5", "_"]

// 匹配元音字母的字符组
"adefglhi".match(/[aeiou]/g); // ["a", "e", "i", ]
```
注意，在字符组内部，一个^表示的是排除组内字符，并不是匹配行开始，就像’|'、-'一样，在字符组内外有着不同的含义

字符组与多选结构的区别：
字符组里面只匹配单个的字符，而多选结构本身就是一个正则
### 5.量词

每一个正则因子都可以用一个正则量词后缀来决定这个因子因该被匹配的次数。包围在一对花括号中的一个数字表示这个因子因该被匹配的次数。？号表示可选，就等同于{0, 1}，*等同于{0, }，+则等同于{1, }。

```
// 重复0次或多次
"test".match(/test\d*/); // ["test"]
"test123".match(/test\d*/); // ["test123"]
```
从上面的结果可以看到正则匹配的子字符串会尽可能返回多的数字，这就是在满足条件的情况下捕获尽可能多的字符-贪婪模式。

对应的”懒惰模式“，就是在满足条件的情况下捕获尽可能少的字符串，使用懒惰模式的方法，就是在字符重复标识后面加上一个 "?"

```
// 数字重复3~5次，满足条件的情况下返回尽可能少的数字
"test12345".match(/test\d{3,5}?/);  // ["test123"]
// 数字重复1次或更多，满足条件的情况下只返回一个数字
"test12345".match(/test\d+?/);  // ["test1"]
```
### 6.字符转义

在正则表达式中元字符是有特殊的含义的，当我们要匹配元字符本身时，就需要用到字符转义。

```
/\./.test("."); // true
```
### 7.分支

当正则表达式需要匹配几种类型的结果时，可以用到分支
```
"asdasd hi  asdad hello asdasd".replace(/hi|hello/,"nihao"); // "asdasd nihao  asdad hello asdasd"

"asdasd hi  asdad hello asdasd".split(/hi|hello/); // ["asdasd ", "  asdad ", " asdasd"]
```
分支条件影响它两边的所有内容， 比如 hi|hello  匹配的是hi或者hello，而不是 hiello 或者 hhello。
分组中的分支条件不会影响分组外的内容

```
"abc acd  bbc bcd ".match(/(a|b)bc/g); // ["abc", "bbc"]

```

### 8.分组

捕获型：一个捕获型分组是一个被包围在圆括号中的正则表达式分支。任何匹配这个分组的字符都会被捕获。每个捕获型分组都被指定了一个数字代表分组号。

非捕获型：非捕获型分组有一个(?:前缀。非捕获型分组仅做简单的匹配，并不会捕获所匹配的文本，也不会干扰捕获型分组的编号。

向前正向匹配：向前正向匹配分组有一个(?=前缀，类似于非捕获型分组，实际上不捕获任何匹配文本，也不分配组号。在这个组匹配后，文本会倒回到它开始的地方。

```
/hello\s(?=world)/.exec("asdadasd hello world asdasd")  // ["hello "]
```


向前负向匹配：向前负向匹配分组有一个(?!前缀。它也类似于非捕获型分组，不捕获任何匹配文本，不分配组号。只有当它匹配失败时才继续向前匹配。

```
/hello\s(?!world)/.exec("asdadasd hello world asdasd") // null

```
### 9.后向引用

正则表达式的分组可以在其后边的语句中通过 \ 紧接着一个数字组号来引用

```
// 匹配重复的单词
/(\b[a-zA-Z]+\b)\s+\1/.exec(" asd sf  hello hello asd"); // ["hello hello", "hello"]
```



### 10.方法

### RegExp.prototype.test

用来测试字符串中是否含有子字符串

```
/hello/.test("abchello");  // true
```
### RegExp.prototype.exec

这个方法从字符串中捕获满足条件的字符串到结果数组中，值得注意的是，这个方法一次只能捕获一份子字符串到结果数组中，无论正则表达式是否有全局属性。

```
var reg=/hello/g;
reg.exec("abchelloasdasdhelloasd");   // ["hello"]
```
如何做全局匹配？

在正则表达式中有分组的情况下，无论正则表达式是否有全局属性，exec函数都只返回一个结果，并捕获分组的结果。

```
/h(ell)o/g.exec("abchellodefhellog"); // ["hello", "ell"]
```



### String.prototype.search

用来找出原字符串中某个子字符串首次出现的index，没有则返回-1

```
"abchello".search(/hello/);  // 3

```
### String.prototype.replace

用来替换字符串中的子串

```
"abchello".replace(/hello/,"hi");   // "abchi"

```

正则表达式存在分组的情况下，第二个参数里边可以用 $ 紧接着一个数字组号来指代第几个分组的内容

```
" the best language in the world is java ".replace(/(java)/,"$1script"); // " the best language in the world is javascript "
```

### String.prototype.split

用来分割字符串

```
"abchelloasdasdhelloasd".split(/hello/);  // ["abc", "asdasd", "asd"]

```
### String.prototype.match

用来捕获字符串中的子字符串到一个数组中。默认情况下只捕获一个结果到数组中，正则表达式有”全局捕获“的属性时(定义正则表达式的时候添加参数g)，会捕获所有结果到数组中。

```
"ahelloasdasdhelloasd".match(/hello/);  // ["hello"]

"ahelloasdasdhelloasd".match(/hello/g);  // ["hello","hello"]
```
当没有全局属性，但是正则表达式有分组时，会返回分组匹配结果。

```
"abchellodefhellog".match(/h(ell)o/); // ["hello", "ell"]
```

## 一个例子

```
var parse_url = /^(?:([A-Za-z]+):)?(\/{0,3})([0-9.\-A-Za-z]+)(?::(\d+))?(?:\/([^?#]*))?(?:\?([^#]*))?(?:#(.*))?$/;
```

## One more thing...