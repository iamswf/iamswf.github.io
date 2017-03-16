---
layout: post
title:  "ES6解构"
date:   2016-11-15
summary: "对象和数组在JavaScript中经常会用到，同时由于JSON数据格式本身就使用JavaScript语法表示，因此对象和数组是JavaScript的重要组成部分。在JavaScript中，定义对象和数组并从中抽出部分数据非常常见。ECMAScript 6增加了解构（destructuring）简化了这项操作，解构顾名思义就是将对象或者数组拆分成更小的数据部分。本章介绍怎么在对象和数组中使用解构。"
category: javascript
---

## 解构，为了更好的数据访问

对象和数组在JavaScript中经常会用到，同时由于JSON数据格式本身就使用JavaScript语法表示，因此对象和数组是JavaScript的重要组成部分。在JavaScript中，定义对象和数组并从中抽出部分数据非常常见。ECMAScript 6增加了解构（destructuring）简化了这项操作，解构顾名思义就是将对象或者数组拆分成更小的数据部分。本章介绍怎么在对象和数组中使用解构。

### 解构为什么有用？

在编写JavaScript程序的过程中，经常会有从对象或者数组中抽出部分数据的需要，这在ECMAScript 5或者更早的版本中会导致很多相似的代码：

```javascript
let options = {
  repeat: true,
  save: false
};

// extract data from the object
let repeat = options.repeat,
    save = options.save;
```

这段代码从`options`对象中抽取出`repeat`和`save`，并且保存到了同名的局部变量中。虽然这段代码看起来很简单，但是设想一下你有大量的变量需要赋值就会意识到问题的存在。如果想从嵌套的对象或数组中抽取数据，你可能还需要深入整个数据结构去找到某个数据。

这就是ECMAScript 6为对象和数组添加解构的原因。有了解构，从对象或者数组中抽取数据变得轻松许多。很多编程语言为了使解构简单易用添加了新的语法。而ECMAScript 6利用你已熟知的对象和数组字面量语法实现了解构。

### 对象的解构

对象的解构语法：在赋值运算符的左边使用对象字面量。例如：

```javascript
let node = {
  type: 'Identifier',
  name: 'foo'
};

let {type, name} = node;

console.log(type); // "Identifier"
console.log(name); // "foo"
```

这点代码里，对象`node`的属性值`node.type`和`node.value`分别赋值给了局部变量`type`和`name`。~~This syntax is the same as the object literal property initializershorthand introduced in Chapter 4.~~ 标识符`type`和`name`不仅是对象`node`的属性名，还用来声明局部变量。

#### 切记初始化值

当使用`var`，`let`或者`const`来声明解构时，必须同时提供初始化值。下面的代码犹豫没提供初始化值会报错：

```javascript
// syntax error
var {type, name};

// syntax error
let {type, name};

// syntax error
const {type, name};
```

#### 解构赋值

前面介绍了解构例子全部是用于变量赋值。解构也可以用于赋值。例如，你可能会像下面这样改变已定义变量的值：

```javascript
let node = {
  type: 'Identifier',
  name: 'foo'
},
    type = 'Literal',
    name = 'bar';

// assgin different valus using destructuring
({type, name} = node);

console.log(type); // "Identifier"
console.log(name); // "foo"
```

这里，`type`和`name`在声明时进行了初始化。然后下一行使用`node`对象的解构赋值改变了这两个变量的值。注意解构赋值语句必须使用圆括号包括，原因是大括号在JavaScript中被认为是块语句，而块语句不能是左值。用圆括号会提示后面的大括号是一个表达式而不是一个块语句，从而保证完成解构赋值操作。

解构赋值表达式的计算结果是赋值操作符（即=）右边的的值。这意味着你可以在任何期望某个值的地方使用解构赋值表达式。例如给函数传值：

```javascript
let node = {
  type: 'Identifier',
  name: 'foo'
},
    type = 'Literal',
    name = 'bar';

function outputInfo(value) {
  console.log(value === node); // true
}

outputInfo({type, name} = node);

console.log(type); // "Identifier"
console.log(name); // "foo"
```

函数`outputInfo()`调用时传的实参是一个解构赋值表达式。因为赋值操作符右边的值是node，因此该解构赋值表达式的计算结果是也是`node`。`node`作为实参传入函数，而对`type`和`name`的赋值照常进行。

注意，在解构赋值表达式中，如果赋值操作符右值的计算结果是`null`或者`undefined`会报错。这是因为从`null`或者`undefined`中读取属性值会报错。

#### 默认值

在解构赋值中，如果使用被解构对象中不存在的属性名作为局部变量名，则该局部遍历被赋值`undefined`，例如：

```javascript
let node = {
  type: 'Identifier',
  name: 'foo'
};

let {type, name, value} = node;

console.log(type); // "Identifier"
console.log(nam); // "foo"
console.log(value); // undefined
```

