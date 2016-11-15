---
layout: post
title:  "Understanding ES6 - 函数"
date:   2015-03-11 23:26:52
summary: "函数在任何编程语言中都非常重要，在ECMAScript6之前，JavaScript的函数变化并不大。在ECMAScript6之前使用函数容易引起错误，而且经常为了实现一些基本逻辑写更多代码。ECMAScript6从开发者的角度出发向前迈了一大步，使得在JavaScript中使用函数更加不易错，而且功能更加强大。"
category: typora
---

## 函数

函数在任何编程语言中都非常重要，在ECMAScript6之前，JavaScript的函数变化并不大。在ECMAScript6之前使用函数容易引起错误，而且经常为了实现一些基本逻辑写更多代码。ECMAScript6从开发者的角度出发向前迈了一大步，使得在JavaScript中使用函数更加不易错，而且功能更加强大。

### 默认参数

JavaScript中的函数有个非常独特的特性：不论函数定义时声明了几个形参，函数调用时允许传入任意多个参数。这个特性使我们定义的函数能够处理不同的参数个数。函数调用时未提供的参数可以指定默认值。本章讨论在ECMAScript6中以及之前如何使用默认参数，`arguments`，使用表达式作为参数，以及另一个TDZ。

#### ECMAScript5中默认参数的用法

在ECMAScript5中或者之前，一般用下面的做法创建带默认参数的函数：

```javascript
function makeRequest(url, timeout, callback) {
  timeout = timeout || 2000;
  callback = callback || function () {};
  
  // the rest of the function
}
```

这里`timeout`和`callback`均提供了默认参数，因此函数调用时这两个参数可传可不传。当第一个操作数为`false`时，逻辑运算符（`||`）会返回第二个操作数作为运算结果。这种做法有个缺点，假如我们传的`timeout`的值为合法值`0`，那么在函数内部也会被替换为`2000`。

由于这个原因，可以用下面这种更安全的做法：

```javascript
function makeRequest(url, timeout, callback) {
  timeout = (typeof timeout !== 'undefined') ? timeout : 2000;
  callback = (typeof calback !== 'undefined') ? callback : function () {};
  
  // the rest of the function
}
```

这种实现方式更加健壮，但是为了完成这样一个小逻辑却需要很多额外代码。

#### ECMAScript6中默认参数的用法

在ECMAScript6中为函数提供默认参数非常简单：

```javascript
function makeRequest(url, timeout = 2000, callback = function () {}) {
  // the rest of the function
}
```

这个函数调用时只要求必须提供第一个参数，另外两个参数有默认值。这种实现方式使得函数体更加简洁。

如果`makeRequest`调用时三个参数均提供，则忽略默认值。例如：

```javascript
// uses default timeout and callback
makeRequest('/foo');

// uses default callback
makeRequest('/foo', 500);

// doesn't use defaults
makeRequest('/foo', 500, function (body) {
  doSomething(body);
});
```

函数调用时参数`url`必须提供，另外两个有默认参数的可以选择提供。

#### 默认参数对arguments的影响

TODO：此处未完待续

#### 默认参数表达式

默认参数不一定是原始数据类型，也可以是用于获取数据的函数：

```javascript
function getValue() {
  return 5;
}
function add(first, second = getValue()) {
  return first + second;
}
console.log(add(1, 1)); // 2
console.log(add(1)); // 6
```

这里如果函数调用时不提供第二个参数，则调用`getValue()`来获取默认值。注意`getVaue()`仅当未提供第二个参数时`getValue`函数才会执行。需要注意下面这种用法：

```javascript
let value = 5;
function getValue() {
  return value++;
}
function add(first, second = getValue()) {
  return first + second;
}
console.log(add(1, 1)); // 2
console.log(add(1)); // 6
console.log(add(1)); // 7
```

默认参数表达式有一个有趣的用法：可以将前面的参数提供给后面的参数表达式使用：

```javascript
function add(first, second = first) {
  return first + second;
}
console.log(add(1, 1)); // 2
console.log(add(1)); // 2
```

更近一步，你可以将前面的参数传给一个函数为后面的参数提供默认值：

```javascript
function getValue(value) {
  return value + 5;
}
function add(first, second = getValue(first)) {
  return first + second;
}
console.log(add(1, 1)); // 2
console.log(add(1)); // 7
```

