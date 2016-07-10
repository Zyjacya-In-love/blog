---
title: Hexo去除代码块滑动条
tags:
- Hexo
date: 2016-07-07 17:48:52
---

最近将hexo升级到3.0,突然发现在代码块部分出现了滑动条,滑动条不仅挡住了代码,而且特别的难看啊有木有!(chrome除外,由此对Google更加喜欢了,May God use chrome !),对于我这种追求美的重度强迫症患者来说,简直不能忍,于是开启了修bug之路.
![](/img/hexo去除代码块滑动条/hexo3codeblockbug.JPG)

<!--more-->

其实这个问题确实是hexo3.0的一个bug,在github的issues中也讨论过,第一个方法就是降级hexo,用老版本,第二个方法就是更改hexo的代码,由于自己技术水平有限,源代码读不懂,下面我把自己的方法贴出来.

#### **更改该主题中的代码块显示设置** , 我的主题[Yilia](https://github.com/litten/hexo-theme-yilia)

源代码:`themes/yilia/source/css/_partial/highlight.styl`文件

```
pre
    @extend $code-block
    color: code-word
    overflow: hidden //添加
    code
      background: none
      text-shadow: none
      padding: 0
      color: code-word
```

- 通过浏览器的检查功能,定位到滑动条的位置,在`color: code-word`属性下面添加`overflow: hidden`

> 最终效果:
  ![](/img/hexo去除代码块滑动条/hexo3codeblock.JPG)
  



