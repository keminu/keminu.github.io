---
title: 精通JavaScript正则之模板引擎
date: 2018-04-08 22:50:08
tags: [JavaScript,正则表达式,模板引擎]
---

在前端的开发中经常会使用到模板引擎，前端渲染中比如前端框架Knockout、Vue和Angular（React的JSX不属于模板，它是一个带语法糖的手写 AST），在解析指令语法时都会使用对应的模板引擎进行解析，而基于Node.js进行渲染的EJS，DoTjs这些，原理也都是类似。

现在我们一步步来，看看它们是怎么实现的。

首先，我想它的使用方式是这样的：

```javascript
const templateEngine = (tpl, data) => {
    // 这里模板引擎实现方式
}
const template = '<p>Hi, my name is <%name%>. My job is <%job%>.</p>';
console.log(templateEngine(template, {
    name: "nuoka",
    job: 'write bug'
}));
```

当使用templateEngine这个函数，并传入一些参数，我们期待它得到

```html
<p>Hi, my name is nuoka. My job is write bug.</p>
```

第一步我们要得到动态模板，也就是`<%name%>`这些，然后再用真实的数据去替换它。对于字符串的处理，我们很容易就想到使用正则来处理，关于正则的使用基础，可以看看前面的文章。好了，来写我们的第一个正则表达式，

```javascript
const reg = /<%([^%>]+)?%>/g;
```

通过这个正则，我们可以获取到以`<%`开头，以`%>`结尾的片段。用`exec`执行下，可以得到一个包含匹配结果的数组：

```javascript
const match = reg.exec(template);
console.log(match);
// 得到下面的输出
// [
//		"<%name%>"
//		1: "name"
//		groups: undefined
//		index: 18
//		input: "<p>Hi, my name is <%name%>. My job is <%job%>.</p>"
// ]
```

可以看到返回的数组里只包括一个匹配结果，所以这是我们还需要使用一个`while`循环来获取全部的结果。

```javascript
let match;
const reg = /<%([^%>]+)?%>/g,
      template = '<p>Hi, my name is <%name%>. My job is <%job%>.</p>';
while (match = re.exec(template)) {
    console.log(match);
}
```

好了，现在我们能够获取到所有的模板片段，只需要把数据填充进去就可以了，这里使用replace方法来实现。来看看我们最粗略的模板函数：

```javascript
const templateEngine = (tpl, data) => {
    let reg = /<%([^%>]+)?%>/g, 
        match;
    while(match = reg.exec(tpl)) {
        tpl = tpl.replace(match[0], data[match[1]])
    }
    return tpl;
}
console.log(templateEngine(template, {
    name: "nuoka",
    job: 'write bug'
}));
```

Boom，第一个模板函数实现了，是不是很简单？

这样足够了吗？然而并没有，考虑下面的情况，我们把输入数据变成这样：

```javascript
{
    name: 'nuoka',
    profile: {
        job: 'write bug',
    }
}
```

当使用`profile.job`去填充数据的时候，发现使用不了了，因为并没有找到`profile.job`这个属性。要如何解决这个问题呢？

要是`profile.job`是一个语句就好了，这样就可以通过`.`操作符去获取到job属性。通过这个思路，也容易想到，在JavaScript中要把字符串当作语句来执行，可用的方式有`eval`和`new Function`，在这里我们就用`new Function`来试试。来看一下它的基本使用方式：

```javascript
const fn = new Function('arg', 'console.log(arg + 1);');
fn(1);  // 输出2
```

上面的代码等价于：

```Javascript
const fn = (arg) => {
    console.log(arg + 1);
};
fn(1);  // 输出2
```

现在，我们可以定义一个函数，它的参数和函数体都来自于字符串，很棒，这正是我们想要的。在创建这个函数之前，我们还需要想想怎么构造出它的函数体？

这个函数体应该返回一个处理好的模板字符串，像是这样：

```javascript
return 
"<p>Hi, my name is " + 
this.name + 
"My job is" + 
this.profile.job + 
" .</p>";
```

但是这种方式并不完美，因为如果我们想在模板里嵌套循环的话，这没法做，例如：

```javascript
const template = 
'My skills:' + 
'<%for(var index in this.skills) {%>' + 
'<a href=""><%this.skills[index]%></a>' +
'<%}%>';
```

转化为等价函数：

```javascript
return
'My skills:' + 
for(var index in this.skills) { +
'<a href="">' + 
this.skills[index] +
'</a>' +
}
```

报语法错误了！

怎么办？

最终我们是要返回一个字符串的，但是又想在返回语句里把真实的语法和字符串分开，并且支持for循环，这里可以先把字符串放入数组里，再通过join方法转化为字符串：

```javascript
let r = [];
r.push('My skills:'); 
for(let index in this.skills) {
r.push('<a href="">');
r.push(this.skills[index]);
r.push('</a>');
}
return r.join('');
```

下一步，我们来重写我们的模板函数，为了方便在浏览器中测试，我们使用旧的语法：

