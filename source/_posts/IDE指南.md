---
title: IDE指南
date: 2016-08-11 09:34:19
toc: true
tags:
- IDE
---
记录常用IDE的使用, 自己常用的IDE有Jetbrains家族的`IDEA`,`Pycharm`,`DataGrip`,在写C++代码时候用`Visual studio`, 平时练习算法和数据结构用`CodeBlocks`
<!--more-->

### **IDEA**

> Ctrl + 

- `Ctrl + H` 打开类视图
- `Ctrl + R` 在当前文件进行文本替换
- `Ctrl + F` 在当前文件进行文本查找
- `Ctrl + X` 剪切行
- `Ctrl + Y` 删除光标所在行 
- `Ctrl + D` 复制行
- `Ctrl + O` 重写方法
- `Ctrl + G` 定位行
- `Ctrl + +` 展开代码
- `Ctrl + -` 折叠代码
- `Ctrl + /` 注释光标所在行代码，会根据当前不同文件类型使用不同的注释符号 
- `Ctrl + Space` 基础代码补全

> Ctrl+Shift +

- `Ctrl + Shift + F` 根据输入内容查找整个项目 或 指定目录内文件 
- `Ctrl + Shift + R` 根据输入内容替换对应内容，范围为整个项目 或 指定目录内文件 
- `Ctrl + Shift + J` 自动将下一行合并到当前行末尾 
- `Ctrl + Shift + Z` 取消撤销 
- `Ctrl + Shift + W` 递进式取消选择代码块。可选中光标所在的单词或段落，连续按会在原有选中的基础上再扩展取消选中范围
- `Ctrl + Shift + N` 通过文件名定位 / 打开文件 / 目录，打开目录需要在输入的内容后面多加一个正斜杠 
- `Ctrl + Shift + U` 对选中的代码进行大 / 小写轮流转换 

> Ctrl+Alt +

- `Ctrl + Alt + L` 格式化代码，可以对当前文件和整个包目录使用 
- `Ctrl + Alt + O` 优化导入的类，可以对当前文件和整个包目录使用
- `Ctrl + Alt + 左方向键`  退回到上一个操作的地方 
- `Ctrl + Alt + 右方向键`  前进到上一个操作的地方 
- `Ctrl + Alt + 前方向键`  在查找模式下，跳到上个查找的文件
- `Ctrl + Alt + 后方向键`  在查找模式下，跳到下个查找的文件
- `Ctrl + Shift + /` 代码块注释
- `Ctrl + Shift + Enter` 自动结束代码，行末自动添加分号


**打包jar文件**

1. 菜单：`File`->`project stucture`

2. 在弹窗最左侧选中`Artifacts`->`"+"`,选`jar`，选择`from modules with dependencies`，然后会有配置窗口出现，配置完成后`ok`保存

3. 然后菜单：`Build`->`build Artifacts`->`build`

4. 最后在项目的`out/artifacts/`目录找输出的jar包

### **VS2013**

**项目相关的快捷键**

`Ctrl + Shift + B` = 生成项目

`Ctrl + Alt + L` = 显示Solution Explorer（解决方案资源管理器）

`Shift + Alt+ C` = 添加新类

`Shift + Alt + A` = 添加新项目到项目

**编辑相关的快捷键**

`Ctrl + Enter` = 在当前行插入空行

`Ctrl + Shift + Enter` = 在当前行下方插入空行

`Ctrl +空格键` = 使用IntelliSense（智能感知）自动完成

`Alt + Shift +箭头键(←,↑,↓,→)` = 选择代码的自定义部分

`Ctrl + }` = 匹配大括号、括号

`Ctrl + Shift +}` = 在匹配的括号、括号内选择文本

`Ctrl + Shift + S` = 保存所有文件和项目

`Ctrl + K，Ctrl + C` = 注释选定行

`Ctrl + K，Ctrl + U` = 取消选定行的注释

`Ctrl + K，Ctrl + D` = 正确对齐所有代码

`Shift + End` = 从头到尾选择整行

`Shift + Home` = 从尾到头选择整行

`Ctrl + Delete` = 删除光标右侧的所有字

**导航相关的快捷键**

`Ctrl +Up/Down` = 滚动窗口但不移动光标

`Ctrl + -` = 让光标移动到它先前的位置

`Ctrl ++` = 让光标移动到下一个位置

`F12` = 转到定义

**调试相关的快捷键**

`Ctrl + Alt + P` = 附加到进程

`F10` = 调试单步执行

`F5` = 开始调试

`Shift + F5` = 停止调试

`Ctrl + Alt + Q` = 添加快捷匹配

`F9` = 设置或删除断点

**搜索相关的快捷键**

`Ctrl + K  Ctrl + K` = 将当前行添加书签

`Ctrl + K  Ctrl + N` = 导航至下一个书签

`Ctrl + . `= 如果你键入一个类名如`Collection<string>`，且命名空间导入不正确的话，那么这个快捷方式组合将自动插入导入

`Ctrl + Shift + F` = 在文件中查找

`Shift  + F12` = 查找所有引用

`Ctrl + F` = 显示查找对话框

`Ctrl + H` = 显示替换对话框

`Ctrl + G` = 跳转到行号或行

`Ctrl + Shift + F` = 查找所选条目在整个解决方案中的引用





