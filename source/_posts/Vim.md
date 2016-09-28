---
title: Vim
date: 2016-05-21 09:42:45
toc: true
tags:
  - Vim
categories:
  - Linux
---
Vim是Linux下的文本编辑器, 在布置服务器的过程中修改一些配置文件的时候十分方便, 自己平时也就是改改配置文件和一些程序的时候用到, 整理一下常用的Vim用法
<!--more-->

### **Vim模式**

在Linux系统中, 绝大部分的配置文件都是以` ASCII `的纯文本形态存在, Vim具有程序代码高亮的功能, 掌握必要的Vim操作提高工作效率, Vim有三种工作模式(状态):

- 一般模式(默认) : 该模式下可以使用光标来浏览内容, 也可以删除, 复制, 粘贴内容但不能添加内容

- 编辑模式 : 按下`i, I, o, O, a, A, r, R`进入编辑模式, 在左下方会出现`INSERT(插入) 或 REPLACE(替换)`, 如果要回到一般指令模式时，按下`Esc`按键即可退出编辑模式

- 命令行模式 : 在一般模式中, 输入`: / ?` 任意一个符号进入命令行模式, 在该模式中可以搜寻数据，而读取、存盘、替换字符、离开 vi 、示行号等功能

### **一般模式命令**

|    命令    |    解释    |
|---------------- | ---------------|
|h 或 向左方向键（←）| 光标向左移动一个字符|
|j 或 向下方向键（↓）| 光标向下移动一个字符|
|k 或 向上方向键（↑）| 光标向上移动一个字符|
|l 或 向右方向键（→）| 光标向右移动一个字符|
|$ 或功能键`[End]` |移动到这一行的最后面字符处|
|G              |移动到这个文件的最后一行 |
|gg       |移动到这个文件的第一行，相当于1G| 
|n`<Enter>` | n 为数字, 光标向下移动 n 行|
|/word |向光标之下寻找一个名称为 word 的字串|
|x |x 为向后删除一个字符（相当于 `[del]`）|
|X  |X 为向前删除一个字符（相当于 `[backspace]`）| 
|dd| 删除光标所在的那一整行|
|20dd| 删除光标所在的向下 20 行|
|yy |复制光标所在的那一行|
|20yy |复制光标所在的向下 20 行|
|*y | 复制全部 |
|p|已复制的数据在光标所在行的下面粘贴|
|P| 已复制的数据在光标所在行的上面粘贴|
|u |复原前一个动作 |
|`[Ctrl]`+r |重做上一个动作|
|`.`|重复前一个动作|

### **进入编辑模式**

|    命令    |    解释    |
|---------------- | ---------------|
|i,I |`i`为“从目前光标所在处插入”，`I`为“在目前所在行的第一个非空白字符处开始插入”|
|a, A|`a`为“从目前光标所在的下一个字符处开始插入”， A 为“从光标所在行的最后一个字符处开始插入”|
|o, O|`o`为“在目前光标所在的下一行处插入新的一行”，`O` 为在目前光标所在处的上一行插入新的一行|
|r, R|取代模式（Replace mode）：`r` 只会取代光标所在的那一个字符一次；`R`会一直取代光标所在的文字，直到按下 ESC 为止|

- `[Esc]`退出编辑模式，回到一般指令模式

### **命令行模式**
|    命令    |    解释    |
|---------------- | ---------------|
|:w |将编辑的数据写入硬盘文件中|
|:q |离开 Vim|
|:q!|若修改过文件，又不想储存，使用` ! `为强制离开不储存|
|:wq |储存后离开，若为 `:wq!` 则为强制储存后离开|
|:w [filename]| 将编辑的数据储存成另一个文件|
| :r [filename]|在编辑的数据中，读入另一个文件的数据。亦即将“filename” 这个文件内容加到光标所在行后面|
|:set nu|显示行号|
| :set nonu |不显示行号|

### **图示**
                                                                          
![](/img/Vim/vim1.png)

[Vim](http://michael.peopleofhonoronly.com/vim/)

![](/img/Vim/vim.png)

   - `Green`  = Essential
   - `Yellow`   = Basic
   - `Orange/Blue` = Advanced
   - `Red`   = Expert
   
### **配置C++代码提示**

1. 安装Vim插件管理工具 [Vundle](https://github.com/VundleVim/Vundle.vim#about)

   `git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`
   
   - 上面命令将vundle克隆到用户根目录的`~/.vim/bundle/Vundle.vim`

2. 在用户根目录下创建`.vimrc`文件, 在该文件中配置Vundle使用Vim相关插件, [.vimrc文件](https://github.com/VundleVim/Vundle.vim#quick-start)

   `vim ~/.vimrc`
   
   - 安装代码补全插件[YouCompleteMe](https://github.com/Valloric/YouCompleteMe), 进入`~/.vim/bundle/`将YouCompleteMe克隆到该目录
   
   - 添加`Plugin 'Valloric/YouCompleteMe'`, 然后进入Vim命令模式执行`:PluginInstall`, 最后还需要去`~/.vim/bundle/YouCompleteMe`中`./install.sh`一下
   
3. 编译YouCompleteMe

   - 在编译之前下载编译工具，准备编译YouCompleteMe
   
   `sudo yum install cmake python python-dev build-essential automake`
   
   - 编译YouCompleteMe使其支持C/C++ 自动补全
   
   `cd ~/.vim/bundle/YouCompleteMe`
   `./install.py --clang-completer`
   
4. 配置Vim : 修改`.vimrc`文件

   ```
   let g:ycm_global_ycm_extra_conf = '~/.vim/bundle/YouCompleteMe/.ycm_extra_conf.py  
   let g:ycm_seed_identifiers_with_syntax=1    " 语法关键字补全  
   let g:ycm_confirm_extra_conf=0   " 打开vim时不再询问是否加载ycm_extra_conf.py配置  
   inoremap <expr> <CR>  pumvisible() ? "\<C-y>" : "\<CR>"    "回车即选中当前项  
   set completeopt=longest,menu    "让Vim的补全菜单行为与一般IDE一致(参考VimTip1228)  
   ```

   
   

   



