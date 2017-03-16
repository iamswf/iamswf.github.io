---
layout: post
title:  "使用ES6生成器函数处理异步问题"
date:   2016-11-16
summary: "对ES6的生成器函数熟悉之后，我们可以开始利用它做一些有意义的事情了，比如处理异步问题。"
category: javascript
---

## 使用ES6生成器函数处理异步问题

对ES6的生成器函数熟悉之后，我们可以开始利用它做一些有意义的事情了，比如处理异步问题。

生成器强大的地方在于，它**提供了一种单线程的，看似同步的代码风格，而将具体的异步实现细节隐藏起来**。这能够使我们的代码读起来非常自然，也更加易于维护，虽然本质上还是异步代码。

### 基础用法

这里我们看下使用生成器处理异步问题的基本用法，假设我们已有的待改进的代码是这样的：

```javascript
function makeAjaxCall(url, cb) {
  // do some ajax
  // call `cb(result)` when complete
}

makeAjaxCall('http://some.url.1', function (result1) {
  var data = JSON.parse(result1);
  makeAjaxCall('http://some.url.2/?id=' + data.id,
    function (result2) {
    	var resp = JSON.parse(result2);
    	console.log('The value: ' + resp.value);
  	});
});
```

我们可以使用生成器函数来改进一下：

```javascript
var it;

function request(url) {
  makeAjaxCall(url, function (response) {
    it.next(response);
  });
}

function *main() {
  var result1 = yeild request('http://some.url.1');
  var data = JSON.parse(result1);
  
  var result2 = yield request('http://some.url.2/?id=' + data.id);
  var resp = JSON.parse(result2);
  console.log('The value: ' + resp.value);
}

it = main();
it.next(); // git it all started
```

这样看起来确实自然了许多。这里`yield __`表达式总是调用异步`ajax`请求，但是假如我们本地缓存有我们需要的数据因而不需要调用`ajax`请求呢？也好办，我们可以这样修改下`request`函数：

```javascript
var cache = {};
function request(url) {
  if (cache[url]) {
    setTimeout(function () {
      it.next(cache[url]);
    }, 0);
  }
  else {
    makeAjaxCall(url, function (resp) {
      cache[url] = resp;
      it.next(resp);
    })
  }
}
```

这样就可以用同样的方式处理本地缓存的同步数据了，而我们在`*main`中的业务代码完全不用动，**这必须归功于我们将具体的异步实现逻辑抽离到了生成器函数之外**。

### 高级用法

前面讨论的基础用法对于简单的异步处理还说没问题，但是某些时候还是无法满足我们的需求，比如无法实现并行异步。我们这里采用一种新的更强大的模式：`yield promises`。

之前我们使用的是`yield request(…)`，而`request(…)`没有返回任何值，因此这里实际上是`yield undefined`。我们对`request(…)`方法稍加修改，从而使`yield request(…)`返回`promise`：

```javascript
function request(url) {
  // Note: returning a promise now!
  return new Promise(function (resolve, reject) {
    makeAjaxCall(url, resolve);
  });
}
```

我们的生成器函数依依然不变：

```javascript
function *main() {
  var result1 = yield request('http://some.url.1');
  var data = JSON.parse(result1);
  
  var result2 = yield request('http://some.url.2/?id=' + data.id);
  var resp = JSON.parse(result2);
  console.log('The value: ' + resp.value);
}
```

对于`yield promises`模式，生成器的用法如下：

```javascript
function runGenerator(g) {
  var it = g();
  var res;
  (function iterate(val) {
  	res = it.next(val);
    if (!res.done) {
      res.value.then(iterate);
    }
  })();
}

runGenerator(main);
```

接着我们看下如何在`yield promises`模式中处理本地缓存的同步数据，保持生成器函数不变，我们修改下`request`函数和`runGenerator`函数：

```javascript
var cache = {};
function request(url) {
  if (cache[url]) {
    return cache[url];
  }
  else {
    return new Promise(function (resolve, reject) {
      makeAjaxCall(url, function (resp) {
        cache[url] = resp;
        resolve(resp);
      });
    }); 
  }
}

function runGenerator(g) {
  var it = g();
  var res;
  (function iterate(val) {
  	res = it.next(val);
    if (!res.done) {
      if (typeof res.value.then === 'function') {
        res.value.then(iterate);
      }
      else {
        setTimeout(function () {
          iterate(res.value);
        }, 0);
      }
    }
  })();
}

runGenerator(main);
```

下面我们看下如果在`yield promise`中进行错误处理：

```javascript
function request(url) {
  return new Promise(function (resolve, reject) {
    makeAjaxCall(url, function (err, text) {
      if (err) reject(err);
      else resolve(text);
    })
  })
}

function *main() {
  try {
    var result1 = yield request('http://some.url.1');
  }
  catch (err) {
    console.log("Error: " + err);
    return;
  }
  var data = JSON.parse(result1);
  try {
    var result2 = yield request('http://some.url.2/?id=' + data.id);
  }
  catch (err) {
    console.log('Error: ' + err);
    return;
  }
  var resp = JSON.parse(result2);
  console.log('value: ' + resp.value);
}

runGenerator(main);
```

这里如果一个promise被`reject`了，那么失败信息会回传到生成器内部，然后就可以`catch`住进行错误处理了。下面我们看下如果在`yield promise`中进行异步并行处理：

```javascript
function *main() {
  var search_terms = yield Promise.all([
    request('http://some.url.1'),
    request('http://some.url.2'),
    request('http://some.url.3')
  ]);
  var search_results = yield request('http://some.url.4?search=' + search_terms.join('+'));
  var resp = JSON.parse(search_results);
  console.log('Search results: ' + resp.value);
}

runGenerator(main);
```

这里`Promise.all([…])`创建了一个包含三个子promise的并行promise。

### ES7 async

ES7中又新增了另外一种函数：`async function`，其作用与`runGenerator`非常类似：

```javascript
async function main() {
  var result1 = await request('http://some.url.1');
  var data = JSON.parse(result1);
  
  var result2 = await request('http://some.url.2/?id=' + data.id);
  var resp = JSON.parse(result2);
  console.log('value: ' + resp.value);
}

main();
```

正如你所看到的，`async function`可以直接调用，在其内部使用的是`await`关键字而不是`yield`，来告诉`async function`等待promise返回之后再继续执行。