```javascript
var templateEngine = function(tpl, data) {
    var re = /<%([^%>]+)?%>/g,
        code = 'var r=[];\n',
        cursor = 0, // 这个用来指示处理到了哪个字符
        match;
    var add = function(line) {
        // 替换掉转义字符'"'，并放入数组里
        code += 'r.push("' + line.replace(/"/g, '\\"') + '");\n';
    }
    while(match = re.exec(tpl)) {
        // 通过正则把模板和非模板分开
        add(tpl.slice(cursor, match.index));
        add(match[1]);
        cursor = match.index + match[0].length;
    }
    add(tpl.substr(cursor, tpl.length - cursor));
    code += 'return r.join("");'; // <-- return the result
    console.log(code);
    return tpl;
}
var template = '<p>Hi, my name is <%this.name%>. My job is <%this.profile.job%>.</p>';
console.log(templateEngine(template, {
    name: "nuoka",
    profile: { job: 'write bug' }
}));
```

这里要注意的是处理双引号那部分，如果不进行转义，那么它将是无效的js语法。运行一下上面的例子，看看它打印出什么来。

```javascript
var r=[];
r.push("<p>Hi, my name is ");
r.push("this.name");
r.push(". My job is ");
r.push("this.profile.job");
r.push(".");
return r.join("");
```

出了点意外，*this.name*和*this.profile.job*不是我们想要的，它们不应该被双引号包裹着。来修改一下add方法：

```javascript
var add = function(line, js) {
    js ? code += 'r.push(' + line + ');\n' 
       : code += 'r.push("' + line.replace(/"/g, '\\"') + '");\n';
}
var match;
while(match = re.exec(tpl)) {
    add(tpl.slice(cursor, match.index));
    add(match[1], true); // 匹配的字符应该是有效的js语法
    cursor = match.index + match[0].length;
}
```

最后生成了：

```javascript
var r=[];
r.push("<p>Hi, my name is ");
r.push(this.name);
r.push(". My job is ");
r.push(this.profile.job);
r.push(".");
return r.join("");
```

在templateEngine函数的最后，就可以执行生成的函数体了：

```javascript
return new Function(code.replace(/[\r\t\n]/g, '')).apply(data);
```

我们使用apply方法来调用这个函数，它生成一个函数作用域，这样里面this就自动指向了data。

到现在我们已经做得非常好了，能够支持基本的js语法，分离出模板，最后我们还想支持更多的js语法，比如说*if/else*、*for*循环。

```javascript
var template = 
'My skills:' + 
'<%for(var index in this.skills) {%>' + 
'<a href="#"><%this.skills[index]%></a>' +
'<%}%>';
console.log(templateEngine(template, {
    skills: ["js", "html", "css"]
}));
```

对于上面的模板就会抛出一个*Uncaught SyntaxError. Unexpected token for*语法错误。我们调试一下，在控制台上就会看到错误所在：

```javascript
var r=[];
r.push("My skills:");
r.push(for(var index in this.skills) {);
r.push("<a href=\"\">");
r.push(this.skills[index]);
r.push("</a>");
r.push(});
r.push("");
return r.join("");
```

*for*循环不应该放入数组里面，我们再添加一个正则表达式，把**if, *for*, *else*, *switch*, *case*, *break*, { 或者 }** 开头的单独的放在一行，

```Javascript
var re = /<%([^%>]+)?%>/g,
    reExp = /(^( )?(if|for|else|switch|case|break|{|}))(.*)?/g,  // 新增处理更多语法
    code = 'var r=[];\n',
    cursor = 0;
var add = function(line, js) {
    js ? code += line.match(reExp) 
       ? line + '\n' 
       : 'r.push(' + line + ');\n' 
       : code += 'r.push("' + line.replace(/"/g, '\\"') + '");\n';
}
```

增加一个正则处理后的结果像这样：

```javascript
var r=[];
r.push("My skills:");
for(var index in this.skills) {
r.push("<a href=\"#\">");
r.push(this.skills[index]);
r.push("</a>");
}
r.push("");
return r.join("");
```

模板渲染后就是这样啦：

```javascript
My skills:<a href="#">js</a><a href="#">html</a><a href="#">css</a>
```

当然我们还可以使用更多复杂的表达式了，比如：

```javascript
var template = 
'My skills:' + 
'<%if(this.showSkills) {%>' +
    '<%for(var index in this.skills) {%>' + 
    '<a href="#"><%this.skills[index]%></a>' +
    '<%}%>' +
'<%} else {%>' +
    '<p>none</p>' +
'<%}%>';
console.log(templateEngine(template, {
    skills: ["js", "html", "css"],
    showSkills: true
}));
```

现在再把代码稍微整理下，得到一个最终版本的模板处理函数：

```javascript
var templateEngine = function(html, options) {
    var re     = /<%([^%>]+)?%>/g, 
        reExp  = /(^( )?(if|for|else|switch|case|break|{|}))(.*)?/g, 
        code   = 'var r=[];\n', 
        cursor = 0, 
        match;
    var add = function(line, js) {
        js ? (code += line.match(reExp) 
           ? line + '\n' 
           : 'r.push(' + line + ');\n') 
           : (code += line != '' ? 'r.push("' + line.replace(/"/g, '\\"') + '");\n' : '');
        return add;
    }
    while(match = re.exec(html)) {
        add(html.slice(cursor, match.index))(match[1], true);
        cursor = match.index + match[0].length;
    }
    add(html.substr(cursor, html.length - cursor));
    code += 'return r.join("");';
    return new Function(code.replace(/[\r\t\n]/g, '')).apply(options);
}
```



参考资料：

1、https://github.com/krasimir/absurd/blob/master/lib/processors/html/helpers/TemplateEngine.js