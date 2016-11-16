---
layout: post
title:  "ES6生成器函数基础"
date:   2016-11-16
summary: "ES6引入了一种新函数类型，**生成器**（generator）。名字看起来有点古怪，但是行为更加古怪。本节将介绍ES6生成器的基础，并让我们对它的强大特性建立起初步认识。"
category: javascript
---

## ES6生成器函数基础

原文链接：[The Basics Of ES6 Generators](https://davidwalsh.name/es6-generators)

ES6引入了一种新函数类型，**生成器**（generator）。名字看起来有点古怪，但是行为更加古怪。本节将介绍ES6生成器的基础，并让我们对它的强大特性建立起初步认识。

### 运行..直到完成

我们对普通函数有一点常识性认识：一个函数一旦开始运行，在其他代码执行之前一定会运行完毕。例如：

```javascript
setTimeout(function () {
  console.log('Hello World');
}, 1);

function foo() {
  for (var i = 0; i < 1E10; i++) {
    console.log(i);
  }
}

foo();
// 0...1E10
// 'Hello World'
```

这里`for`循环会执行一段时间（正好能够大于1ms），但是定时执行的回调函数并不会打断`foo()`函数的执行，定时函数时间到之后会追加到执行队列并等待执行。

但是如果`foo()`函数能够被打断执行，会不会是在“搞事情”？其实这正是多线程编程面临的挑战。还好我们不需要在JavaScript中担心这个问题，因为JavaScript是单线程的。

### 运行..暂停..再运行

ES6生成器函数则跟普通函数不一样，它能在运行期间暂停一次或多次，而且在暂停期间允许其他代码运行。在生成器函数体内，使用`yield`关键字暂停函数允许。无法在生成器函数外部停止其运行，生成器函数只在遇到`yield`时暂停运行。但是当生成器函数遇到`yield`关键字并暂停运行时，无法在生成器内部恢复运行，只能在生成器函数外部重启生成器函数的允许。因此我们可以看到，一个生成器函数能够按我们的意愿暂停和重启任意次。

更重要的是，生成器函数的暂停和重启并不是仅仅能控制函数的执行过程，在暂停和重启同时还能够在生成器函数内部和外部之间通信。对于普通函数，我们在函数运行开始时接收参数，在函数运行结束时`return`返回值；对于生成器函数，通过`yield`向函数外部通信，通过重启向函数内部通信。

### 生成器函数的语法

让我们来看一下生成器函数的语法，首先是新的函数声明语法：

```javascript
function *foo() {
  // ..
}
```

注意到`*`了吧？看起来可能有点奇怪，仅仅用来标识该函数是一种生成器函数而已。你也许看到过其他文章使用`function* foo() {}`，而不是`function *foo() {}`（不同之处在于`*`符号的位置）来声明生成器函数。这两种都合法，但是推荐使用后者。

接下来我们看一下生成器函数的函数体。生成器函数和普通函数在函数体内只有很少区别。最主要的一点区别是`yield`关键字。`yield __`是一个“yield表达式”，而不是一条语句，因为当我们重启生成器函数时，我们会向生成器函数内部通信，传回一个值，而这个回传的值就是`yield__`表达式的计算结果。例如：

```javascript
function *foo() {
  var x = 1 + (yield 'foo');
  console.log(x);
}
```

这里`yield 'foo'`表达式会在该生成器函数暂停时发送`'foo'`字符串到外部，当该生成器函数重启时回传的任意值将作为该表达式的计算结果，并将加到`1`上，然后赋值给变量`x`。

从这个例子看到双向通信的过程了吧？我们在生成器函数暂停时将`'foo'`字符串发送到函数外部，接着会在之后的某个时刻重启该生成器函数并回传一个值进来。这看起来就像是`yield`关键字发送了一个请求（有发送参数和回传结果）。

在任何能用表达式的地方都可以使用`yield`，默认`yield`的值为`undefined`。例如：

```javascript
// note: `foo(..)` here is Not a generator!!
function foo(x) {
  console.log('x: ' + x);
}

function *bar() {
  yield; // just pause
  foo(yield); // pause waiting for a parameter to pass into `foo(..)`
}
```

### 迭代器

我们从外部控制生成器函数暂停和重启的方法是使用迭代器（iterator）。举例如下：

```javascript
function *foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
}
```

为了走一遍生成器函数`*foo()`，我们需要通过该生成器函数构建一个迭代器：

```javascript
var it = foo();
```

上面的代码会生成一个迭代器`it`，同时可以看出，调用生成器函数`foo()`并不会执行该生成器函数的函数体，而是返回一个迭代器对象。接下来我们开始一步一步进行迭代：

```javascript
var message = it.next();
console.log(message); // {value: 1, done: false}
```

我们调用迭代器的`next()`方法返回了一个包含`value`属性和`done`属性的对象，`value`属性对象生成器`yield`出的值，`done`属性用来标识生成器函数是否已运行完毕。继续迭代：

```javascript
console.log(it.next()); // {value: 2, done: false}
console.log(it.next()); // {value: 3, done: false}
console.log(it.next()); // {value: 4, done: false}
console.log(it.next()); // {value: 5, done: false}
```

这里需要注意，当输出`5`时，`done`仍然是`false`，这是因为从技术上讲生成器函数还未执行完毕。我们需要继续调用一次迭代器的`next()`方法，可以同时传入一个回传值作为`yield 5`的计算结果，这时候生成器函数才执行完毕：

```javascript
console.log(it.next()); // {value: undefined, done: true}
```

为了体会生成器函数如果处理最后一次调用`next()`，我们重写一下上面这段代码：

```javascript
function *foo() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    let last = yield 5;
    console.log('last: ' + last);
    return last + 1;
}

let it = foo();
console.log(it.next());
console.log(it.next());
console.log(it.next());
console.log(it.next());
console.log(it.next());
console.log(it.next(100));
```

执行结果如下：

```javascript
Object {
  "done": false,
  "value": 1
}
Object {
  "done": false,
  "value": 2
}
Object {
  "done": false,
  "value": 3
}
Object {
  "done": false,
  "value": 4
}
Object {
  "done": false,
  "value": 5
}
"last: 100"
Object {
  "done": true,
  "value": 101
}
```

这次最后一次调用`next()`时回传一个值`100`。这个回传的值`100`作为表达式`yield 5`的计算结果赋值给变量`last`。接着生成器函数执行最后一个`yield 5`表达式之后的所有直到`return`语句的剩余代码，这点可以根据打印的`'last: 100'`看出。`return`语句返回值作为最后一次调用`next()`的返回值，因此最后打印输出了`{value: 101, done: true}`。这里我们在生成器函数最后返回了一个返回值，但是这也许并不是一个好主意，因为如果我们使用`for..of`循环迭代执行生成器函数内的代码，最后的`return`语句返回的值会被忽略。

接下来我们看一下生成器函数内部与外部的双向通信：

```javascript
function *foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield(y / 3);
  return (x + y + z);
}

var it = foo(5);

console.log(it.next());   // {value: 6, done: false}
console.log(it.next(12)); // {value: 8, done: false}
console.log(it.next(13)); // {value: 42, done: true}
```

这里我们像普通函数那样给生成器函数传了参数`5`，因此`x`被赋值为5。

* 第一次调用`next()`，我们没有回传任何参数进生成器函数，因为这时还没有`yield`表达式接受我们的回传值。表达式`yield (x + 1)`会向外发送值`6`。
* 第二次调用`next(12)`，我们回传值`12`给处于等待状态的`yield (x + 1)`，因此`y`被赋值为`12 * 2`，为`24`。表达式`yield (y / 3)`向外发送值`8`。
* 第三次最后调用`next(13)`，我们回传值13给处于等待状态的表达式`yield (y / 3)`，因此`z`被赋值为13。最后`return`的返回值即为最后一次向外发送的值`(x + y + z)`等于`(5 + 24 + 13)`等于`42`。

第一次看到这种函数调用方式确实很奇怪，最好多看几遍。

### for..of

ES6提供了迭代器自动调用的方法：`for..of`循环。例如：

```javascript
function *foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (var v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
console.log(v); // still 5, not 6 :(
```

正如你所看到的那样，`for..of`循环忽略了`return 6`返回值。而且犹豫`for..of`循环没有暴露出迭代器的`next()`方法，我们不能回传值进生成器函数。

### 总结

这些就是生成器函数的基础内容。如果感到迷惑也别着急，我们第一次遇到生成器函数都会觉得古怪。很自然的我们会想到这个ES6加入的新类型函数能够给我们带来什么实用价值。我们本节看到的只是生成器函数的皮毛，还有很多内容我们需要深入研究才会发现它是多么强大。现在我们可以提出以下几个问题：

1. 如何处理错误？
2. 一个生成器函数能够调用另一个生成器函数吗？
3. 如何使用生成器进行异步编程？

后面的几篇文章我们会介绍这些问题。









