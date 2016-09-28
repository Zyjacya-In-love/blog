---
title: Github指南
date: 2015-11-19 20:43:58
toc: true
tags:
  - Github
categories:
  - DevTools 
---
在玩儿Github的过程中, 自己的博客就是搭在Github Page上的, 在用caffe的过程中, [Netscope](http://ethereon.github.io/netscope/quickstart.html) 可以直接可视化`Github Gist`分享的`.prototxt`文件, 整理Github相关的用法 
<!--more-->

### **SSH for Github**

> 环境: Ubuntu14.04 64bit

1. `sudo apt-get install ssh`
   - 安装ssh服务
   
2. 在`/home`目录下检查SSH公钥, `cd ~/.ssh`查看`.ssh`是否存在

3. 如果不存在, 使用你在注册github时使用的Email生成一个SSH公钥私钥对
   $`ssh-keygen -t rsa -C "simshangy@gmail.com"`
   Generating public/private rsa key pair.
   Enter file in which to save the key (/home/you/.ssh/id_rsa):
   
4. 中间要求输入密码可以为空, 然后在.ssh中可以看到`id_rsa`, `id_rsa.pub`和`known_hosts`
   - 其中idrsa为私钥，idrsa.pub为公钥, 我们需要的是公钥
   
5. 添加SSH公钥到Github, 打开github的`setting`，找到`SSH and GPG keys`，然后点击`New SSH key` , 把idrsa.pub内容复制到key里面, 点击 `Add SSH key`

6. `ssh -T git@github.com` 测试是否成功: 输入yes

   ```
   The authenticity of host 'github.com (207.97.227.239)' can't be established.
   RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
   Are you sure you want to continue connecting (yes/no)?
   
   Hi username! You've successfully authenticated, but GitHub does not
   provide shell access.
   ```


### **删除Github上的文件夹**

1. 首先在本地将文件夹删除

`git rm [-r] [-f] fodername`

2. 然后commit

`git commit -a -m"[commit text]"`

3. push到Github

`git push`

### **Create a new repository**

登录Github之后可以创建远程仓库, 私有仓库要收费

- 如果想向Github添加手中已有的Git仓库，建议不要勾选Initialize this repository with a README选项；
- `Add.gitignore` ：可以在初始化时生成.gitignore文件，这个设定会帮我们把不需要在Git仓库中进行版本管理的文件记录在.gitignore文件中，省去了每次根据框架进行设置的麻烦。若不使用任何框架，则可不选择。
- `Add a license` ：选择要添加的许可协议文件，一般可不选。

**Fork**
我们知道在操作系统中, 父进程创建子进程的时候用到了一个`fork()`函数,  在Github上的Fork也是这个意义, 将你要修改代码的项目仓库Fork到自己的Github账号上, 创建一个属于你的子仓库, 这个仓库本质上就是朱仓库的一个分支

**Pull Request**
用户修改自己Fork的仓库里的代码后可以请求对方仓库和自己的仓库合并, 也就是将分支Merge到Master上

**Wiki**

Wiki是一个使用简单的语法就能编写文档的功能。

所有有权限的人都可以对文中进行修改。

Wiki多被用于编写博客文章、教程、使用手册。


### **new issuse**

在软件开发过程中，开发者们为了跟踪BUG及进行软件相关讨论，进而方便管理，创建了Issue。

在Github上，可以将它作为开发者之间的交流工具，多多加以利用。

Issue可以在以下情况使用：

- 发现软件的Bug并报告
- 有事想向作者询问、探讨
- 事先列出今后准备实施的任务
- Issue支持markdown语法，也支持添加标签便于管理, 在Issue里可以添加图片，可以使用表情

### **Github Page**

[Github Pages](https://pages.github.com/)是面向用户、组织和项目开放的公共静态页面搭建托管服务，站点可以被免费托管在Github 上，你可以选择使用Github Pages 默认提供的域名github.io 或者自定义域名来发布站点。我们可以利用Github Page来搭建自己的博客


### **Github Gist**

[Gist](https://gist.github.com/)是`Github`提供的用来分享代码片段的服务, 对于我来说, 我使用的一个Caffe Prototxt的可视化工具, 可以直接解析Gist的`.prototxt`文件
 
 **功能**
 
 - 无限制的免费创建私有代码片段
 - 不需要拥有Github账号就可以使用Gist
 - 版本管理功能
 - 使用嵌入网页代码功能, 会获得一个相应js地址, 然后如gist显示一样显示出内容到网页中
 - 支持Markdown语法, 文件名要以`.md`结尾
 - 在本地创建gist, 可以使用Git进行操作
 
 **使用**
 
 - 点击[Gist](https://gist.github.com ) 直接填写内容或者在自己的Gist 右上角上点击 `New gist`
 - 一个可以有Gist多个文件, 使用 `Add file` 添加即可, 可以设置indent为空格space还是tab, tab长度, 是否行缩进
 - `Create secret gist` 创建私有代码, `Create public gist` 创建开放的gist
 - 创建Gist后,点选自己的某个Gist, 进去后右上角可进行网上的编辑/修改: `Edit编辑`, `Delete删除`, `Star收藏`, 修改后下方的`Update public/secret gist`即可保存修改, 编辑时上方的`Make Secret`可以转为私有库

 
### **更多Github**

[Student Developer Pack](https://education.github.com/)