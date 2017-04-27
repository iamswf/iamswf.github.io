---
layout: post
title:  "设计模式-抽象工厂模式"
date:   2017-04-07
summary: "工厂模式包括『简单工厂』模式，『工厂方法』模式以及『抽象工厂』模式，本篇主要介绍『抽象工厂』模式。"
category: designPattern
---

## 抽象工厂模式

原文链接：[Abstract Factory Pattern Tutorial With Java Examples](https://dzone.com/articles/design-patterns-abstract-factory)

工厂模式包括**简单工厂**模式，**工厂方法**模式以及**抽象工厂**模式，本篇主要介绍**抽象工厂**模式。

### 现实世界中的工厂

考虑现实世界的工厂非常简单-工厂即为生产汽车、电脑或者电视等物品的地方。维基百科关于现实世界中的工厂的定义如下：

> 工厂又称制造厂，是一所用以生产货物的大型工业楼宇。大部分工厂皆设有大型机器或设备构成的生产线。

工厂的定义很简单，那么设计模式中的工厂如何工作呢？

### 抽象工厂模式

**抽象工厂**模式是建造者模式的一种-该模式使得具体对象的创建与客户程序解耦。我们先看一下抽象工厂模式的类图：

![abstract factory pattern](/assets/abstract_factory_pattern.png)

虽然抽象工厂模式的概念比较简单，但还是有必要研究一下上述类图。我特意将`ConcreteFactory2`负责的部分标红了。`AbstractFactory`定义了所有具体工厂需要实现的接口，具体工厂只有实现了该接口才能够生产`Products`。`ConcreteFactory1`和`ConcreteFactory1`均实现了该接口，这两座工厂分别用于生产不同的产品族。

这里以袜子为例顺便解释一下我对**产品族**的理解：比如`ConcreteFactory1`为`玉珠集团`，`ConcreteFactory1`为`杨氏集团`，`玉珠袜业`生产`玉珠短袜`以及`玉珠手套`，而`杨氏集团`生产`杨氏短袜`和`杨氏手套`。这里`玉珠短袜`、`玉珠手套`以及`杨氏短袜`、`杨氏手套`分别由不同的工厂生产，即为不同的产品族。

接着讨论抽象工厂模式。`AbstractProductA`和`AbstractProductB`分别是不同大类的产品的接口，这里我想用`大类`这个名词来区分`短袜`和`手套`这两种不同的产品。每个具体工厂都会生产这两大类产品，只不过每个工厂都会生产大类下面的具体产品。

`Client`仅针对`AbstractFactory`、`AbstractProductA`和`AbstractProductB`编程，而对任何具体类型透明。`Client`真正使用的具体工厂在运行时确定，这点会在后面的`Java`程序实现中体现出来。正如你所看到的，这个模式的一个优点就是`Client`是与具体`Product`解耦的。如果想要添加一个新的产品族，则需要添加一个实现了`AbstractFactory`接口的`ConcreteFactory`，同时还需要创建与该具体工厂对应的具体产品。

从`Client`的角度来看，我仅针对接口编程，因此可以保证自身代码不依赖于具体类型，与具体类型解耦。

### 抽象工厂模式用在哪里？

如果你的系统中需要创建不同的产品族，则可以使用抽象工厂模式。比如你想让你的UI家族（windows, buttons, textfields等等）支持不同的操作系统（跨平台），那么使用抽象工厂模式可以使你的客户端代码与具体的平台解耦。现在我们使用`Java`程序实现一个生成window的例子：

首先创建`Window`接口，这里`Window`是一种`AbstractProduct`：

```java
// Our AbstractProductA
public interface Window {
  public void setTitle(String text);
  public void repaint();
}
```

接着创建`Window`的两种不同的具体实现，作为`ConcreteProduct`：

```java
// ProductA1 微软Windows系统的窗口部件
public class MSWindow implements Window {
  public void setTitle() {
    // MS Windows specific behaviour
  }
  public void repaint() {
    // MS Windows specific behaviour
  } 
}
```

```java
// ProductA2 苹果OSX系统的窗口部件
public class MacOSXWindow implements Window {
  public void setTitle() {
    // Mac OSX specific behaviour
  }
  public void repaint() {
    // Mac OSX specific behaviour
  }
}
```

现在我们来实现工厂。首先定义`AbstractFactory`，这里我们仅要求工厂能够生产`window(窗口)`，当然也可以生成其他大类的东西，不过我们简化处理：

```java
// AbstractFactory
public class AbstractWidgetFactory {
  public Window createWindow();
  // ... of course we can create other types
}
```

接着我们针对微软和苹果两个操作系统实现两座具体工厂：

```java
// ConcreteFactory1
public class MSWindowsWidgetFactory {
  public Window createWindow() {
    MSWindow window = new MSWindow();
    return window;
  }
}
```

```java
// ConcreteFactory2
public class MacOSXWidgetFactory {
  public Window createWindow() {
    MacOSXWindow window = new MacOSXWindow();
    return window;
  }
}
```

最后我们针对接口编程，实现`Client`程序：

```java
// Client
public class GUIBuilder {
  public void buildWindow(AbstractWidgetFactory widgetFactory) {
    Window window = widgetFactory.createWindow();
    window.setTitle("New Window");
  }
}
```

`Client`真正使用的具体工厂在运行时确定：

```java
public class Main {
  public static void main(String[] args) {
    GUIBuilder builder = new GUIBuilder();
    AbstractWidgetFactory widgetFactory = null;
    // check what platform we're on
    if (Platform.currentPlatform() == "MACOSX") {
      widgetFactory = new MacOSXWidgetFactory();
    }
    else {
      widgetFactory = new MSWindowWidgetFactory();
    }
    builder.buildWindow(widgetFactory);
  }
}
```

这里给出本实现的抽象工厂模式的类图：

![abstract_factory_pattern_java](/assets/abstract_factory_pattern_java.png)

### 抽象工厂模式的缺点

该模式将`Client`和具体实现解耦是其核心优点，但是有时候需要在抽象类（`AbstractFactory`, `AbstractProduct`）中新增属性，这样修改抽象类和`Client`代码不可避免。