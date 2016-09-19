---
title: Shell编程笔记
date: 2016-03-21 19:21:43
tags:
  - Shell
categories:
  - 笔记
---
学习Shell笔记
<!--more-->

###  **什么是Shell?**

![Linux系统结构](/img/Shell编程/shell.svg)

- 上图代表整个计算机系统, 由图可知 ,有内层到外层就是由计算机底层硬件到上层应用
- `Hardware`代表计算机底层硬件
- `Kernel` 代表操作系统的核心
- `Programs`和`Utilities`是运行在操作系统上的应用
- `Shell`和`GUI`严格意义上讲是应用的一种, 但是它们是用户和操作系统之间的接口, 与用户产生交互
- `GUI `提供了一种图形化的用户接口，使用起来非常简便易学,而`Shell` 则为用户提供了一种命令行的接口，接收用户的键盘输入，并分析和执行输入字符串中的命令，然后给用户返回执行结果

$ `echo $SHELL`
`/bin/bash`
$ `ls -l /bin/bash`
`-rwxr-xr-x 1 root root 906248 5月  11 07:21 /bin/bash`

###  **bash应用**

> bash不仅能够解释简单的命令，而且可以解释一个具有特定语法结构的文件，这种文件被称作`脚本`（Script）,即`.sh`文件

`chmod +x ./test.sh`
`./test.sh`
- 执行sh文件, 首先`test.sh`文件要有可执行权限, 如果没有添加可执行权限 
- `bash test.sh`
- `source test.sh`
- `. test.sh`

> 更多shell例子 [Shell编程学习笔记](https://tinylab.gitbooks.io/shellbook/content/zh/appendix/02-chapter1.html#toc_19246_27800_3)

### 布尔运算

> Bash 里头的 true 和 false 是我们通常认为的 1 和 0 吗？

回答是：`否`。

- true 和 false 它们本身并非逻辑值，它们都是 Shell 的内置命令，只是它们的返回值是一个“逻辑值”：
$ `true`
$ `echo $?`
0
$ `false`
$ `echo $?`
1

- 可以看到 true 返回了 0，而 false 则返回了 1 。跟我们离散数学里学的真值 1 和 0 并不是对应的，而且相反的
- `$?` 是一个特殊变量，存放有上一次进程的结束状态（退出状态码）
- 在 C 语言程序设计中为什么会强调在 main 函数前面加上 int，并在末尾加上 return 0 。因为在 Shell 里，将把 0 作为程序是否成功结束的标志，这就是 Shell 里头 true 和 false 的实质，它们用以反应某个程序是否正确结束，而并非传统的真假值（1 和 0），相反地，它们返回的是 0 和 1 。不过庆幸地是，我们在做逻辑运算时，无须关心这些。

1. `-a` : and 与运算
2. `-o` : or  或运算
3. `!`  : 非运算

### **更多**

[Shell脚本调试技术](http://www.ibm.com/developerworks/cn/linux/l-cn-shell-debug/index.html)
[Shell十三问](http://bbs.chinaunix.net/thread-218853-1-1.html)
[Shell十二篇](http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=2198159)