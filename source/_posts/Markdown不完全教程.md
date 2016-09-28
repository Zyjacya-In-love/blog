---
title: Markdown不完全教程
date: 2016-03-17 14:44:42
toc: true
tags: 
  - Markdown
categories:
  - DevTools 
---

- Github上Markdown使用教程中文版(基本用法)，并补充了使用MathJax引擎,使得markdown支持Tex语法编写数学公式。

<!--more-->

### 什么是Markdown？

- Markdown 是一种轻量级标记语言，创始人为約翰·格魯伯（John Gruber）。 它允许人们“使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML(或者HTML)文档”，通常保存为`.md`文件。

### Markdown的基本语法



#### 标题：1-6级标题

`#一级标题`

`##二级标题`

`######六级标题`

# 一级标题

## 二级标题

###### 六级标题

#### 斜体

`hello *Markdown* !`

`hello _Markdown_ !`

>hello *Markdown* !

#### 加粗

`hello **Markdown** !`

hello **Markdown** !

### 删除线
~~删除线~~

#### 引用 PS：块引用使用多个'>'
`> hello markdown !`
> >hello markdown !

### 高亮:单个间隔号 PS:不是单引号，Esc键下面的按键
形式：`间隔号 hello markdown！ 间隔号`
> `hello markdown!`

### 代码块:三个间隔号 PS:不是单引号，Esc键下面的按键

```
(```)语言类型
...代码...
(```)
```


```c++
#include <iostream>

using namespace std;

int main(void)
{
    cout << "Hello Markdown!" << endl;
    return 0;
}
// 
```

#### 链接URL和图片

[Markdown Logo][LogoAddr]  //链接URL
![Markdown Logo][LogoAddr] //链接图片
[LogoAddr]:http://7xqkff.com1.z0.glb.clouddn.com/MarkdownLogo.png  //定义链接路径

> [Markdown Logo][LogoAddr]
> ![Markdown Logo][LogoAddr]
[LogoAddr]:http://7xqkff.com1.z0.glb.clouddn.com/MarkdownLogo.png

#### 无序表
```
 * Item 1
 * Item 2
   * Item 2a
   * Item 2b
```

> * Item 1
> * Item 2
>   * Item 2a
>   * Item 2b

#### 有序表
```
1. Item 1
2. Item 2
3. Item 3
   * Item 3a
   * Item 3b
```

> 1. Item 1
> 2. Item 2
> 3. Item 3
>  * Item 3a
>  * Item 3b`




There is a literal \`backtick\`here.

###　字符转译
\*literal asterisks\*



*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.

### 复合段落必须4个空格的缩进
1.  This is a list item with two paragraphs. Lorem ipsum dolor
    sit amet, consectetuer adipiscing elit. Aliquam hendrerit
    mi posuere lectus.

    Vestibulum enim wisi, viverra nec, fringilla in, laoreet
    vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
    sit amet velit.

2.  Suspendisse id sem consectetuer libero luctus adipiscing.


### 表格
|  字段1  |  字段2  |  字段3  |
|--------|--- -----|--------|
|  shang |  coder  |  bupt  |

### Mathjax公式:采用Tex语法

可以创建行内公式，例如：$\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$
或者块级公式，

$$ x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$



