---
title: 一些杂碎的笔记整理 | JS、Linux、Sublime相关
tags: 
  - 前端
  - Nodejs
categories:
  - Web
  - 开发
date: 2016-05-29 15:30:00
---
最近一直没有能够找到一段相对空余的时间来写东西。实际上这段时间也没有闲着，不过记着笔记始终没有发成博客也确实是我越发地懒了。  
话不多说，本次的杂碎笔记整理包括的内容是JS、Linux相关的一些琐碎的点。纯粹是笔记的整理，感觉并不是很好的一篇学习教程。我尽量写得通俗易懂点。  

<!--more-->

## JS相关

首先是个老生常谈但是却又是个很有效的办法：

### 从JS异步回调函数中取值的解决办法

#### 问题描述

```js
function load_val(){
  $.get("url",function(data){
    // 如何把这里取到的data通过load_val函数返回出去？
   });
  });
}
```
如果通过一个全局变量来获取，自然也不是不可以。不过这里就涉及到一点：用全局变量获取后，该怎么使用呢？  
为什么会有这问题，还是上面这个例子，我们稍微改造一下：

```js
var obj = "";
function load_val(){
  $.get("url",function(data){
    obj = data; // 此处将data赋予全局变量
   });
  });
}

load_val();

function use_val(){
  obj += 1;
  console.log(obj);
}

use_val();

```
上面这个例子很好理解，我们想通过`obj`这个全局变量获取ajax异步过来的`data`数据，然后在`use_val`这个函数中使用`obj`这个变量。看似没问题，实际上问题很严重：  
在`use_val()`中的`obj`真的是`data`的值么？不是的。而是`""`。为什么，因为就这段代码而言，`obj = data`是在`use_val()`执行完才在异步回调函数内实现的。在此之前，`obj`一直是`""`。于是又有人说，那我写个延时函数，等待`obj = data`后再执行呗。那样就太不优雅了。那么该如何解决呢？

#### 问题解决

```js
function load_val(callback){//定义一个回调函数
  $.get('url' , function(data){
    callback(data); //将返回结果当作参数返回
  });
}
 
load_val(function(data){
  obj = data; //这里可以得到值
  use_val();
});

function use_val(){
  obj += 1;
  console.log(obj);
}
```
也就是在所需要调用的回调函数外加一个函数，这个函数包含一个参数，该参数是个函数，然而这个函数有着依赖于回调函数给出的值的参数。所以经过这两层，就能将原本回调函数里的值给取出来。

关于js的异步编程，可以参考阮一峰老师的：[Javascript异步编程的4种方法](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)。

### JS中的鼠标滚轮值的获取

在大多数浏览器（IE6, IE7, IE8, Opera 10+, Safari 5+, Chrome）中，都提供了鼠标滚轮事件叫做：`mousewheel`。然而Firefox3.5+不支持这个事件。今天我们先说说非Firefox的情况。

在不考虑火狐浏览器的情况下：

mousewheel事件中，event.wheelDelta的返回值，如果是正值说明是滚轮向上滚动，反之则说明是向下滚动。尤其需要注意，返回值均为120的倍数。即鼠标滚轮真正滚动幅度=返回值/120。

经过测试，用较新的jquery的情况下，要获取wheelDelta的值需要用以下的办法：

```js
$(object).on("mousewheel",function(e){
  var delta =  e.originalEvent.wheelDelta;
})
```
其中那个`object`是你要指定的滚动的元素对象。
同时，为了防止浏览器默认滚动条滚动，还必须做到阻止默认事件。简单来说就是：

```js
$(object).on("mousewheel",function(e){
  var delta =  e.originalEvent.wheelDelta;
  event.preventDefault();
})
```

## Linux相关

### Oh-My-Zsh