这里定义了一个局部变量`value`并尝试赋值。但是在被解构对象`node`中没有对应的属性，所以该局部变量被赋值`undefined`。

当被解构对象中某个属性不存在时，你可以为被赋值的局部变量指定默认值。指定默认值的具体做法是，在局部变量右边加赋值操作符（即=），并指定默认值即可：

```javascript
let node = {
  type: 'Identifier',
  name: 'foo'
};

let {type, name, value = true} = node;

console.log(type); // "Identifier"
console.log(name); // "foo"
console.log(value); // true
```

这里，变量`value`指定默认值为`true`，默认值只有在被解构对象`node`中没有相应的属性名`value`或者值为`undefined`时才生效。~~This works similarly to the default parameter values for functions, as discussed in Chapter 3.~~

#### 变量名与对象属性名不同时的赋值

前面的每个解构赋值的例子都是使用被解构对象的属性名作为局部变量名，例如`node.type`的值赋值给变量`type`。这仅适用于变量名与属性名相同的场景。~~ECMAScript 6 has an extended syntax that allows you to assign to a local variable with a different name, and that syntax looks like the object literal nonshorthand property initializer syntax.~~例如：

```javascript
let node = {
  type: 'Identifier',
  name: 'foo'
};

let {type: localType, name: localName} = node;

console.log(localType); // "Identifier"
console.log(localName); // "foo"
```

这里使用解构赋值声明变量`localType`和`localName`，并分别被赋值为`node.type`和`node.name`。`type: localType`的意思是从被解构对象`node`中读取属性`type`并将其值赋值给变量`localType`。这种语法与对象字面量语法正好相反。因为对象字面量中属性名在冒号左边，属性值在冒号右边。而这里变量名在冒号右边，被赋的值（确切的说是值对应的被解构对象属性名）在冒号左边。

变量名与对象属性名不同时也可以使用默认值。方法还是在局部变量后跟赋值操作符和默认值。例如：

```javascript
let node = {
  type: 'Identifier'
};

let {type: localType, name: localName = 'bar'} = node;

console.log(localType); // "Identifier"
console.log(localName); // "bar"
```

这里，变量`localName`具有默认值`bar`。因为被解构对象没有属性`name`，所有该变量被赋予默认值。

到目前为止，我们知道如何解构属性是原生类型的对象。当然，对象解构也可以用于嵌套对象。

#### 嵌套对象的解构

使用与对象字面量相似的语法，可以从嵌套对象中抽出需要的数据。例如：

```javascript
let node = {
  type: 'Identifier',
  name: 'foo',
  loc: {
    start: {
      line: 1,
      column: 1
    },
    end: {
      line: 1,
      column: 4
    }
  }
};

// extract node.loc.start
let {loc: {start}} = node;

console.log(start.line); // 1
console.log(start.column); // 1
```

这里，使用大括号表明需要从被解构对象的属性`loc`中寻找`start`属性。只要是在解构表达式中，冒号左边表示值在被解构对象中的位置，冒号右边表示被赋值的变量。如果冒号右边是大括号，则表示值在被解构对象中再深一层。

嵌套对象解构时也可以使用与属性名不同的变量名：

```javascript
let node = {
  type: 'Identifier',
  name: 'foo',
  loc: {
    start: {
      line: 1,
      column: 1
    },
    end: {
      line: 1,
      column: 4
    }
  }
};

// extract node.loc.start to localStart
let {loc: {start: localStart}} = node;

console.log(localStart.line); // 1
console.log(localStart.column); // 1
```

这里，`node.loc.start`被赋值给了`localStart`。

使用嵌套解构时要注意不要创建无效的声明。空大括号在解构中是合法的，但是并没有任何作为。例如：

```javascript
// no variables declared!
let {loc: {}} = node;
```

这里没有声明任何变量。

### 数组的解构

解构数组和解构对象语法很相似，对象的解构使用对象字面量语法，数组的解构使用数组字面量语法。对象的解构依据是被解构对象的属性名，而数组的解构依据的则是数组中元素的位置。例如：

```javascript
let colors = ['red', 'green', 'blue'];

let [firstColor, secondColor] = colors;

console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

上面代码从数组`colors`中抽出值`"red"`和`"green"`，分别赋值给变量`firstColor`和`secondColor`。之所以解构这两个值，是基于它们在数组`colors`中的位置，被赋值的变量名则可以是任意的。注意，解构完之后原数组保持不变。

在解构数组时你可以忽略那些不感兴趣的数组元素位置，而仅对感兴趣的数组元素提供变量名称。例如，如果你仅仅想解构数组中第三个元素，那就不必为前两个数组元素提供变量名。具体可以看下面的代码：

```javascript
let colors = ['red', 'green', 'blue'];

let [ , , thirdColor] = colors;

