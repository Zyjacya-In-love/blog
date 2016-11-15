---
title: BigO算法分析
date: 2016-07-05 09:31:07
toc: true
tags:
  - BigO
categories:
  - 编程之美
---

`算法`是求解一个问题需要遵循的并且被清楚的指定的简单指令的集合, 当确定算法的正确性之后, 首当其冲的是分析该算法所需的CPU计算时间和消耗的内存空间, 在评估算法的时候有很多种方法, 这篇文章介绍了如何用`Big-O`大法进行算法复杂度的分析

<!--more-->

### **时间复杂度和空间复杂度**

一般情况下，算法中基本操作重复执行的次数是问题规模$n$的某个函数，用$T(n)$表示

- **不同算法复杂度的运行时间**

![](/img/BigO算法分析/math.gif)

- 横坐标$n$代表输入的规模, 纵坐标$T(n)$代表`运行时间`,即`算法复杂度`

- 上图表示了线性复杂度$O(n)$, 平方复杂度$O(n^2)$, 立方复杂度$O(n^3)$, 对数复杂度$O(log(n))$, 指数复杂度$O(2^n)$, 开方复杂度$O(\sqrt{n})$, 线性复杂度 $O(n)$ 表示每个元素都要被处理$1$次; 平方复杂度 $O(n2)$ 表示每个元素都要被处理 $n$ 次, 以此类推

### **常用算法复杂度检索**

- 最坏情况（Worst）：任意输入规模的最大运行时间。（Usually）
- 平均情况（Average）：任意输入规模的期待运行时间。（Sometimes）
- 最佳情况（Best）：通常最佳情况不会出现。（Bogus）

![](/img/BigO算法分析/bigO1.JPG)

![](/img/BigO算法分析/bigO2.JPG)

![](/img/BigO算法分析/bigO3.JPG)

![](/img/BigO算法分析/bigo.png)

