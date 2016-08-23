---
title: IDE指南
date: 2016-08-11 09:34:19
tags:
- IDE
---
记录常用IDE的使用, 自己常用的IDE有Jetbrains家族的`IDEA`,`Pycharm`,`DataGrip`,在写C++代码时候用`Visual studio`, 平时练习算法和数据结构用`CodeBlocks`
<!--more-->

### IDEA

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


### **打包jar文件**

1. 菜单：`File`->`project stucture`

2. 在弹窗最左侧选中`Artifacts`->`"+"`,选`jar`，选择`from modules with dependencies`，然后会有配置窗口出现，配置完成后`ok`保存

3. 然后菜单：`Build`->`build Artifacts`->`build`

4. 最后在项目的`out/artifacts/`目录找输出的jar包





