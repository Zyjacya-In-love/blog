---
title: Java编程笔记
toc: true
tags:
  - Java
categories:
  - 笔记
date: 2016-07-14 17:34:55
---

这篇文章用来记录学习Java过程中的理解和感触,最近负责用Java写一个文本提取的模块，在这个过程中用到了很多jar包，体会到了java作为一个成熟的工业级语言的强大和开发效率的优势，于是准备好好学习一下这门编程语言

> 参考书 : [Java编程思想（第4版）](https://book.douban.com/subject/2130190/)

<!--more-->


## 基本知识

### 静态

在我看来任何的修饰符都是根据特定需求而出现的, 那么`static`的出现解决了什么样的需求呢?

1. 只想为特定数据或者对象分配单一的存储空间, 而不去考虑究竟要创建多少对象
2. 在Java中, 方法必须包含在类中, 希望某一方法独立于任何类, 也就是说不用创建对象, 也能调用这个方法
 

- 在用`static修饰的方法`不需要创建任何对象, 而且即使创建多个对象, 它们共享同一块存储空间, 这块存储空间可以称为`static域`

- 引用static变量, 类名直接引用`ClassName.method()`或者对象访问成员`Object.method()` ,`类名直接引用`是特有的而且优先选择这种方式

- 对于`main()`方法而言, 作为程序的入口, 需要不创建任何对象就可以调用, 所以必须定义为`static`类型

> 通过 _**final** **static**_ 修饰的容器类型变量中所“装”的对象是可改变的, `final static`可以理解为全局常量

### 引用

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

   
### **我的程序**

用Java写了一个ftp下载的工具, 见[ftpDownloader-Github](https://github.com/Simshang/ftpDownloader)
   
### **Java Error Solution**

- 记录在Java开发过程中遇到的错误以及解决办法

> Error:(20, 20) java: 从内部类中访问本地变量args; 需要被声明为final类型

- 将相应的变量被`final`修饰符修饰 

- 解释:
  首先`final`修饰符,可以修饰类/方法/变量,告诉编译器这一块数据是固定不变的.我们要知道内部类编译完之后是生成一个独立的.class文件, 那么其实问题就转化于怎么从一个类去访问另一个类的方法的局部变量, 在正常的情况下这显然是不可能的, 但是如果是局部内部类的话，你在局部变量上加上final修饰符，会在这个局部内部类所生成的.class文件的常量池中加入这个局部变量,所以就能访问到了。