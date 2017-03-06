---
title: Catch单元测试
date: 2016-08-29 16:55:38
toc: true
tags:
  - C++
  - 单元测试
categories:
  - DevTools 
---

最近在写C++的一些小程序, 在书上讲到测试驱动开发的方法, 也就是TDD(Test-Driven Development), 所以要打算掌握单元测试方法, 至少自己在写算法题的时候可以自己测试代码的健壮性, 从网上了解了一个C++的测试工具叫`Catch`, 看是来十分简单友好, 来学习一下

<!--more-->

### **关于Catch**

![](https://raw.githubusercontent.com/philsquared/Catch/master/catch-logo-small.png)

[Catch](https://github.com/philsquared/Catch) 友好到你只需要引入一个头文件就可以做单元测试

> `#include "catch.hpp"`

  - 查看`catch.hpp`[点击这里](https://raw.githubusercontent.com/philsquared/Catch/master/single_include/catch.hpp)
  
### **阶乘例子**

假如我们写了一个求阶乘的函数, 我们要测试这个函数的输出结果, 代码如下:

```
#define CATCH_CONFIG_MAIN  // This tells Catch to provide a main() - only do this in one cpp file
#include "catch.hpp"

unsigned int Factorial( unsigned int number ) {
    return number <= 1 ? number : Factorial(number-1)*number;
}

TEST_CASE( "Factorials are computed", "[factorial]" ) {
    REQUIRE( Factorial(1) == 1 );
    REQUIRE( Factorial(2) == 2 );
    REQUIRE( Factorial(3) == 6 );
    REQUIRE( Factorial(10) == 3628800 );
}
```

运行结果:

`All tests passed (4 assertions in 1 test case)`

- 在一个单元测试中的4个`REQUIRE`测试结果都正确

但是我们知道`0`的阶乘为`1`, 我们添加一项`REQUIRE( Factorial(0) == 1 );`, 则运行结果为:
```
FAILED:
  REQUIRE( Factorial(0) == 1 )
with expansion:
  0 == 1
```

- 表示我们实际上得到的计算值为`0`, 但是我们希望运行结果为`1`, 这就出现了错误

我们修改函数为

```
unsigned int Factorial( unsigned int number ) {
  return number > 1 ? Factorial(number-1)*number : 1;
}
```

现在测试就全部通过了

假如我们添加这样一个测试`REQUIRE(Factorial(15) == 1307674368000);`, 运行结果是

```
FAILED:
  REQUIRE( Factorial(15) == 1307674368000 )
with expansion:
  2004310016 (0x77775800) == 1307674368000
```

我们知道`unsigned int`的范围是`0~4294967295(4 Bytes)`, 我们的计算结果`1307674368000`超出了`unsigned int`的范围, 所以报错, 我们将函数返回类型改为`long long`, 代码如下:

```
#define CATCH_CONFIG_MAIN  // This tells Catch to provide a main() - only do this in one cpp file
#include "catch.hpp"

long long Factorial(unsigned int number) {
	return number > 1 ? Factorial(number - 1)*number : 1;
}

TEST_CASE("Factorials are computed", "[factorial]") {
	REQUIRE(Factorial(0) == 1);
	REQUIRE(Factorial(1) == 1);
	REQUIRE(Factorial(2) == 2);
	REQUIRE(Factorial(3) == 6);
	REQUIRE(Factorial(10) == 3628800);
	REQUIRE(Factorial(15) == 1307674368000);
}
```

现在测试就全部通过了

### **TDD风格**

```
#define CATCH_CONFIG_MAIN  // This tells Catch to provide a main() - only do this in one cpp file
#include "catch.hpp"

TEST_CASE( "vectors can be sized and resized", "[vector]" ) {

    std::vector<int> v( 5 );

    REQUIRE( v.size() == 5 );
    REQUIRE( v.capacity() >= 5 );

    SECTION( "resizing bigger changes size and capacity" ) {
        v.resize( 10 );

        REQUIRE( v.size() == 10 );
        REQUIRE( v.capacity() >= 10 );
    }
    SECTION( "resizing smaller changes size but not capacity" ) {
        v.resize( 0 );

        REQUIRE( v.size() == 0 );
        REQUIRE( v.capacity() >= 5 );
    }
    SECTION( "reserving bigger changes capacity but not size" ) {
        v.reserve( 10 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 10 );
    }
    SECTION( "reserving smaller does not change size or capacity" ) {
        v.reserve( 0 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 5 );
    }
}
```

上面是`TDD(测试驱动开发)`的单元测试代码, `resize(n)`函数重新分配大小，改变vector的大小，并且创建对象, `reserve(n)`函数用来给vector预分配存储区大小，即capacity的值($capacity>=n$) ，但是没有给这段内存进行初始化, 其中

1. `#define CATCH_CONFIG_MAIN`提供了默认的` main `函数入口去测试代码

2. 在一个测试单元`TEST_CASE`中包含4个部分`SECTION`和若干个`REQUIRE`

3. `REQUIRE`中包含我们待测试的结果

- `TEST_CASE`中有 4 个` SECTION`，它们并不是单纯的顺序执行关系, 在`第一个 SECTION `执行完成之后, 会重头开始执行并`跳过已经执行过的 SECTION`, 也就是说上面的代码去掉了` SECTION 宏`之后的执行路径大概是这样的 ：

```
// SECTION 1
std::vector<int> v( 5 );
REQUIRE( v.size() == 5 );
REQUIRE( v.capacity() >= 5 );
v.resize( 10 );
REQUIRE( v.size() == 10 );
REQUIRE( v.capacity() >= 10 );

// SECTION 2
std::vector<int> v( 5 );
REQUIRE( v.size() == 5 );
REQUIRE( v.capacity() >= 5 );
v.resize( 0 );
REQUIRE( v.size() == 0 );
REQUIRE( v.capacity() >= 5 );

// SECTION 3
std::vector<int> v( 5 );
REQUIRE( v.size() == 5 );
REQUIRE( v.capacity() >= 5 );
v.reserve( 10 );
REQUIRE( v.size() == 5 );
REQUIRE( v.capacity() >= 10 );

// SECTION 4
std::vector<int> v( 5 );
REQUIRE( v.size() == 5 );
REQUIRE( v.capacity() >= 5 );
v.reserve( 0 );
REQUIRE( v.size() == 5 );
REQUIRE( v.capacity() >= 5 );

```

### **BDD风格**

```
#define CATCH_CONFIG_MAIN  // This tells Catch to provide a main() - only do this in one cpp file
#include "catch.hpp"

SCENARIO( "vectors can be sized and resized", "[vector]" ) {

    GIVEN( "A vector with some items" ) {
       
        std::vector<int> v( 5 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 5 );

        WHEN( "the size is increased" ) {
            v.resize( 10 );

            THEN( "the size and capacity change" ) {
                REQUIRE( v.size() == 10 );
                REQUIRE( v.capacity() >= 10 );
            }
        }
        WHEN( "the size is reduced" ) {
            v.resize( 0 );

            THEN( "the size changes but not capacity" ) {
                REQUIRE( v.size() == 0 );
                REQUIRE( v.capacity() >= 5 );
            }
        }
        WHEN( "more capacity is reserved" ) {
            v.reserve( 10 );

            THEN( "the capacity changes but not the size" ) {
                REQUIRE( v.size() == 5 );
                REQUIRE( v.capacity() >= 10 );
            }
        }
        WHEN( "less capacity is reserved" ) {
            v.reserve( 0 );

            THEN( "neither size nor capacity are changed" ) {
                REQUIRE( v.size() == 5 );
                REQUIRE( v.capacity() >= 5 );
            }
        }
    }
}
```

上面是`BDD(行为驱动开发)`的单元测试代码, BDD(Behaviour-Driven Development)是最新的一种测试方式，它强调的是`行为`而不是`测试`, 实际上这两种方式是等价，`SCENARIO` 只是` TEST_CASE `的别名，`GIVEN、WHEN、THEN` 最终也是 map 到 `SECTION` 上面的，这其中的差异只是存在于测试的思维不同而已

### **注意事项**

```
#define CATCH_CONFIG_MAIN  // This tells Catch to provide a main() - only do this in one cpp file
#include "catch.hpp"
```

最好将以上两行代码只放在源文件中的一个头文件中, 这个头文件只包含`#define`和`#include`, 把测试代码写在其他的文件中, 其他文件只要引入`#include "catch.hpp"`就可以了, 不需要`CATCH_CONFIG_MAIN or CATCH_CONFIG_RUNNER`, 之所以这样做是因为Catch是单头文件库，这意味着它里面的内容会最终出现在所有的包含这个头文件的编译单元中。如果我们把测试代码和上面两行代码放在一起会导致每次编译测试代码的时候都需要编译 Catch 的内核，这会导致编译速度非常非常的慢。如果把两者分开，Catch 的内核只需要在一个文件中编译一次，因为 Catch 内部做了判断，即使你在多个文件中使用了 #include "catch.hpp"，如果内核编译过了是不需要再次编译的，这个文件的编译速度相对较慢，但是这个文件不需要改动，所以整个开发周期中它只需要编译一次，对于不断更新的测试代码编译速度就会快很多。

> [Catch官方文档](https://github.com/philsquared/Catch/blob/master/docs/Readme.md)








