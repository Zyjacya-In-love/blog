---
title: Linux运行后台程序
date: 2016-06-13 11:04:49
tags:
  - Linux
categories:
  - Linux
---
问题 : 用 `telnet/ssh` 登录了远程的 Linux 服务器，运行了一些耗时较长的任务， 结果却由于网络的不稳定导致任务中途失败, 如何让命令提交后不受本地关闭终端窗口或者网络断开连接的干扰呢？
<!--more-->

### **后台运行程序**

- 原因 : 当用户注销（logout）或者网络断开时，终端会收到 HUP（hangup）信号从而关闭其所有子进程。

- 思路 : 要么让进程忽略 HUP 信号，要么让进程运行在新的会话里从而成为不属于此终端的子进程。

> `nohup` : 让提交的命令忽略 hangup(关闭子进程) 信号

   - 在要处理的命令前加上` nohup` 即可，标准输出和标准错误缺省会被重定向到 nohup.out 文件中。
   - 可以在结尾加上"&"来将命令同时放入后台运行
   - 可用">filename 2>&1"来更改缺省的重定向文件名
   $ `nohup ping www.ibm.com &`
   输出 : `[1] 5558`

   $ `ps -ef |grep 5558`
   `root      5558  4995  0 10:55 pts/0    00:00:00 ping www.ibm.com`
    
   $ `ps -ef |grep 4995`
   `root      4995  4994  0 10:48 pts/0    00:00:00 bash`
   `root      5558  4995  0 10:55 pts/0    00:00:00 ping www.ibm.com`
   - 可以看出运行的ping程序(子进程)属于bash进程(父进程),该命令只是忽略 hangup(关闭子进程) 信号
    
> `setsid` : 让进程不属于接受HUP信号的终端的子进程

   - `setsid ping www.ibm.com`
   - `ps -ef |grep www.ibm.com`

- 与`nohup`比较 :

```
[root@VM_49_124_centos shang]# ps -ef |grep www.ibm.com
root      4527     1  0 10:41 ?        00:00:00 ping www.ibm.com
root      5558  4995  0 10:55 pts/0    00:00:00 ping www.ibm.com
#
```

- setsid的进程ID(PID)为4527，而它的父 ID（PPID）为1（即为 init 进程 ID），并不是当前终端的进程 ID, 说明该命令让进程运行在新的会话里从而成为不属于此终端的子进程

> `&` : 

   - 将一个或多个命名包含在`()`中就能让这些命令在子 shell 中运行, 当我们将`&`也放入`()`内之后，我们就会发现所提交的作业并不在作业列表中，也就是说，是无法通过jobs来查看的。
   - `(ping www.ibm.com &)`
   - `ps -ef |grep www.ibm.com`
   `root      6767     1  0 11:14 ?        00:00:00 ping www.ibm.com`
   - 新提交的进程的父 ID（PPID）为1（init 进程的 PID），并不是当前终端的进程 ID。因此并不属于当前终端的子进程，从而也就不会受到当前终端的 HUP 信号的影响了。
   
> 教程网址 [Linux 技巧：让进程在后台可靠运行的几种方法](https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/)
    