---
layout: post
title:  "深入ES6生成器函数"
date:   2016-11-16
summary: "如果你对ES6的生成器函数不是很熟，可以先阅读基础部分ES6生成器函数基础。弄懂了基础内容，接下来就可以进一步深入研究了。"
category: javascript
---

## 深入ES6生成器函数

如果你对ES6的生成器函数不是很熟，可以先阅读基础部分[ES6生成器函数基础](https://iamswf.github.io/javascript/2016/11/16/es6-generators.html)。弄懂了基础内容，接下来就可以进一步深入研究了。

### 错误处理

ES6生成器函数的一个强大特性是其内部的代码看起来是**同步**的，即便是外部的迭代控制部分是异步的。

在生成器函数内部我们可以使用非常熟悉的`try..catch`错误处理技术，例如：

```javascript
function *foo() {
  try {
    var x = yield 3;
    console.log('x: ' + x); // may never get here!
  }
  catch (err) {
    console.log('Error: ' + err);
  }
}
```

生成器函数可能会在`yield 3`表达式这里暂停任意长时间，如果一个错误被回传给生成器函数，则`try..catch`会catch住。但是具体怎么回传一个错误到生成器函数呢？

```javascript
var it = foo();

var res = it.next(); // {value: 3, done: false}

// instand of resuming normally with another `next()` call,
// let's throw a wrench (an error) into the gears:
it.throw('Oops!'); // Error: Oops!
```

这里你可以看到我们调用了迭代器的另一个方法`throw()`，向生成器函数内部抛了一个错误，而此时在生成器函数内部就好像在`yield`表达式暂停的地方抛出了一个错误，因此`try..catch`会catch住这个错误。注意如果我们向生成器函数内`throw()`了一个错误，但是并没有在生成器函数内部catch住该错误，那么错误会再反弹出去：

```javascript
function *foo() {}
var it = foo();
try {
  it.throw('Oops!');
}
catch (err) {
  console.error('Error: ' + err); // Error: Oops!
}
```

















