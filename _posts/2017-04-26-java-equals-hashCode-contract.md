---
layout: post
title:  "Java中equals()和hashCode()之间的契约"
date:   2017-04-26
summary: "Java中的宇宙祖先类（java.lang.Object）定义了两个重要方法，分别是equals()和hashCode()，本文介绍它们之间如何协同工作，以及协同工作需要遵守的约定。"
category: java
---

## Java中equals()和hashCode()之间的契约

原文链接：[Java equals and hashCode Contract](http://www.programcreek.com/2011/07/java-equals-and-hashcode-contract/)

![java equals hasCode contract](/assets/java_equals_hash_code_contract.png)

Java中的宇宙祖先类`java.lang.Object`定义了两个重要的方法：

```java
public boolean equals(Object obj)
public int hashCode()
```

本篇博客我将首先展示一种常见的错误做法，然后解释Java中的`equals()`和`hashCode()`之间的契约到底是什么。

### 常见错误做法

下面的例子展示了大家容易犯的错误做法：

```java
import java.util.HashMap;

public class Apple {
  private String color;
  
  public Apple(String color) {
    this.color = color;
  }
  
  public boolean equals(Object obj) {
    if (obj == null) return false;
    if (!(obj instanceof Apple)) return false;
    if (obj == this) return true;
    return this.color.equals(((Apple)obj).color);
  }
  
  public static void main(String[] args) {
    Apple a1 = new Apple("green");
    Apple a2 = new Apple("red");
    
    // hashMap stores apple type and its quantity
   	HashMap<Apple, Integer> m = new HashMap<Apple, Integer>();
    m.put(a1, 10);
    m.put(a2, 10);
    System.out.println(m.get(new Apple("green")));
  }
}
```

前面这个例子中，一个`green`颜色的`Apple`对象成功存到了`hashMap`中，但是当我们从`hashMap`中读取该对象时，并未找到。本例的打印结果为`null`，然后通过调试器我们发现该对象确确实实存在于`hashMap`中。

### 由hashCode()引起的问题

出现上面这个问题是因为`Apple`类没有重写`hashCode()`方法。`equals()`和`hashCode()`方法直接的契约为：

1. 如果两个对象相等，则它们的`hash code`也必须相同；
2. 如果两个对象的`hash code`相同，这两个对象可能相等也可能不相等。

`Object`类中`hashCode()`的默认实现是对于不同的对象返回不同的值。因此在前面的例子中，创建的第二个`Apple`对象的`hash code`与第一个`Apple`对象的不同。`HashMap`内部的实现是数组的数组，第一维数组的`index`即为`key`（key为一个对象）的`hash code`。第二维数组则是为了解决hash碰撞问题，因此如果两个对象的`hash code`相同，但是这两个对象不相等（也就是`equals()`返回`false`），则这两个对象的`value`会再第二维的数组中依次存放；如果两个对象的`hash code`相同，同时这两个对象也相等（也就是`equals()`返回`true`），则第二个对象的`value`会覆盖第一个对象的`value`。

因此可以看出，第一维数组的查找为hash映射查找，效率非常高，第二维数组的查找为线性查找，效率较低。因此`hash code`的分布越分散，查找效率越高（本篇不重点讨论这个话题）。

上面的错误例子的解决方法是重新下`hashCode()`方法，这里我们使用`color`字符串的`hash code`：

```java
public int hashCode() {
  return this.color.hashCode();
}
```

