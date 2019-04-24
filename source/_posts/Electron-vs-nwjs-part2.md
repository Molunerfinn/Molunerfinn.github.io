---
title: Electron vs nwjs (part 2)[译]
tags: 
  - 前端
categories: 
  - Web
  - 日志
  - 翻译
date: 2016-02-16 20:30:00
---
写在前头：最近要写点东西，准确来说是用前端技术写桌面应用。之前曾经对Electron有所了解，最近在查阅资料的时候又发现了一个“新的东西”，叫做nwjs——前身是node-webkit。作为一个还未入坑的人来说，到底选择哪个东西作为我们开发的工具会更好呢？我看了这个[作者](http://www.akawebdesign.com/author/arthurakay/)写的两篇文章，发现应该能给另外想要从事同样开发的或者有兴趣的人一些启发。文章不错，而且没发现有中文译版，所以我给出自己的翻译。只是兴趣之作，有错误之处还请指出。  
这是两篇文章中的第二篇。第一篇的翻译在[这里](http://molunerfinn.com/Electron-vs-nwjs/)。
以下是译文：
<!--more-->

------

几个月前我写过一个叫做[Electron vs nwjs](http://www.akawebdesign.com/2015/05/06/electron-vs-nwjs/)的博客，并得到了不少的阅读量。我花费了5个月的时间，用两种办法构建了相当多的应用（明确一下，是不同的应用）并由此我感觉我对于Electron和nwjs有着相当明确的喜欢或者不喜欢的地方——在我现在用这两种办法来制作的应用中也是如此。  
注意：下面我所写的纯粹是因为好玩。我只是一个在互联网上制作软件和吐槽的普通人。我的建议和经历，你可以相信或者不相信，一切取决于你。  

### 开源行为

在我前面的那篇文章里，Electron仍然是我更喜欢的一个项目。对于Electron我有一点尤其喜欢的是它的发行版的更新速度之快——到11月底已经有10个发行版本了！尽管更新速度快但是本身还是很稳定的，并且bug出现后也能被很快地修复。开发团队在GitHub的issues上也一直是很活跃。  

对比之下，到我写这片文章的时候，nwjs只有两个发行版本。一些相当重要的（[IMO](https://github.com/nwjs/nw.js/issues/1440#issuecomment-150635133)）的bugs在v0.12.x版本里还未被修复。

> 译者注：原文发表于2015年11月2日  

发行版更新（频率）的行为虽然并不能作为“更好的选择”的标志，但是考虑到Electron坚持在GitHub上更新的行为，我觉得这是一个很好的理由让你现在的选择倾向Electron。  

Nwjs在GitHub上的文档貌似有点过时了。文档里经常能看到涉及“node-webkit”的字眼，然而这个项目已更名（nwjs）了将近一年了。  

**胜者：Electron**

### 源码保护

从很高的层面上来说，Electron和nwjs几乎是同一个东西。他们都允许你将一个web应用（以及一些Node.js后端的东西）打包成一个独立的桌面应用。当然这两个项目在对于如何得到“上下文”里有着细微的差别，但总体上APIs管理着windows（视窗），menus（菜单）等等。但对于大多数来说，这两个项目中一个唯一显著的区别归结于这个特征：保护你源码的（或是相关的）能力。  

Nwjs 提供了一个功能，[能够将JS源码用v8引擎快照保护起来](https://github.com/nwjs/nw.js/wiki/Protect-JavaScript-source-code-with-v8-snapshot)。Electron则没有这个功能。  

我已经对此和Electron的开发者探讨并[得出了一个结论](https://github.com/atom/electron/issues/3041#issuecomment-150378069)。在我看来，仅这个特征就能够成为开发者为什么选择nwjs而不是Electron的理由了。然而他们貌似对推行这个理念并不感兴趣。  

他们的观点是有他们的理由的——v8引擎快照并不是真正地保护了源码，它只是很好的混淆了代码——并且因此而带来了相当大的性能损失。但是如果（不采用这种方法的话）唯一的替代方法只能是完全清除源代码，开发者将会更愿意选择哪怕只是看起来保护了源码而生成应用的方法。我认为这是一件很容易添加（保护源码功能）的事；他们则不这么认为。至少到目前为止是这样。

**胜者：nwjs**

### 构建过程：Grunt

在我自己的项目里我频繁地使用了Grunt去做那些本地构建的事情——它真的很简单地就能配置，并且它对于你的应用的“产品”版本有着很大的帮助。Electron和nwjs都有着存在在npm和GitHub上的非常有用的Grunt工具包。  

我用在Electron里的包：

- **grunt-exec**：运行命令行任务非常有用
- **grunt-contrib-uglify** 和 **grunt-scram**：还记得Electron里并没有v8引擎快照或者任何机制去保护你的源代码么？那么结合UglifyJS并且混淆你的代码之后至少能够阻止大部分的人窥视你的源代码了。（但这并不是安全的！）
- **grunt-contrib-copy**：在整个文件系统里对于拷贝文件/文件夹很有用
- **grunt-string-replace**：对于更新构建数字和其他类似的东西很有用
- **grunt-electron**：真正意义上的构建过程。能够指定你的应用想定的名字/版本/平台。甚至包括对你的app文件夹进行.asar压缩  

我用在nwjs里的包：

- **grunt-exec**：运行命令行任务非常有用（和我在Electron里用的一样）
- **grunt-contrib-copy**：在整个文件系统里对于拷贝文件/文件夹很有用（和我在Electron里用的一样）
- **grunt-contrib-concat**：对于很多事情都很有用，尤其是在如果你需要拼接许多JS文件，并将它们运用在v8引擎快照之前的时候。
- **grunt-nw-builder**：类似于`grunt-electron`，
对于构建过程，它能够指定你的应用想定的名字/版本/平台

举个例子，创建一个v8引擎快照，你可能需要做如下的事情：

```json
grunt.initConfig({
    exec : {
        nwjc : {
            cwd : 'build/nwjc',
            cmd : 'nwjc app.js app.bin'
        }
    },
 
    concat : {
        nwjc : {
            dest : 'build/nwjc/app.js',
            src  : [
                'app/js/fileA.js',
                'app/js/fileB.js',
                'app/js/fileC.js'
            ]
        }
    },
 
    nwjs : {
        options : {
            platforms : [ 'win32', 'win64', 'osx64' ],
            buildDir  : 'build/platforms',
            version   : '0.12.3',
            //macIcns : '',
            //winIco  : ''
        },
        src     : [ 'build/production/**/*' ]
    }
});
 
grunt.loadNpmTasks('grunt-exec');
grunt.loadNpmTasks('grunt-contrib-concat');
grunt.loadNpmTasks('grunt-nw-builder');
 
grunt.registerTask('default', [
    'concat',
    'exec',
    'nwjs'
]);
```
这里的最后一步（“nwjs”输出地址）以及其他一些内容是在我的构建/成品文件夹里的，所以不要生搬硬套，你懂的。  

我对Electron的Grunt包安装的过程与上面的几乎相同。在个别地方上只有极细微的差别。

**胜者：平手**

### 启动时间

虽然我并没有尝试记录我的应用的精确启动时间，但是我还是得承认采用Electron的应用在视觉上来说不管是在OSX还是在Windows上都启动地更快——不管是在本地开发还是在构建的速度上来说都是这样。  
奇怪的是，我的Electron的应用比起我的nwjs应用来说复杂得多。Nwjs的应用确实是用了v8引擎快照（为此带来了巨大的性能损失），但是连运行没有v8引擎快照的版本的应用都显得启动地更慢。

**胜者：Electron**  

### 你的看法是？ 

请记住：我只是一个在互联网上制作软件和吐槽的普通人。你的情况可能与这些项目有着显著的差异，但是除非你需要确保你源代码的安全，否则我将会毫不犹豫选择Electron。  

如果你已经用这些项目构建了应用，那么我将很乐意去倾听你们的想法以及经历！  

请分享这篇文章！

------

> 原文地址：http://www.akawebdesign.com/2015/11/02/electron-vs-nwjs-part-2/