这种在默认参数表达式中引用其他参数的用法仅适用于后面的参数引用前面的，反之不行：

```javascript
function add(first = second, second) {
  return first + second;
}
console.log(add(1, 1)); // 2
console.log(add(undefined, 1)); // throws error
```

为了弄明白这样用为什么会报错，需要理解临死区TDZ。

#### 默认参数的临死区TDZ

TODO：此处未完待续

### 使用未命名参数

Javascript提供的`arguments`对象能够在不显示定义的情况下访问函数的所有参数。虽然在大多数情况下使用`arguments`都工作良好，但是用法却不优雅。例如：

```javascript
function pick(object) {
  let result = Object.create(null);
  
  // start at the second parameter
  for (let i = 1, len = arguments.length; i < len; i++) {
    result[arguments[i]] = object[arguments[i]];
  }
  
  return result;
}

let book = {
  title: 'Understanding ECMAScript 6',
  author: 'Nicholas C. Zakas',
  year: 2015
};

let bookData = pick(book, 'author', 'year');

console.log(bookData.author); // 'Nicholas C. Zakas'
console.log(bookData.year); // 2015
```

这个函数模拟了*Underscore.js*中`pick()`的实现，pick从给定对象中挑出某些指定属性组成新对象。该函数定义时仅定义了一个参数，即目标对象。除了该参数之外的所有后续其他参数均为指定属性名。该实现版本有几个值得优化的地方。首先根据函数的定义并不能明显的看出该函数处理多个参数。其次在遍历`arguments`对象时需要从下标1开始，而不是更符合程序员习惯的0。ECMAScript 6提供了`剩余参数`来帮助处理这些问题。

#### 剩余参数

`剩余参数`的语法为三点（`...`）跟一个参数名。该参数在函数体内部为一个`数组`，数组的元素为所有传入的剩余的参数，这也是“剩余”参数名称的由来。例如，使用剩余参数可以重写`pick()`函数如下所示：

```javascript
function pick(object, ...keys) {
  let result = Object.create(null);
  
  for (let i = 0, len = keys.length; i < len; i++) {
    result[keys[i]] = object[keys[i]];
  }
  
  return result;
}
```

在该版本的实现中，`keys`即为一个剩余参数，这个参数包含紧跟着`object`传入的所有其他参数，不像`arguments`包含包括第一个参数的所有参数。这里使用剩余参数使我们能够优雅的从0遍历到最后。而且从函数签名我们能够参数该函数能够处理任意多个参数。

##### 剩余参数的两个限制

剩余参数的使用有两个限制。第一个限制是在一个函数定义中剩余参数最多只能有一个，而且必须是最后一个。下面的用法会报错，原因是参数`last`跟在了剩余参数`keys`后面：

```javascript
// Syntax error: Can't have a named parameter after rest parameters
function pick(object, ...keys, last) {
  let result = Object.create(null);
  
  for (let i = 0, len = keys.length; i < len; i++) {
    result[keys[i]] = object[keys[i]];
  }
  
  return result;
}
```

第二个限制是剩余参数不能用在对象字面量的`setter`上。下面的用法也会报错：

```javascript
let object = {
  // Syntax error: Can't use rest param in setter
  set name(...value) {
    // do something
  }
}
```

之所以存在这个限制是因为对象字面量的`setter`参数个数限制为1，而剩余参数包含的参数无个数限制。

##### 剩余参数对arguments对象的影响

ECMAScript 6中引入剩余参数是为了取代`arguments`。早年ECMAScript 4中就引入了剩余参数来为函数提供无限制的参数，然而ECMAScript 4并未诞生，该想法在ECMAScript 6中保留了下来。`arguments`能够和剩余参数协同工作，例如：

```javascript
function checkArgs(...args) {
  console.log(args.length);
  console.log(arguments.length);
  console.log(args[0], arguments[0]);
  console.log(args[1], arguments[1]);
}

checkArgs('a', 'b');
```

执行`checkArgs()`的输出为：

```javascript
2
2
a a
b b
```

无论是否使用剩余参数，`arguments`对象总是能够正确的表示传入的参数。

### 构造函数Function的新功能

TODO：此处未完待续

### 延展运算符

