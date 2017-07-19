title: 用Markdown来写Wordpress文章
id: 83
categories:
  - WordPress日常
  - 日志
date: 2015-05-21 22:44:10
tags:
---

### 安装Jetpack插件

WordPress里要用MARKDOWN写东西要用到一个插件就是JP-MARKDOWN。这个是官方出的插件。而在国内网络貌似是无法链接到Wordpress官网的，翻墙之后安装好JP-MARKDOWN就行了。

<!--more-->
不过Wordpress的Markdown语法貌似跟正统的Markdown不太一样，而跟Markdown extra差不多，可以参见Markdown extra的文档。
[MARKDOWN_EXTRA](https://michelf.ca/projects/php-markdown/extra/ "https://michelf.ca/projects/php-markdown/extra/")

不过大体上的效果还是差不多的。注意Markdown的语法，尤其是空格的使用，否则挺麻烦的。

### 安装Crayon Syntax Highlighter插件

这个是个语法高亮的插件，功能很强大，也完美兼容WordPress的Markdown。在下载安装Crayon Syntax Highlighter后，注意去插件设置处，将在 Crayon Syntax Highlighter 设置中不要勾选“捕获 反引号 为 标签”，否则会和Jetpack插件冲突。

#### **注意的点**
其他的东西都差不多，但是有一个点要注意，那就是代码的加入。若是在本地文本编辑器中，例如sublimetext里加入代码，我们通常在代码块的头尾加三个重音符，也就是数字键1左边那个键，一个点号。但是如果在WordPress文章里直接复制你在本地写的MARKDOWN文章，若不是从Crayon插件里加入代码，则一定要记得在代码段的前后加上pre标签，并且删去那三个重音符的代码标记符号，否则会出现代码错误。

## 还有其他方式利用Markdown写文章
我自己非常喜欢的一款文本编辑器Sublime Text在安装OmniMarkupPreviewer插件后，也能对Markdown有很好的支持，并且支持在浏览器中预览。在离线的情况下用Sublime Text来进行Markdown文章的书写体验还是相当棒的。可以尝试。另外t这个轻量级的文本编辑器在WINDOWS系统上的体验效果是相当棒的。并且支持VIM的大部分命令，也就是可以拿ST来练手VIM的操作。