Linux默认的shell环境是bash。bash环境实际上已经挺不错的了。但是其实你还可以有更好的一个选择：zsh并与之配对的Oh-My-Zsh。首先zsh也是个shell环境，与bash不一样的地方在于zsh的默认配置比较复杂，因此在很长一段时间内它因为配置复杂的原因让它一直没能够受到大家的青睐。而自从[Oh-My-Zsh](http://ohmyz.sh/)诞生以来，真是能够惊叹：原来我的终端也能如此的棒。  
Oh-My-Zsh是一个开源项目，它重新改变了zsh默认配置复杂的概念，它简化了zsh配置的难度，并且加入很多实用的插件，漂亮的主题，让单调的终端的变得更加富有活力。

安装的方法很简单，官网已经给出了，一行代码就能安装完毕。

Oh-My-Zsh的配置文件是在`~/.zshrc`中。我们需要更改的主题、配置的插件也都在里面可以修改。

其中`ZSH_THEME`是主题文件的修改地方，`plugins=()`是启用插件的地方。

我说说我使用的插件：

#### z

这个插件比起市面上经常提起的`autojump`我觉得更方便——因为是Oh-My-Zsh自带的，不需要安装只需要在`plugins()`里添加一个字`z`就行了。而且效果是一样的。自从用了这个插件，我觉得我已经离不开Oh-My-Zsh了。这个插件有什么用呢？它是一个快速跳转目录的插件。具体来说，它能够记录下你曾经进过的任何目录，比如我在开启了z之后：
`cd /usr/local/bin`，然后下一次我需要进入`/usr/local/bin`的话，我只需要`z bin`然后回车就行了。那如果有同名的文件夹呢？没关系，在按回车之前按下tab就能自动补全路径让你看到进入的是哪个bin文件夹。它能够记录下你进入文件夹的次数由此来排序出优先级。如果你想看优先级，只需要按一下`z`回车就行，就能看到你之前进入过的文件夹并且按照进入次数排好序了。

#### gitfast

Oh-My-Zsh默认开启了`git`插件，它有些很好的简写比如`gco`代表了`git chekout`，然而这还不够，`gitfast`是另外必须开启的。否则你在切换分支的时候，按tab补全的时候会变得特别特别慢。加上这个`gitfast`插件就能解决git在补全的时候卡顿严重的问题。

另外我使用的主题是[ys](http://blog.ysmood.org/my-ys-terminal-theme/)。更多的主题可以参考[这里](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes)。

### Linux Screen命令下 vim中文乱码的解决办法

此处首先是终端软件的编码已经是UTF-8，然后正常情况下VIM的中文显示正常，而Screen下才显示不正常的时候采用的办法。

#### screen配置文件设置

解决办法：在 ~/.screenrc 里加入如下设置: 

```
defutf8 on 
defencoding utf8 
encoding UTF-8 UTF-8 
```

#### 进入screen的命令需要一点更改：

screen -r 重新进去screen的时候，又会产生乱码。
解决办法：
在进入screen的时候加上 -U 参数， screen -U -r xxx 
则能正确显示。


### Sublime相关

#### sublimecodeintel插件自动补全的问题

按`;`键会出现自动补全的提示，如何取消呢？

在WIN下可以这么实现：

sublime的'key bindings - user'里加入：
`{ "keys": [";"], "command": "run_macro_file", "args": {"file": "Packages/User/unAutoSemiColon.sublime-macro"} }`
然后打开sublime的`preferences->browser packages`,然后进入User文件夹，新建一个文件叫做`unAutoSemiColon.sublime-macro`，然后输入如下内容：
```
[{
"args":{
"characters": ";"
},
"command": "insert"
}]
```

然后保存退出。重启sublime。就行了。实际上这里是借助了sublime的宏指令实现的一种办法。

MAC平台下的解决办法可以看这里：https://github.com/SublimeCodeIntel/SublimeCodeIntel/issues/461

#### 绑定快捷键快速开启chrome

WIN下是这样：

在`Preferences`->`Key Bindings`->`User`下，复制以下代码能够按`F1`打开Chrome。
 
```js
{ "keys": ["f1"], "command": "side_bar_files_open_with",
        "args": {
            "paths": [],
            "application": "C:/Program Files (x86)/Google/Chrome/Application/chrome.exe", // 这里的chrome路径需要自己确认。
            "extensions":".*"
        }
}
```

MAC下也基本一样。