跟剩余参数紧密相联的是延展运算符。剩余运算符将多个参数合并进一个数组，而延展运算符将数组拆分为多个独立元素传入函数进行调用。剩余运算符用于函数定义，延展运算符用于函数调用。以`Math.max()`为例，该函数接受任意多个参数，返回最大值：

```javascript
let value1 = 25,
    value2 = 50;
console.log(Math.max(value1, value2)); // 50
```

这里你仅需要求两个值的最大值，`Math.max()`使用起来非常简单。但是当我们要求的是一个数组中所有元素的最大值怎么办？`Math.max()`不允许传入数组，因此在ECMAScript 5中和之前都是通过`apply()`：

```javascript
let values = [25, 50, 75, 100];

console.log(Math.max.apply(Math, values)); // 100
```

这样虽然可行，但是在这里使用`apply()`可能会让人困惑。ECMAScript 6中的延展运算符能够很优雅的处理这种情况。无需调用`apply()`，你可以直接传入`Math.max()`数组，只需要在传入的数组前面加上`...`即可。JavaScript引擎会帮你将传入的数组拆分为独立的元素传入函数，如下所示：

```javascript
let values = [25, 50, 75, 100];

// equivalent to
// console.log(Math.max(25, 50, 75, 100));
console.log(Math.max(...values)); // 100
```

这里`Math.max()`的用法变得非常简洁。我们可以将延展运算符和其他普通参数混用。如果你想让`Math.max()`返回值最小为0（数组元素全为负数的情况），可以使用延展运算符传入待比较的数组，同时单独传入参数0：

```javascript
let values = [-25, -50, -75, -100];

console.log(Math.max(...values, 0)); //0
```

这里传入`Math.max()`的最后一个参数为0，而其他待比较参数仍旧使用延展运算符传入。在函数调用传参时使用延展运算符可以方便我们使用数组作为函数参数，可以代替很多使用`apply()`的场景。

### 函数的name属性

TODO：未完待续



### 块级作用域中的函数

在ECMAScript 3中和之前，在一个块内声明函数会报错，然而所有浏览器都支持这种用法，但是每一种浏览器的支持程度存在细微差别，因此最好能避免在块级作用域内声明函数（其实可以用函数表达式替代）。为了限制这种兼容性，ECMAScript 5的严格模式要求块级作用域内不能出现函数声明，否则会报错：

```javascript
'use strict';

if (true) {
  // throws a syntax error in ES5, not so in ES6
  function doSomething() {
    // empty
  }
}
```

在ECMAScript 5，这里会抛出错误。在ECMAScript 6，函数`doSomething()`为块级作用域函数，在定义该函数的块级作用域可以访问该函数，例如：

```javascript
'use strict';
if (true) {
  console.log(typeof doSomething); // 'function'
  function doSomething() {
    // ...
  }
  doSomething();
}
console.log(typeof doSomething); // 'undefined'
```

块级作用域中定义的函数会被提升块级作用域的顶部，因此尽管出现在函数声明之前`typeof doSomething`返回`function`。这里`if`块语句执行完之后，`doSomething`即被销毁。

#### 何时使用块级作用域函数

这里我们对比两种函数用法来讨论，一种是块级作用域函数，另一种是`let`函数表达式。这两种有一个相同点就是当函数所在块执行完之后函数即被销魂。这两种用法的重要区别是块级作用域函数会被提升至函数作用域顶部，而`let`函数表达式则不存在函数提升现象。

```javascript
'use strict';
if (true) {
  console.log(typeof doSomething); // throws error
  let doSomething = function () {
    // ...
  }
  doSomething();
}
console.log(typeof doSomething); // 'undefined'
```

这里会报错，因为执行`typeof doSomething`时，`let`声明的函数表达式还未执行，这时`doSomething()`还在临死区。了解这两种用法的区别，我们可以自由选择是否期望定义的函数在块级作用域内被提升。

#### 非严格模式的块级作用域

ECMAScript 6允许在非严格模式下使用块级作用域，但是行为与严格模式下稍微不同。在非严格模式下块级作用域中定义的函数会被提升至函数作用域或者全局，而不是仅仅提升至块级顶部。例如：

```javascript
// ECMAScript 6中的行为
if (true) {
  console.log(typeof doSomething); // 'function'
  function doSomething() {
    // ...
  }
  doSomething();
}
console.log(typeof doSomething); // 'function'
```

