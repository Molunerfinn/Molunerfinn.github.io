title: 在SublimeText3中使用Markdown语法的种种小技巧。
id: 127
categories:
  - WordPress日常
  - 日志
date: 2015-05-26 17:09:31
tags:
---

##MarkDown在Wordpress里的应用
这篇文章是用Sublimetext编辑的。Sublimetext对于MarkDown有个很好的插件支持，那就是OmniMarkupPreviewer。 用ST3这个轻量级的编辑器写东西是一个很舒服的选择。而Wordpress又支持MarkDown，所以我可以用ST3写完文章再发表到WP。

##Markdown在Sublime的snippet
[这个是Markdown在Sublime里的一些有趣的用法](http://blog.leanote.com/post/54bfa17b8404f03097000000 "http://blog.leanote.com/post/54bfa17b8404f03097000000")</br>

用snippet能加速我们在Sublime text3中对Markdown的开发，而不仅仅局限于Markdown。</br>
自定义了一些snippet </br>

*   mdlink - 插入链接
*   mdacr - 插入参考式链接
*   mdfn - 插入脚注
*   mdimg - 插入图片
*   tab - 首行缩进四个字符（两个汉字）
*   br - 换行

</br>
另外，我电脑中自定义snippet的目录在`C:\Users\lenonvo\AppData\Roaming\Sublime Text 3\Packages\User`下,不同电脑可能不一样，不过都是在ST3文件夹下的`Packages\User`下，文件名命名为`*.sublime-snippet`

##记录一下在Sublime中如何自定义snippet
首先先给出一个snippet的最基本的格式

> 以下是snippet的基本代码，代码类型是xml

```xml
<snippet>
    <content><![CDATA[Type your snippet here]]></content>
    <!-- Optional: Tab trigger to activate the snippet -->
    <tabTrigger>hello</tabTrigger>
    <!-- Optional: Scope the tab trigger will be active in -->
    <scope>source.python</scope>
    <!-- Optional: Description to show in the menu -->
    <description>My Fancy Snippet</description>
</snippet>
```


</br>
&emsp;&emsp;`content`里代表你想要自定义的代码片段,注意，`![CDATA[]]`这段框体是一定要有的否则会出错</br>
&emsp;&emsp;`tabTrigger`里代表你输入的简短代码，按tab键后就能快速展开成content里的代码段 </br>
&emsp;&emsp;`scope`里的代码代表这段Snippet适用于什么语言类型，如果留空就是适用于任何类型 </br>
&emsp;&emsp;`description`里的是描述性文字，用来描述你这段Snippet用来做什么的，会在你输入`tabTrigger`里关键字的时候在模糊匹配栏后显示出来，来提示你这段代码的作用 </br>

####接下来给出一段Snippet代码用来分析

```xml
<snippet>
    <content><![CDATA[
[${1:Display_Text}][${2:id}]$5

[$2]:${3:http://example.com/} ${4:"$3"}
]]></content>
    <tabTrigger>mdacr</tabTrigger>
    <scope>text.html.markdown.multimarkdown, text.html.markdown</scope>
    <description>Link Anchor</description>
</snippet>
```

</br>
&emsp;&emsp;这是一段基于MARKDOWN语法的自定义snippet。我们注意到，在`![CDATA[]]`内部出现了`$`,这个符号的作用是让tab键按下后光标所在的位置。`$1`就是tab键按下后首先出现的位置，而后再按下Tab键，光标会移动到`$2`的位置，以此类推。 </br>
&emsp;&emsp;如果想光标停留的地方留下一些默认的提示性文字，那么就用这个结构`${*:Display_Text}`,其中的`*`是你要用到的数字，
`Display_Text`里放的就是你想要的提示性文字。这样就能做到用Tab键快速在代码块中穿梭输入，提升我们的输入效率

</br>
##记录一下如何在MarkDown首行缩进
在行首输入`&amp;emsp`，&amp;emsp是代表2个字符宽度的空格,也就是一个汉字的宽度。为了符合中文段落行首两个汉字的习惯，通常我们可以输入两次`&amp;emsp;&amp;emsp;`,但是明显每次都要这么输入实在是很麻烦，所以结合上SublimeText3的Snippet特性就能很快定义这块代码块，实现快速输入。 </br>
代码如下： </br>

```xml
<snippet>
    <content><![CDATA[&emsp;&emsp;]]></content>
    <tabTrigger>tab</tabTrigger>
    <scope>text.html.markdown.multimarkdown, text.html.markdown</scope>
    <description>indent</description>
</snippet>
```

这里我们使用了输入tab然后按tab键实现输出 `&amp;emsp;&amp;emsp;`，输入什么文字你可以自己定，当然是越简单越容易记越好。
</br>

##记录一下Sublime text3的一些快捷键
* 接下来记录一下ST3的一些实用的快捷键。</br>
* 分屏快捷键，`alt+shift+1~4`这能将屏幕分成纵向的1~4屏。而`alt+shift+5`会将屏幕分成田字格的4屏，`alt+shift+8`会将屏幕上下分屏。 </br>
* GOTO ANYTHING的快捷键，`ctrl+p` </br>
* 打开package control的快捷键，`ctrl+shift+p`</br>
* 复制当前光标所在的一行快捷键，`ctrl+shift+d`</br> 
* 预览MARKDOWN在浏览器中显示，`ctrl+alt+O`</br>
* 新建文件`ctrl+alt+n`</br>
* 复制代码但不失去原有格式`ctrl+shift+v`</br>
