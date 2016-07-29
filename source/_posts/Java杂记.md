---
title: Java杂记
tags:
  - Java
  - 更新中
date: 2016-07-14 17:34:55
---

这篇文章用来记录学习Java过程中的理解和感触,最近负责用Java写一个文本提取的模块，在这个过程中用到了很多jar包，体会到了java作为一个成熟的工业级语言的强大和开发效率的优势，于是准备好好学习一下这门编程语言,学习书籍为`Java编程思想`。

<!--more-->

## 基本知识

### 什么是静态？

在我看来任何的修饰符都是根据特定需求而出现的, 那么`static`的出现解决了什么样的需求呢?

1. 只想为特定数据或者对象分配单一的存储空间, 而不去考虑究竟要创建多少对象
2. 在Java中, 方法必须包含在类中, 希望某一方法独立于任何类, 也就是说不用创建对象, 也能调用这个方法
 

- 在用`static修饰的方法`不需要创建任何对象, 而且即使创建多个对象, 它们共享同一块存储空间, 这块存储空间可以称为`static域`

- 引用static变量, 类名直接引用`ClassName.method()`或者对象访问成员`Object.method()` ,`类名直接引用`是特有的而且优先选择这种方式

- 对于`main()`方法而言, 作为程序的入口, 需要不创建任何对象就可以调用, 所以必须定义为`static`类型

### 别名问题

- 对于赋值操作, 当对象进行赋值的时候, 实际上是对对象的引用, 也就是说将左值指向右值的堆内存地址, 相当于给右边的对象起了一个别名, 它们实际上操作的是同一块内存空间

- 对象在方法中的传递, 也是传递了一个引用, 实际上操作的是传递的真实对象, 换句话说, 方法操作的是被调用方法外面的对象.

### 短路问题

在Java的逻辑运算中, 一旦可以准确无误的确定整个表达式的值, 就不再进行运算了, 比如与运算中有`false`

### 类型转换

`窄化转换`也就是说能容纳更多的数据类型转换为容纳信息更少的数据类型, 这是十分危险的事情, 必须`显示`进行转换, 一般来说表达式中出现的最大的数据类型决定了表达式最终结果的类型

- 截尾: 在将`float`或者`double`转为`int`时, 总是进行截尾

```java
public class CastingNumbers {
  public static void main(String[] args) {
    double above = 0.7, below = 0.4;
    float fabove = 0.7f, fbelow = 0.4f;
    print("(int)above: " + (int)above);
    print("(int)below: " + (int)below);
    print("(int)fabove: " + (int)fabove);
    print("(int)fbelow: " + (int)fbelow);
  }
} 
/* Output:
(int)above: 0
(int)below: 0
(int)fabove: 0
(int)fbelow: 0
*/

```

- 四舍五入: `java.lang.Math`中的round()方法实现

```java
public class RoundingNumbers {
  public static void main(String[] args) {
    double above = 0.7, below = 0.4;
    float fabove = 0.7f, fbelow = 0.4f;
    print("Math.round(above): " + Math.round(above));
    print("Math.round(below): " + Math.round(below));
    print("Math.round(fabove): " + Math.round(fabove));
    print("Math.round(fbelow): " + Math.round(fbelow));
  }
} 
/* Output:
Math.round(above): 1
Math.round(below): 0
Math.round(fabove): 1
Math.round(fbelow): 0
*/

```

### Javadoc:API文档

```
javadoc -d [路径] -author -version

/**
@author Simshang
@version V0.1
*/

```

- 在生成javadoc的时候，该类必须被public所修饰，并且该类中的方法只有被public所修饰才可以在方法摘要列表中。

## 面向对象

### 类与对象
对于刚刚接触面向对象语言的新手来说，理解面向对象的概念就是一个门槛，感觉模模糊糊地似懂非懂，其实有时候读偏哲学的书的时候也会给你同样的感觉。世间万物都可以用两个维度来描述，一个是它本身所具有的属性，另一个是本身可以发生的动作，二者结合起来就是一个对象。我们把生物世界分成不同的物种，是因为寻找到了相同物种之间的共同之处，寻找的过程就是一个抽象的过程，举个例子，把不同品种的猫都叫做猫，猫就是一个类，而特定的某一品种的猫就是一个对象，因为他们有共同的属性可以发出相同的动作，而这个品种具体的一只活生生的猫就是这个对象（猫）的实例。对象一定是抽象出来的，而这种设计哲学更加的抽象出了现实世界。
- 类是一个抽象的概念, 而对象是通过`new`操作产生的抽象类的一个实例, 可以把对象理解为一个具象的概念.下面比较二者不同:

|  名称 |  概念  | 存放位置 |
|------|-----------|---------|
|  类  |  抽象  |  堆栈(对象引用)  |
|  对象  |  具象  |  堆内存(对象本身)  |

- 为了提高效率, Java的基本类型`int`,`char`,`boolen`等是存放在堆栈中的, 也可以通过包装器类`new`到堆内存中.

### 三大特性

> 所有面向对象的特性来源于两点:`效率`和`设计`

1. `封装`: 把数据和方法包装进类中 , 以及把具体实现的过程隐藏起来 , 成为一个同时带有特征和行为的数据类型

- 

2. `继承`:

3. `多态`:





