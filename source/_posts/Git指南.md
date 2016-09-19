---
title: Git指南
date: 2016-09-18 19:37:49
toc: true
tags:
  - Git
categories:
  - 笔记
---

在一开始接触实验室项目的时候, 用的是SVN, 后来把自己写的东西分享到Github上开始接触Git, 整理一下常用知识点, 详细内容参考[Git Pro](http://iissnan.com/progit/)

<!--more-->

> 条件: 本地安装`Git`

### **创建仓库(repository)**

创建新文件夹, 进入文件夹执行下面命令

`git init`

### **克隆仓库(clone)**

`git clone /path/to/repository`

- 克隆本地仓库

`git clone username@host:/path/to/repository`

- 克隆远端服务器上的仓库

### **工作流**

![](/img/Git指南/trees.png)

如上图所示, Git的有三个区域组成, `工作目录(working dir)`, `暂存区（Index or Stage）`和`HEAD`, `HEAD`指向你最后一次提交的结果

`git add <filename>`或者` git add *`

- 将`工作目录`中没有被跟踪的文件添加到暂存区, 这样当这些文件产生变化的时候Git就可以跟踪到了

`git commit -m "代码提交的备注"`

- `add`命令并没有把文件真正的添加到本地的分支里, 只是放在了暂存区, 需要用`commit`真正的提交到本地分支

`git status`

- 查看仓库里文件的跟踪状态

### **Push本地分支**

`git push [origin master]`

- 在`commit`改动之后就有了新的本地分支, 执行`push`将这些改动提交到远端仓库, 可以把`master`换成你想要推送的任何分支

`git remote add origin <server>`

- 如果没有克隆现有仓库，并想将仓库连接到某个远程服务器，使用`remote add`添加远程分支

###　**分支(brancd)**

![](/img/Git指南/branches.png)

在创建一个仓库的时候, `master`是`默认的`主分支

`git checkout -b feature_brancd`

- 创建一个叫`feature_brand`的分支, 并切换到该分支下

`git checkout master`

- 切换回主分支

`git branch -d feature_branch`

- 删除新建的分支

> 除非你将分支推送到远端仓库，不然该分支就是 不为他人所见的：

`git push origin <branch>`

### **更新(pull)**

`git pull`

- 更新本地仓库使得和远程仓库一致, 相当于在你的工作目录中 获取（fetch）并合并（merge）远程仓库的改动

### **合并(merge)**

`git merge <branch>`

- 将其他分支和当前分支合并, 合并不是每次都成功，并可能出现冲突（conflicts）, 这时候就需要你修改这些文件来手动合并这些冲突（conflicts）
- 改完之后，你需要执行如下命令以将它们标记为合并成功：`git add <filename>`

`git diff <source_branch> <target_branch>`

- 用`diff`命令预览两个分支的不同

### **标签(tag)**

`git log`

- 查看标记提交的ID

`git tag 1.0.1 1b2e1d63ff`

- 创建一个叫做`1.0.1`的标签, `1b2e1d63ff` 是你想要标记的提交ID的前10位字符


### **更多**

[git简明指南](http://rogerdudler.github.io/git-guide/index.zh.html)

[图解Git](http://marklodato.github.io/visual-git-guide/index-zh-cn.html)