这里`doSomething()`被提升至了全局，因此在`if`块语句之外也能访问到该函数。ECMAScript 6将这种行为标准化了，限制了浏览器的兼容性问题。

### 箭头函数

顾名思义，箭头函数的语法中含有“箭头”（`=>`）。箭头函数与传统的JavaScript函数含有多个不同之处：

* **无`this`，`super`，`arguments`，和`new.target`绑定**-在箭头函数内部，`this`，`super`，`arguments`，和`new.target`由最近的包含该箭头函数的函数决定。
* **无法通过`new`调用**-箭头函数不含有`[[Construct]]`方法，因此无法当做构造函数使用。当通过`new`调用时会抛出错误。
* **无`prototype`**-正因为无法通过`new`调用箭头函数，所以也没有原型prototype存在的必要。箭头函数的`prototype`属性不存在。
* **无法改变`this`**-箭头函数内部`this`无法被更改，整个函数的执行过程保持不变。
* **没有`arguments`对象**-因为箭头函数内部没有绑定`arguments`，在箭头函数内部只能通过命名参数和剩余参数来访问函数参数。
* **No duplicate named parameters**-未完待续

之所以会存在这些区别是有原因的。第一个同时也是最重要的原因是JavaScript中的`this`绑定非常容易引起错误。在JavaScript函数内部很容易弄混`this`的指向，这非常容易导致一些错误的发生，而箭头函数则不存在这个问题。第二，普通函数可以当做构造函数来使用，因此普通函数中的`this`指向不定，而箭头函数内部只有一个`this`，因此JavaScript引擎能够很容易的优化箭头函数内部的操作。

#### 箭头函数的语法

箭头函数的语法根据需要有多种。箭头函数由函数参数，箭头和函数体组成。函数参数和函数体根据需要有不同的形式。例如下面的箭头函数接受一个参数并返回其值：

```javascript
var reflect = value => value;
// effectively equivalent to:
var reflect = function (value) {
  return value;
};
```

这里仅有一个参数，因此不需要括号包围参数列表，参数后面是箭头，箭头函数右边的表达式计算并作为箭头函数的返回值。即便这里没有显示的`return`语句，该箭头函数也能够正确返回。

如果传入多个参数，则必须用小括号包围参数列表：

```javascript
var sum = (num1, num2) => num1 + num2;
// efectively equivalent to:
var sum = function (num1, num2) {
  return num1 + num2;
}
```

如果没有参数，则必须用一对小括号表示空参数列表：

```javascript
var getName = () => 'Nicholas';
// effectively equivalent to
var getName = function () {
  return 'Nicholas';
};
```

箭头函数的函数体也可以像普通函数那样，用一对大括号包围：

```javascript
var sum = (num1, num2) => {
  return num1 + num2'
};
// effectively equivalent to:
var sum = function (num1, num2) {
  return num1 + num2;
};
```

注意在箭头函数的函数体内无`arguments`绑定，因此无法引用该对象。

空箭头函数的函数体也需要用大括号包围：

```javascript
var doNothing = () => {};
// effectively equivalent to:
var doNothing = function () {};
```

如果想在箭头函数内部直接返回对象字面量，需要用小括号包围该对象字面量。用小阔号包围对象字面量用来指示大括号是对象字面量，而不是函数体：

```javascript
var getTempItem = id => ({id: id, name: 'Temp'});
// effectively equivalent to:
var getTempItem = function (id) {
  return {
    id: id,
    name: 'Temp'
  }
};
```

#### 箭头函数中无this绑定

JavaScript中函数内部的`this`绑定非常容易引发错误。例如：

```javascript
var PageHandler = {
  id: '123456',
  init: function () {
    document.addEventListener('click', function (event) {
      this.doSomething(event.type); // error
    }, false);
  },
  doSomething: function (type) {
    console.log('handling ' + type + ' for ' + this.id);
  }
};
```

这里`PageHandler`用来处理页面交互。`init()`方法调用时会监听页面的交互行为，监听到响应的点击事件后会调用`this.doSomething()`。然而并不是这样。调用`this.doSomething()`会报错，因为`this`是对事件目标对象的引用（这里是`document`），而不是绑定到`PageHandler`。而`document`对象不存在`doSomething()`方法，因此会报错。为解决这个问题，可以使用`bind()`将`this`手动绑定到`PageHandler`：

