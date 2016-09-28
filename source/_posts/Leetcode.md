---
title: LeetCode
date: 2016-07-18 19:35:11
toc: true
tags:
  - LeetCode
categories:
  - 算法
---
问题分析:

- 画图 : 抽象问题形象化
- 举例 : 抽象问题具体化
- 分解 : 抽象问题简单化

<!--more-->

提高代码和算法能力的练习记录, 不仅仅只有Leetcode上的题, 还有`剑指offer`的读书笔记

### **LeetCode**

#### **String(字符串)**

[344. Reverse String](https://leetcode.com/problems/reverse-string/)
  代码实现 : [C++](https://github.com/Simshang/LeetcodeCpp/blob/master/LeetcodeCpp/ReverseString.cpp)
  - `双指针`实现
  - `栈`结构实现
  
[345. Reverse Vowels of a String](https://leetcode.com/problems/reverse-vowels-of-a-string/)
  代码实现 : 
  [C++](https://github.com/Simshang/LeetcodeCpp/blob/master/LeetcodeCpp/ReverseString.cpp) 
  [Java](https://github.com/Simshang/LeetCode/blob/master/src/ReverseString.java)
  


### **参考资料**

- [九章算法](http://www.jiuzhang.com/solutions/)

- [细语呢喃Leetcode算法题解](https://www.hrwhisper.me/leetcode-algorithm-solution/)

### **剑指offer**

#### **效率优化**

- **优化菲波那切数列**

```
int fibonacci(int n) {
    if (n == 1)
    return 0;
    if (n == 2)
    return 1;
    return fibonacci(n-1) + fibonacci(n-2);
}
```

- 通过递归调用树分析上面的代码的时间复杂度为$T(n)=T(n-1)+T(n-2)+O(1)= O(1.618 ^ n)$, 空间复杂度取决于递归的深度是$O(n)$, `总耗时:1007ms`

用`递归法`实现的代码包含着大量的重复计算, 如下图所示

![](\img\Leetcode\fibonacci.jpg)

我们在计算$F(5)$和$F(4)$的时候包含了大量的$F(3)$的重复计算, 所以我们采用`递推法`从$F(1)$开始计算, 代码如下

```c++
int fibonacci(int x){
    if (x == 1)
        return 0;
    if (x == 2)
        return 1;
    else
    {
        int p1 = 0;
        int p2 = 1;
        int result = 0;
        while (x > 2)
        {
            result = p1 + p2;
            p1 = p2;
            p2 = result;
            --x;
        }
    return result;
    }
}
```

- 时间复杂度是O(n)，空间复杂度是O(1), `总耗时 : 19 ms`              