console.log(thirdColor); // "blue"
```

这里使用数组解构从数组`colors`中抽出第三个元素，并赋值给变量`thirdColor`。变量`thirdColor`前面的逗号是被解构数组中相应位置元素对应的占位符。这样可以非常容易的解构出数组中特定位置上的元素。

注意，与对象的解构一样，在使用`var`，`let`或者`const`声明数组解构时，需要提供初始化值，否则会报错。

#### 解构赋值

赋值时也可以解构数组。与对象解构赋值不同的是数组的解构赋值不需要圆括号包围。例如：

```javascript
let colors = ['red', 'green', 'blue'],
    firstColor = 'black',
    secondColor = 'purple';

[firstColor, secondColor] = colors;

console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

可以看出，这里的解构赋值与前面的数组解构声明并赋值的例子基本相同。唯一不同点就是变量`firstColor`和`secondColor`在解构赋值前就已经定义过了。数组的解构赋值有个典型的应用场景：交互两个变量的值。ECMAScript 5中交换两个变量的值需要引入第三个临时变量，如下所示：

```javascript
// Swapping variables in ECMAScript 5
let a = 1,
    b = 2,
    temp;

tmp = a;
a = b;
b = temp;

console.log(a); // 2
console.log(b); // 1
```

这里为了交换变量`a`和`b`的值，第三个变量是必须的。使用数组解构赋值，不在需要第三个变量即可交换两个变量的值。请看ECMAScript 6的写法：

```javascript
// Swapping variables in ECMAScript 6
let a = 1,
    b = 2;

[a, b] = [b, a];

console.log(a); // 2
console.log(b); // 1
```

这种数组解构用法看起来如同一面镜子。

注意，与对象的解构赋值一样，如果数组解构赋值的右值计算结果是`null`或者`undefined`，将抛出错误。

#### 默认值

数组的解构允许为被解构数组任意位置指定默认值。当被解构数组中相应位置元素不存在，或者值为`undefined`时会使用指定的默认值。例如：

```javascript
let colors = ['red'];

let [firstColor, secondColor = 'green'] = colors;

console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

这里数组`colors`仅含一个元素，因此`secondColor`没有能够匹配上的数组元素，由于指定了默认值，因此`secondColor`被赋值为指定的默认值`secondColor`，而不是`undefined`。

#### 嵌套解构

与对象的嵌套解构一样，数组解构也可以嵌套，如下所示：

```javascript
let colors = ['red', ['green', 'lightgreen'], 'blue'];

let [firstColor, [secondColor]] = colors;

console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

与对象的嵌套解构一样，数组嵌套解构层次也可以任意深。

#### 剩余元素（Rest Items）

第三章介绍过函数的剩余参数（rest parameters），数组解构有个相似的概念叫做`剩余元素`。`剩余元素`使用`...`语法将一个数组中剩余的元素赋值给某个变量。例如：

```javascript
let colors = ['red', 'green', 'blue'];

let [firstColor, ...restColors] = colors;

console.log(firstColor); // "red"
console.log(restColors.length); // 2
console.log(restColors[0]); // "green"
console.log(restColors[1]); // "blue"
```

JavaScript数组缺失的一个很重要的功能是数组拷贝。ECMAScript 5使用`concat()`方法拷贝数组。例如：

```javascript
// cloning an array in ECMAScript 5
var colors = ['red', 'green', 'blue'];
var clonedColors = colors.concat();

console.log(clonedColors); // "[red,green,blue]"
```

`concat()`方法一般用来链接两个数组，如果调用改方法时不传参数，则会返回调用数组的一份克隆。在ECMAScript 6中可以使用数组解构中的剩余元素完成数组拷贝：

```javascript
// cloning an array in ECMAScript 6
let colors = ['red', 'green', 'blue'];
let [...clonedColors] = colors;

console.log(clonedColors); // "[red,green,blue]"
```

从代码的可读性来看，ECMAScript 6的这种数组拷贝方法比`concat()`方式要好。

注意，剩余元素在解构表达式中必须在数组最后一个位置。即便是后面出现逗号占位符也会报错。

### 混合解构

对象解构和数组解构可以同时使用。混合数据结构由对象和数组组成，可以使用混合解构从这种数据结构中抽取出需要的数据。例如：

```javascript
let node = {
  type: 'Indertifier',
  name: 'foo',
  loc: {
    start: {
      line: 1,
      column: 1
    },
    end: {
      line: 1,
      column: 4
    }
  },
  range: [0, 3]
}

let {
  loc: {start},
  range: [startIndex]
} = node;

console.log(start.line); // 1
console.log(start.column); // 1
console.log(startIndex); // 0
```

这里从对象`node`中抽取出`node.loc.start`和`node.range[0]`，分别赋值给`start`和`startIndex`。这种用法一般用来从JSON结构中抽取数据。




​               
​           
​       
​   




​               
​           
​       
​   







