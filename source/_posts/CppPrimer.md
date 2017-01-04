---
title: C++ Primer
date: 2016-08-18 19:06:36
toc: true
tags:
  - Cpp
categories:
  - 笔记
---
> 参考书 : [C++ Primer 中文版（第5版）](https://book.douban.com/subject/25708312/)
<!--more-->
> [C++开发文档](http://www.cplusplus.com/)

### **g++的安装与编译**

1. 下载源码

   `wget ftp://ftp.gnu.org/gnu/gcc/gcc-4.8.5/gcc-4.8.5.tar.gz`
   
2. 下载依赖包

   - 编译安装 GCC 需要依赖 mpc，mpfr，gmp包。好在 GCC 源码里自带脚本可以轻松下载依赖包。

   `tar zxf gcc-4.8.5.tar.gz`
   `cd gcc-4.8.5`
   `./contrib/download_prerequisites`
   - 用vim查看文件, 在此脚本里可以看到依赖包的版本号依次是 mpc-0.8.1，mpfr-2.4.2，gmp-4.3.2。

3. 编译安装

   `mkdir gcc-build-4.8.5`
   `cd gcc-build-4.8.5`
   `../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib`
   `make && make install`
   
   - 为了避免安装后系统里出现多个版本的 GCC，这里直接将编译安装的目录指定为 /usr，如果不指定 –prefix，则会默认安装到 /usr/local 下。
   - GCC 4.8.5 光是源代码就有105MB，因此可以预见整个编译过程需要很长时间（差不多 2 个小时左右）。

4. 查看版本号

   `gcc --version`
   gcc (GCC) 4.8.5

   `which gcc`
   /usr/bin/gcc

   `which g++`
   /usr/bin/g++
   
**支持C++11特性**

如果用命令 g++ -g -Wall main.cpp  编译以下代码 ：

```
/*
    file : main.cpp
*/
#include <stdio.h>
 
int main() {
    int a[5] = { 1, 2, 2, 5, 1 };
    for( int i:a ) {
        printf( "%d\n", a[i] );
    }
    return 0;
}
```

提示错误:

```
main.cpp: In function ‘int main()’:
main.cpp:5:13: error: range-based ‘for’ loops are not allowed in C++98 mode
  for( int i:a ) {
```

在C++98中不支持此循环方式，因为这是C++11新增的循环方式。

那么如果一定要编译呢？

通过命令man g++可以得知以下方法：

`g++ -g -Wall -std=c++11 main.cpp`

除了g++ , gcc 也可以类似方法支持C11

`gcc -g -Wall -std=c11 main.cpp`

生成可执行文件:

`g++ -std=c++11 -o main main.cpp`

### **编译源代码**

`gcc `和` g++`分别是GNU的C和C++的编译器, 我们拿`hello world`来举例子, 

`g++指令的一般格式为：g++ [选项] 要编译的文件 [选项] [目标文件]`

```c++
#include<iostream>
using namespace std;

int main (void)
{
    cout<<"hello world"<<endl;
    return 0;
}

```

1. 预处理器对`.cpp`文件进行预处理生成`.i`文件, 预处理做了宏的替换，还有注释的消除，可以理解为无关代码的清除

   `g++ -E hello.cpp > hello.i`
   
   - 如果不重定向, 只激活预处理,这个命令不生成文件, 你需要把它重定向到一个输出文件里`hello.i`
   
   `vim hello.i`打开生成的文件可以看到头文件被替换

2. 编译器对`.cpp`文件进行编译生成包含汇编指令的`.s`文件

  `g++ -S hello.cpp`
  
  - `.s`文件表示是汇编文件，用Vim编辑器打开就都是汇编指令
  
3. 汇编器对汇编指令变为目标代码(机器代码), 即将`.s`文件生成`.o`文件

  `g++ -c hello.s`
  
4. 链接器将目标代码链接起来生成可执行文件

   `g++ hello.o -o hello`
   
   - 注意, 以上命令可以从源文件`.cpp`一次性编译, `g++ -o hello hello.cpp` : 将.cpp文件直接编译成名为`hello`的可执行文件
   - 在Linux系统中, 生成名为`hello`的没有后缀的可执行文件, 用`./hello`执行;如果没有`-o hello`参数, 会生成名为`a.out`的文件, `./a.out`执行
   - Windows系统中生成`.exe`文件

### **编译指令**

`#空指令`
  - 无任何效果
  
`#include`
  - 包含一个源代码文件
`#define`
  - 定义宏
`#undef`
  - 取消已定义的宏
`#if`
  - 如果给定条件为真，则编译下面代码
`#ifdef`
  - 如果宏已经定义，则编译下面代码
`#ifndef`
  - 如果宏没有定义，则编译下面代码
`#elif`
  - 如果前面的`#if`等条件不为真，当前条件为真，则编译下面代码, 相当于`elseif`
`#endif`
  - 结束一个`#if……#else`条件编译块
`#error`
  - 停止编译并显示错误信息


**常用参数**
  
- `-E`参数 : 指示编译器仅对输入文件进行预处理。当这个选项被使用时, 预处理器的输出被送到标准输出而不是储存在文件里, 所以需要重定向保存为`.i`文件
- `-S`参数 : 指示编译器对`.cpp`文件产生汇编语言文件后停止编译, 产生的汇编语言文件的缺省扩展名是`.s`
- `-c`参数 : 指示编译器仅把源代码编译为目标代码(`.o`文件)
- `-o`参数 : 指示编译器为将产生的可执行文件用指定的文件名, Linux下没有后缀

### **建议**

1. 在 C++中几乎不需要用宏，用 `const` 或 `enum` 定义显式的常量，用 `inline` 避免函数调用的额外开销，用模板去刻画一族函数或类型，用 `namespace` 去避免命名冲突。
2. 不要在你需要变量之前去声明，以保证你能立即对它进行初始化。
3. 不要用 `malloc`,`new` 运算会做的更好。
4. 避免使用 `void*`、指针算术、联合和强制，大多数情况下，强制都是设计错误的指示器。
5. 尽量少用`数组`和` C `风格的字符串，标准库中的` string` 和 `vector `可以简化程序。
6. 更加重要的是，试着将程序考虑为一组由`类和对象`表示的相互作用的概念，而不是一堆数据结构和一些可以拨弄的二进制。

### **文件重定向**

在`C++ Primer`的$P_{19}$页

- 在编译好的`.exe`文件拷贝到`数据`目录下, 执行`$ EXEFilename <infile>output.txt`

`./a.out <./data/word_echo>a`

- `可执行文件a.out`在当前目录
- 将`word_echo`文件的内容作为可执行文件`a.out`的输入
- 可执行文件`a.out`的输出重定向到`a文件`里

### **函数重载的底层实现**

> `Name Mangling`是一种在编译过程中，将函数、变量的名称重新改编的机制，简单来说就是编译器为了区分各个函数，将函数通过一定算法，重新修饰为一个全局唯一的名称。

- `C++`利用name mangling(倾轧)技术将函数重命名, 区分参数不同的同名函数

- 倾轧技术发生在`.cpp`文件的编译阶段, 即将编译输出文件中的函数根据一定的规则重命名; 和`.h`文件的声明阶段, 在头文件中声明的函数也要重命名, 很显然, 如果只有其中的一个阶段发生倾轧, 那么在链接时就会出现函数未定义的错误

- 注意:C++完全兼容C语言，那就面临着，`完全兼容C的类库`。由`.c 文件`的类库文件中函数名并没有发生name mangling 行为，而我们在`.cpp文件中`包含`.c 文件所对应的.h 文件`时，`.h 文件`要发生name manling 行为，因而会发生在`链接的时候`的错误。
- C++为了避免上述错误的发生，重载了关键字` extern`。要避免 name manling的函数前，加 `extern "C"` 如有多个函数声明要避免，则放入` extern "C"{}`
     
### **引用**

- 需求 :　进行参数的传递
- 本质 :　引用是个特殊的指针, 对指针的包装，扩展了指针的作用域，　不会暴露地址，　也不必开辟地址空间

- 变量名，本身是一段内存的引用, 是为己有变量起一个别名
- 引用没有定义，是一种关系型声明, 类型与原类型保持一致，且不分配内存, 被引用的变量有相同的地址
- 必须初始化, 一经初始化, 该引用不可变更
- 可以对引用再次建立引用，也就是某一变量可以具有多个别名, 但是不能破坏原有的引用关系
- 重载`&`符号:前面有数据类型时是引用, 其它皆为取地址
- 可以定义指针的引用`int* &ref`, 不能定义引用的引用`int&& ref`
- 可以定义指针的指针`int** val`, 不能定义引用的指针`int& *ref`
- 可以定义数组的引用`int (&arr)[3]=数组名`, 不能定义引用数组`int& refArr[] = {}`,但是可以定义指针数组`int* arr[]={存放指针变量}`
- 当`const`修饰的时候, 引用就开辟了一块未命名的空间

> 一个验证引用的例子:

```
#include <iostream>
using std::cout; using std::endl;

#include <string>
using std::string;

#include <vector>
using std::vector;

#include <iterator>
using std::begin; 
using std::end;  

void printVec(vector<int>& vec)
{
	for (auto index = vec.begin(); index != vec.end(); ++index)
		cout << *index << " ";
}

int main()
{
	vector<int> val = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
	for (auto v : val)
	{
		v *= v;
		cout << v << endl;
	}
	printVec(val);
	for (auto &v : val)
	{
		v *= v;
		cout << v << endl;
	}
	printVec(val);
}
```

### **左值**

`左值`指的是可以取地址的变量，记住: 左值与右值的根本区别在于能否获取内存地址，而能否赋值不是区分的依据。

### **类**

- 本质:类名本质上就是一个命名空间

> 在C++中`class` 和`struct` 的唯一区别 : class的访问权限是private的, 而struct是public的; 但是在C语言中, class只是一些数据的集合, 没有C++中类的特性

### **迭代器**

> [C++迭代器 iterator](http://www.cnblogs.com/yc_sunniwell/archive/2010/06/25/1764934.html)

- 容器是数据结构的泛指，迭代器是指针的泛指, C++迭代器`Interator`就是一个指向某种`STL对象的指针`, 是一种检查容器内元素并遍历元素的`数据类型`

- `指针`也是迭代器


### **const**

> [关键字：Const，Const函数，Const变量，函数后面的Const](http://www.cnblogs.com/Fancyboy2004/archive/2008/12/23/1360810.html)

**摘要**

- 对于非内部数据类型的输入参数，应该将`值传递`的方式改为`const 引用传递`，目的是提高效率。例如将`void Func(A a) `改为`void Func(const A &a)`

- 对于内部数据类型的输入参数，`不要将“值传递”的方式改为“const 引用传递”`, 否则既达不到提高效率的目的，又降低了函数的可理解性。例如`void Func(int x)` `不应该`改为`void Func(const int &x)`

- 如果给以“指针传递”方式的函数返回值加const 修饰，那么函数返回值（即指针）的内容不能被修改，该返回值只能被赋给加const 修饰的同类型指针

- 如果函数返回值采用“值传递方式”，由于函数会把返回值复制到外部临时的存储单元中，加const 修饰没有任何价值。例如不要把函数int GetInt(void) 写成const int GetInt(void)

- 任何`不会修改数据成员的函数`都应该声明为const 类型, 例如`int GetCount(void) const; // const 成员函数`, 如果在编写const 成员函数时，不慎修改了数据成员，或者调用了其它非const 成员函数，编译器将指出错误

**关于Const函数的几点规则：**

- const对象只能访问const成员函数, 而非const对象可以访问任意的成员函数,包括const成员函数
- const对象的成员是不可修改的, 然而const对象通过指针维护的对象却是可以修改的
- const成员函数不可以修改对象的数据, 不管对象是否具有const性质, 它在编译时, 以是否修改成员数据为依据, 进行检查;
- 但是加上`mutable`修饰符的数据成员, 对于任何情况下通过任何手段都可修改, 自然此时的const成员函数是可以修改它的

### **常见问题**

- `VS2013窗口一闪而过` : `Ctrl+F5`运行程序

- 用`while`作为循环输入的时候, 用`Ctrl+Z`结束

- 设置一个断点, 然后点击`本地Windows调试器`

- 所谓的段错误`segmentation fault`就是指访问的内存超出了系统所给这个程序的内存空间
