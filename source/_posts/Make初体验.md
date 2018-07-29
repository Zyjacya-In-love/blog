---
title: Make 初体验
toc: true
tags:
  - Make
categories:
  - 计算机
date: 2018-07-29 15:05:21
---

最近在工程中常用到Makefile构建工具，总结一下常用的使用方法。

<!--more-->

### **Make**

使用Makefile维护的工程, 通常我们使用`make`命令去构建一个工程, 整个构建的过程通常包括`编译(Compile)`和`链接(Link)`两个过程, 对于C++工程来说, 通常编译是把源文件编译成中间代码文件，在Windows下也就是` .obj 文件`，UNIX下是` .o 文件`，即` Object File`, 最后通过链接的过程将大量的目标文件合成可执行文件. 

- 在编译时，编译器只检测程序语法，和函数、变量是否被声明。如果函数未被声明，编译器会给出一个警告，但可以生成Object File 

- 在链接程序时，链接器会在所有的Object File中找寻函数的实现，如果找不到，那到就会报链接错误码（Linker Error）

Makefile中定义了整个工程的构建规则, 所谓的规则就是描述了源代码中的依赖关系, 也就是如何去编译和链接程序

### **规则**

```bash
<target> : <prerequisites> 
[tab]  <commands>

```

- 上面第一行冒号前面的部分，叫做"目标"（target）

- 冒号后面的部分叫做"前置条件"（prerequisites）, 就是要生成target所需要的中间文件或是依赖的其他目标。

- 第二行必须由一个tab键起首，后面跟着make 可执行的"命令"（commands）

target是一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中, 当prerequisites中有一个以上的文件比target文件要新，command所定义的命令就会被执行

下面我们举个例子:

```bash
edit : main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o

main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c

clean :
    rm edit main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o

```

- `\`表示换行符, 为了更方便地进行Makefile的阅读

- 可执行文件`edit`是一个target, 是由`main.o`等中间的Object File通过链接而成的

- 对于`main.o`来说, 其本身也是一个target, 并依赖`main.c defs.h`这两个文件, 通过命令`cc -c main.c`进行编译, **依赖关系的实质上就是说明了目标文件是由哪些文件生成的**

- 对于是否执行命令, make会比较targets文件和prerequisites文件的修改日期，如果prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，make就会执行后续定义的命令, 这一点对于大型项目来说十分重要, 因为大型项目的编译过程十分耗时, 我们通常不会因为改了一个变量名而重新编译这个工程, make的这种机制可以使得只编译更改过的文件

- clean不是target，其冒号后什么也没有，make不会自动去找文件的依赖性，在make命令后明显得指出clean, 即`make clean`, 才能自动执行其后所定义的命令, 这样的方法非常有用，我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份等.


### **变量**

我们重新看一下Makefile里的内容, 发现其中main.o等文件特别的冗长并且出现了三次, 

```bash
edit : main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o

```

如果随着edit功能的增加, 会产生越来越多的Object Files, 那么维护Makefile的成本会越来越高, make提供了变量来解决这个问题:

```bash
OBJECTS = main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o

edit : $(OBJECTS)
    cc -o edit $(OBJECTS)
main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit $(OBJECTS)

```

- `OBJECTS`是一个变量, 当有新的功能的`.o`文件加进来, 我们只需要更改这个变量就好

### **自动推导**

Make的强大之处在于它可以根据文件和文件依赖自动推导`命令`, 比如make看到一个main.o文件，它就会自动的把main.c文件加在依赖关系中，那么main.c就是main.o的依赖文件, 并且` cc -c main.c `也会被推导出来，makefile不用写得这么复杂, 如下所示:

```bash
OBJECTS = main.o kbd.o command.o display.o /
insert.o search.o files.o utils.o

edit : $(OBJECTS)
    cc -o edit $(OBJECTS)
main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
    -rm edit $(OBJECTS)

```

- 由规则可知, `name.o`后面不用加上`name.c`文件, 同时编译命令也不需要显示的写出来

- `.PHONY`表示，clean是个伪目标文件, `-`表示某些文件出现问题，忽略问题继续做后面的事, clean都是要放在文件的最后的

### **引用**

在Makefile使用include关键字可以把别的Makefile包含进来，这很像C语言的#include，被包含的文件会原模原样的放在当前文件的包含位置。

具体的语法是:

`include <filename>`

- `filename`可以是当前操作系统Shell的文件模式, 也就是可以包含路径和通配符

举个例子:

- 有a.mk、b.mk、c.mk三个Makefile文件, 还有一个文件叫foo.make，以及一个变量$(bar)，其包含了e.mk和f.mk

`include foo.make *.mk $(bar)`

等价于：

`include foo.make a.mk b.mk c.mk e.mk f.mk`

make命令会把include所指出的其它Makefile的内容安置在当前的位置, 如果文件都没有指定绝对路径或是相对路径的话，make首先会在当前目录下寻找，如果当前目录下没有找到，make还会在下面的几个目录下找：

1. 如果make执行时，有`-I`或`--include-dir`参数，那么make就会在这个参数所指定的目录下去寻找

2. 如果目录<prefix>/include（一般是：`/usr/local/bin`或`/usr/include`）存在的话，make也会去找

如果有文件没有找到的话，make会生成一条警告信息，但不会马上出现错误信息。它会继续载入其它的文件，一旦完成makefile的读取，make会再重新寻找没有找到或是不能读取的文件，如果还是不行，make才会出现错误信息。

如果你想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个`-`, 即无论include过程中出现什么错误，都不要报错继续执行：

`-include <filename>`

### **拓展**

通常我们的项目中难免会引用一些外部库, 在编译成可执行文件之后, 我们希望将可执行文件和外部库同时打包在任何的路径下都可以运行, 如果想让可执行文件成功的找到外部库就需要在`RPATH`中使用相对路径:

例如要指定相对于可执行文件的路径，可以这样：

`gcc -L/foo/lib/location -lfoo -Wl,--rpath='$$ORIGIN/../lib' source.c -o binary`

- 在执行` binary `可执行文件时，这里的` $ORIGIN `变量会被链接器替换为可执行文件所在的路径, 如果将可执行文件和动态库一起移动到别的位置, 链接器也能通过相对位置找到对应的动态库正常运行

> https://blog.theerrorlog.com/the-gnu-linker-and-rpath.html