> 引自[bigocheatsheet](http://bigocheatsheet.com/) 


### **BigO计算运行时间步骤**

1. 确定决定算法运行时间的组成步骤
2. 找到执行该步骤的代码，标记为` 1`
3. 查看标记为` 1` 的代码的下一行代码，如果下一行代码是一个循环，则将标记` 1 `修改为循环的次数 $1 * n$。如果包含多个嵌套的循环，则将继续计算倍数，例如$ 1 * n * m$
4. 找到标记到的最大的值，就是运行时间的最大值，即算法复杂度描述的上界

### **循环复杂度分析**

#### $O(1)$ : 

- 一个函数调用或是一组语句(expressions)都认为是O(1)的复杂度, 该函数调用不包含循环，递归或其他非常量复杂度的函数, 比如调用的函数`swap()`是$O(1)$的时间复杂度
  
#### $O(n)$ :


```c++
for (int i = 1; i <= n; i += c){
    // some O(1) expressions
}

for (int i = n; i > 0; i -= c){
    // some O(1) expressions
}
```

- 如果在一个大小为n循环中，循环变量按照一个`常量C递增或递减`，这个for循环的复杂度就为$O(n)$
- c为常数

#### $O(n^2)$

```c++
for (int i = 1; i <=n; i += c) {
      for (int j = 1; j <=n; j += c) {
         // some O(1) expressions
      }
  }

  for (int i = n; i > 0; i += c) {
      for (int j = i+1; j <=n; j += c) {
         // some O(1) expressions
  }
```

- 嵌套循环的时间复杂度等于`最内层语句`执行的次数

#### $O(log(n))$

```c++
for (int i = 1; i <=n; i *= c) {
    // some O(1) expressions
  }
  
for (int i = n; i > 0; i /= c) {
    // some O(1) expressions
}

```

- 如果在一个大小为n循环中，循环变量按照一个`常量C的进行倍增或倍减`，这个循环的复杂度就为$O(log(n))$

#### $O(log(log(n)))$

```c++
for (int i = 2; i <=n; i = pow(i, c)) { 
  // some O(1) expressions
}

for (int i = n; i > 0; i = fun(i)) { 
  // some O(1) expressions
}
```

- 如果在一个大小为n循环中，循环变量是指数级的递增或递减，这个循环的复杂度就为$O(log(log(n)))$
- 这里的` fun 函数`可以是`sqrt `或` cuberoot `或任何其他恒定的根
- `pow(i,c)`以i为底的c次方, 其中c为比1大的常量 

**计算多个连续循环的复杂性**

```c++
for (int i = 1; i <=m; i += c) {  
     // some O(1) expressions
}
for (int i = 1; i <=n; i += c) {
     // some O(1) expressions
}
```

- 时间复杂度$ O(m) + O(n) = O(m+n)$, 如果$ m == n $就是$ O(2n)= O(n)$

**如果循环中有许多if …else, 一般情况我们只考虑最快情况下的复杂度**

> 参考 [Analysis of Algorithms | Set 4 (Analysis of Loops)](http://www.geeksforgeeks.org/analysis-of-algorithms-set-4-analysis-of-loops/)

### **递归复杂度分析**

在归并排序中，对一个给定的数组进行排序，我们把它分成两半，并对这两半递归地重复这个过程，最后我们合并结果。时间复杂度可以写成：

$$T(n) = 2T(n/2) + cn$$

#### **数学归纳法**

对于归并排序, 我们假设$T(n) = O(nlog(n))$, 然后用数学归纳法来证明我们的假设, 只需证明$T(n) <= cnlog(n)$, 证明过程如下

$$
T(n) = 2T(n/2) + n
    <= (cn/2)log(n/2) + n
    =  cnlog(n) - cnlog(2) + n
    =  cnlog(n) - cn + n
    <= cnlog(n)
$$

#### **递归树方法**

首先我们画出一个递归调用树, 并计算该树的每一层的时间复杂度, 然将每一次的复杂度相加就得到递归算法的总复杂度, 相加的式子一般是`典型的等差或等比级数`

![](/img/BigO算法分析/re.png)

#### **Master Method**

> 参考 [Analysis of Algorithm | Set 4 (Solving Recurrences)](http://www.geeksforgeeks.org/analysis-algorithm-set-4-master-method-solving-recurrences/)

### 多项式函数

![](/img/BigO算法分析/math1.gif)

- 对于多项式函数而言, 其基本的形状是由最高次项决定的

![](/img/BigO算法分析/math5.gif)
![](/img/BigO算法分析/math6.gif)
![](/img/BigO算法分析/math7.gif)

- 由图得知, $n^2+100n+500$ 和 $n^2$, 当$n$趋近于无穷时, 高次项决定了函数的形状, 而低次项几乎对函数的形状没有影响, 二者几乎一致, 所以$Big-O$估计的是算法渐进的值, 即$O(n^2+100n+500)=O(n^2)$



### 线性函数

![](/img/BigO算法分析/math2.gif)
![](/img/BigO算法分析/math3.gif)
![](/img/BigO算法分析/math4.gif)

- 对于以上三种函数而言, 斜率并不影响函数的形状, 仅仅影响函数的陡峭程度, $O(kn)=O(n)$, k为常数

### 对数函数

![](/img/BigO算法分析/sorting.gif)

- 并归排序是最好的排序算法, 其运行时间是$n{log(n)}$

- 冒泡排序, 选择排序, 插入排序较慢, 运行时间都是$O(n^2)$

### 多项式和对数

![](/img/BigO算法分析/math8.gif)
![](/img/BigO算法分析/math9.gif)

- 当$n$足够大时, 多项式函数始终会超过对数函数
- 但是在特定的输入规模内, 多项式函数是低于对数函数的, 所以每种算法都有特定的适用范围

### `对数`和`开方`


![](/img/BigO算法分析/math10.gif)
![](/img/BigO算法分析/math11.gif)
![](/img/BigO算法分析/math12.gif)
![](/img/BigO算法分析/math13.gif)
 
 


> 引自[Sarah Lawrence College](http://science.slc.edu/~jmarshall/courses/2002/spring/cs50/BigO/)