```javascript
var PageHandler = {
  id: '123456',
  init: function () {
    document.addEventListener('click', (function (event) {
      this.doSomething(event.type); // no error
    }).bind(this), false)
  },
  doSomething: function (type) {
    console.log('Handling ' + type + ' for ' + this.id);
  }
};
```

这样代码能够如期运行，但是看起来有点奇怪。使用`bind(this)`实际上是创建了一个新的将`this`绑定到`PageHandler`的函数。为了避免额外创建一个函数，更好的解决办法是使用箭头函数。箭头函数中没有`this`绑定，也就是说箭头函数的`this`由箭头函数外最近的非箭头函数的`this`决定。如果一个箭头函数由一个非箭头函数包含，则该箭头函数的`this`和该非箭头函数的`this`引用保持一样，否则箭头函数中的`this`为undefined。可以使用箭头函数重写前面的代码：

```javascript
var PageHandler = {
  id: '123456',
  init: function () {
    document.addEventListener('click', event => this.doSomething(event.type), false);
  },
  soSomething: function (type) {
    console.log('Handling ' + type + ' for ' + this.id);
  }
};
```

这里的`click`事件回调函数是一个箭头函数，箭头函数调用了`this.doSomething()`。回调箭头函数中的`this`和`init()`中的`this`保持一致，因此这版重写的代码和前面的`bind()`版本一样正常工作。这里需要注意的是，尽管`doSomething()`没有返回值，但是箭头函数函数体仅包含一条语句，因此可以省略函数体的大括号。

箭头函数设计的初衷是作为“一次性使用”的函数，因此不能用来作为构造函数，箭头函数无`prototype`属性也证明了这一点。如果使用`new`运算符调用箭头函数会报错：

```javascript
var MyType = () => {},
    object = new MyType(); // error - you can't use arrow functions with 'new'
```

犹豫箭头函数中的`this`由外界函数决定，因此也不能使用`call()`，`apply()`，`bind()`来改变箭头函数的`this`绑定。

#### 箭头函数和数组

箭头函数的简洁语法与数组配合使用再合适不过了。例如你想使用自定义比较器对数组排序一般如下所示：

```javascript
var result = values.sort(function (a, b) {
  return a - b;
});
```

这种写法对于实现这么简单的操作来说有点繁琐，我们可以使用更加简短的箭头函数：

```javascript
var result = values.sort((a, b) => a - b);
```

与`sort()`一样，数组的其他接收回调函数作为参数的`map()`，`reduce()`等方法均可以使用箭头函数使代码写起来更加简洁明了。

#### 无arguments绑定

尽管箭头函数内部无`arguments`对象绑定，但是箭头函数内部能够访问外围函数的`arguments`对象：

```javascript
function createArrowFunctionReturningFirstArgs() {
  return () => arguments[0];
}
var arrowFunction = createArrowFunctionReturningFirstArgs(5);
console.log(arrowFunction()); // 5
```

在`createArrowFunctionReturningFirstArgs()`函数中，`arguments[0]`被箭头函数内部引用。之后在箭头函数执行的时候，犹豫作用域链的解析，箭头函数内部依然能够访问外部函数的`arguments`对象。

#### 箭头函数的判别

尽管箭头函数语法不一样，但它仍然是函数：

```javascript
var comparator = (a, b) => a - b;

console.log(typeof comparator); // 'function'
console.log(comparator instanceof Function); // true
```

这里`typeof`和`instanceof`均揭示了箭头函数和其他函数的判断保持一致。同时跟其他函数一样，你可以在箭头函数上使用`call()`，`apply()`和`bind()`，但是箭头函数内的`this`绑定不会被改变，例如：

```javascript
var sum = (num1, num2) => num1 + num2;

console.log(sum.call(null, 1, 2)); // 3
console.log(sum.apply(null, [1, 2])); // 3

var boundSum = sum.bind(null, 1, 2);

console.log(boundSum()); // 3
```

使用`call()`和`apply()`调用`sum()`函数，传参方式与普通函数没什么区别。使用`bind()`方法创建新函数`boundSum()`，使其两个参数绑定为`1`和`2`，因此使用时无需再传入参数。

在之前使用匿名函数的任何地方都可以使用箭头函数以使代码变得更加简洁明了。

#### 尾调用优化

TODO：未完待续

### 总结

TODO：未完待续



























