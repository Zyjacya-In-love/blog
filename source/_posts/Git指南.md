---
title: Git指南
date: 2015-10-24 19:37:49
toc: true
tags:
  - Git
categories:
  - DevTools 
---

在一开始接触实验室项目的时候, 用的是SVN, 后来把自己写的东西分享到Github上开始接触Git, 整理一下常用知识点

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

`git log`

- 查看`commit`的记录

`git commit --amend`

- 修改最后一次提交

`git reset HEAD <file>`

- 取消已经暂存的文件

`git checkout -- <file>`

- 在暂存状态下，取消对文件的修改

> 一定要明确文件所在的工作区（工作区，暂存区，本地代码库）

`git reset --hard commit-id`

- 本地代码库回滚到commit-id

**远程代码库回滚**：

先将本地分支退回到某个commit，删除远程分支，再重新push本地分支。

`git checkout the_branch`

- 切换到the_branch分支

`git pull`

- 更新文件

`git branch the_branch_backup`

- 备份当前分支

`git reset --hard the_commit_id`

- 回滚到某一commit_id

`git push origin :the_branch`

- 删除远程 the_branch

`git push origin the_branch`

- 用回滚后的本地分支重新建立远程分支

`git push origin :the_branch_backup`

- 删除备份分支

### **Push本地分支**

`git push [origin master]`

- 在`commit`改动之后就有了新的本地分支, 执行`push`将这些改动提交到远端仓库, 可以把`master`换成你想要推送的任何分支

`git remote add origin <server>`

- 如果没有克隆现有仓库，并想将仓库连接到某个远程服务器，使用`remote add`添加远程分支

### **分支(brancd)**

![](/img/Git指南/branches.png)

在创建一个仓库的时候, `master`是`默认的`主分支

`git checkout -b feature_brancd`

- 创建一个叫`feature_brand`的分支, 并切换到该分支下

`git checkout master`

- 切换回主分支

`git branch`

- 查看所有分支

`git branch -d feature_branch`

- 删除新建的分支

`git branch -v`

- 查看各个分支最后一个提交对象的信息

`git branch --merged/--no-merged`

- 查看哪些分支已被并入当前分支, 所显示的列表中没有`*`的分支通常都可以用` git branch -d `删掉

> 除非你将分支推送到远端仓库，不然该分支都是本地分支与远程服务器交互没有关系：

`git push origin <branch>`

- 将本地分支提交到远程仓库

- [远程分支的详细介绍](http://iissnan.com/progit/html/zh/ch3_5.html)

### **分支衍合（rebase）**

`git rebase branch_name`

- `衍合`是把在一个分支里提交的改变移到另一个分支里重放一遍

- 实际上是把解决分支补丁同最新主干代码之间冲突的责任，化转为由提交补丁的人来解决，一旦分支中的提交对象发布到公共仓库，就千万不要对该分支进行衍合操作。

- [分支的衍合](http://iissnan.com/progit/html/zh/ch3_6.html)

### **远程仓库**

`git fetch <origin> <master>`

- 创建并更新所有远程分支的本地远程分支, 设定当前分支的FETCH_HEAD为远程服务器的master分支

- FETCH_HEAD指的是: `某个branch在服务器上的最新状态`

`git fetch origin branch_1`

- 设定当前分支的`FETCH_HEAD`为远程服务器的branch_1分支

`git fetch origin branch_1:branch_2`

- 使用远程branch_1分支在本地创建branch_2, 但不会切换到branch_2,

- 如果本地不存在branch_2分支, 则会自动创建一个新的branch2分支

- 如果本地存在branch2分支, 并且是`fast forward', 则自动合并两个分支, 否则, 会阻止以上操作

`git fetch origin :branch2`

- 等价于`git fetch origin master:branch2`

`git pull`

- 更新本地仓库使得和远程仓库一致, 相当于在你的工作目录中 获取（fetch）并合并（merge）远程仓库的改动

- `git fetch origin` + `git merge FETCH_HEAD`

### **合并(merge)**

`git branch`

- 查看所有分支并确定HEAD在哪个分支上

`git merge <branch>`

- 将其他分支和当前分支合并, 合并不是每次都成功，并可能出现冲突（conflicts）, 这时候就需要你修改这些文件来手动合并这些冲突（conflicts）

`git status`

- 查看哪些文件有冲突，并人工编辑文件留下想要的代码

`git add <filename>`

- 改完之后，你需要执行`git add`以将它们标记为合并成功，然后`git commit`进行提交到本地代码库

`git branch -d branch_name`

- 合并之后可以将之前的分支删除

`git diff <source_branch> <target_branch>`

- 用`diff`命令预览两个分支的不同

### **标签(tag)**

`git log`

- 查看标记提交的ID

`git tag v_1.0.1 1b2e1d63ff`

- 创建一个叫做`v_1.0.1`的标签, `1b2e1d63ff` 是你想要标记的commit_ID的前10位字符

### [**团队协作**](http://iissnan.com/progit/html/zh/ch5_2.html)

### **修改commit历史**

### **子模块**

`git submodule add <git_url>`

- 将外部项目加为子模块

`git submodule init`

- 初始化你的本地配置文件`.git/config`

`git submodule update`

- 从子项目拉取所有数据并检出上层项目里所列的合适的提交

`git submodule foreach --recursive git pull origin master`

- 一次全部更新所有的` submodule`

`git rm --cached /path/to/files`
`rm -rf /path/to/files`

- 刪除 `submodule`

`git submodule sync`

-  `submodule `的远程地址发生改变，编辑`.gitmodules`中修改，然后执行该命令

> 改动并提交Submodule代码:

- 需要单独把` submodule `的代码提交，把` submodule` 当做一个独立的 `git` 工程对待:

```
cd submodules // 进入 submodule 目录
git add .
git commit -m "xxxx"
git push origin master  //提交submodule代码
```
- 返回main工程:

```
git add .
git commit -m "update submodule"
git push origin master  //提交main 代码
```

为什么分别提交了两次 ？
Git 对submodule 的管理是这样的，在 main 工程中，并不递归记录 submodule 中文件的变动，仅仅记住 submodule 的分支和Commit Id ，这样足够从一个 submodule 中取出需要的代码，而 submodule的管理是完全独立的。所以需要先把submodule的改动提交了，再在 main 工程更新submodule 的分支和commit id 变化。

### **更多**

[Git Pro](http://iissnan.com/progit/)

[git简明指南](http://rogerdudler.github.io/git-guide/index.zh.html)

[图解Git](http://marklodato.github.io/visual-git-guide/index-zh-cn.html)













