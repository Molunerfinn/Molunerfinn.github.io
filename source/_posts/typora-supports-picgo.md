---
title: Typora 支持 PicGo 来上传图片了
tags: 
  - 前端
  - Vue
  - Electron
categories:
  - Web
  - 开发
  - 随笔
date: 2020-03-01 21:25:00
---

Typora 最近的一次更新支持图片自定义图片上传服务了，增加了对 [uPic](https://github.com/gee1k/uPic)，[PicGo](https://github.com/Molunerfinn/PicGo) 以及自定义上传命令的支持。其中针对 PicGo 和 PicGo-Core 都做了兼容，可以说非常有诚意了。本文会简单介绍一下如何配置并使用。

<!-- more -->

## 自定义图片上传服务的设置

更新 Typora 的最新版，可以在设置-图像处找到自定义图片上传服务的设置区域：

![image-20200226105152275](https://cdn.jsdelivr.net/gh/Molunerfinn/test/blog/typora-image-setting.png)

Typora 官方关于图像自定义上传相关的配置、介绍的页面 [在这里](https://support.typora.io/Upload-Image/)。

如上图，你可以选择自己喜欢用的图片上传工具，可选的工具如下图：

![image-20200226114210451](https://cdn.jsdelivr.net/gh/Molunerfinn/test/blog/upload.png)

同时 Typora 提供了上传测试功能，如下图你可以找到 `Test Uploader` 按钮来测试你的上传功能是否正常：

![image-20200226114658030](https://cdn.jsdelivr.net/gh/Molunerfinn/test/blog/typora-test-upload.png)

Typora 会上传的图片就是它家的 Logo 了，如下：

![image-20200226192722744](https://cdn.jsdelivr.net/gh/Molunerfinn/test/blog/image-20200226192722744.png)

当测试成功之后，还别忘了开启图片自动上传功能：

![image-20200226120345146](https://cdn.jsdelivr.net/gh/Molunerfinn/test/blog/typora-image-settings.png)

注意选则第三项，即允许通过读取 YAML 配置来决定是否自动上传图片。经过测试，在 macOS 上必须开启这个选项，同时在文章的顶部写下如下的 YAML 配置：

```yaml
---
typora-copy-images-to: upload
---
```

这样才可以开启自动上传图片的功能。应该是 Typora 的一个 bug，后续版本不知道会不会修复。

## 自动上传图片的效果

说了这么多，Typora 里引入图片即上传的效果是怎么样的呢？我录制了一个 gif：

![typora-upload-image-gif-v2](https://cdn.jsdelivr.net/gh/Molunerfinn/test/blog/typora-upload-image-gif-v2.gif)

可以说整体效果还是比较流畅的。

同时如果你未开启自动上传图片的功能，把图片拖入 Typora 或者粘贴到 Typora，右键图片看到一个上传图片的选项：

![upload-image-with-context-menu.png](https://cdn.jsdelivr.net/gh/Molunerfinn/test/blog/upload-image-with-context-menu.png)

这样也能根据你配置的上传服务来上传图片。

## 使用 PicGo 上传的相关说明

Typora 支持了两种 PicGo 的上传模式，作为 PicGo 的开发者，我觉得有有必要跟朋友们说说区别。Typora 支持的两种 PicGo 上传模式分别是：PicGo-Core（命令行）以及 PicGo.app（图形界面）

### 1. PicGo.app

[PicGo.app](https://github.com/Molunerfinn/PicGo) 就是用户平时经常使用的图形化界面的 PicGo。而 Typora 对接的上传服务来自于 PicGo v2.2.0+提供的 [PicGo-Server](https://picgo.github.io/PicGo-Doc/zh/guide/advance.html#picgo-server) 的功能，它是一个小型的 HTTP 服务器，会默认开启 36677 端口来监听上传的请求。而 Typora 则会往 36677 端口发送请求来上传图片。所以如果你的 PicGo 版本过低或者 PicGo-Server 功能没有开启，或者端口不是 36677，都无法通过 Typora 的这个功能上传图片。

### 2. PicGo-Core

这个是 PicGo 底层依赖的 [核心库](https://github.com/PicGo/PicGo-Core)，是 PicGo 上传图片、插件机制的核心。它是一个 npm 包，意味着你可以通过 npm 全局安装来实现上传。同时 Typora 也提供了预编译的二进制文件，它是把 PicGo-Core 所有依赖都打包成了一个可执行的文件。

Typora 对这两种 PicGo-Core 的用法都支持，官方的文档对此有详细的 [配置说明](https://support.typora.io/Upload-Image/#config-picgo-core)。不过需要注意的是 macOS 由于系统的原因，不支持预编译的二进制文件那个使用方法，而只能使用 npm 全局安装的方式，再通过 `custom command` 自定义命令的方式来使用 PicGo-Core：

![custom](https://cdn.jsdelivr.net/gh/Molunerfinn/test/blog/custom.png)

### 3. 二者的区别

官方 [文档](https://support.typora.io/Upload-Image/#difference-between-picgoapp-and-picgo-core-command-line) 里对二者的区别有做出描述，我觉得写得挺到位的。不过还是跟大家聊聊这二者的区别：

1. 使用 PicGo.app 模式上传意味着 PicGo 需要开启常驻后台。如果对性能要求比较高的用户可能不太能接受。
2. 用 PicGo-Core 来上传只有运行时的消耗，上传结束后会自动销毁进程，性能方面会更好。
3. PicGo-Core 上传的配置跟 PicGo 用的不是同一个文件，因此如果需要用 PicGo-Core 来上传需要重新配置一遍。
4. PicGo 提供了更多的功能，比如上传前重命名、上传的历史记录等
5. PicGo 的一些插件只有 GUI 版本支持，而不支持 PicGo-Core，所以如果需要使用插件功能，更推荐使用 PicGo。不过 PicGo 只在语言设定为中文版的 Typora 里才能使用，因为目前 PicGo 没有英文文档、英文界面。

**跪求 T T 有兴趣的小伙伴一起来翻译，如果对 PicGo 的国际化有意向的小伙伴，可以加入官方 [gitter](https://gitter.im/picgo-all/PicGo?utm_source=share-link&utm_medium=link&utm_campaign=share-link) 频道一起来聊。**

就我自己的使用来说，我是更喜欢直接用 PicGo 来上传的，因为配置什么的不用再调了，可视化界面也更容易操作~

## 小结

前不久 PicGo2.0 发布的时候，PicGo-Core 还收到了来自 Typora 官方的 PR。我以为需要好几个月的时间才能支持自定义图床，没想到支持来得这么快。我觉得对于一个 Markdown 编辑器而言，图片的管理、上传一定是一种刚需。而此次开放了自定义上传的功能，想必也是戳中了很多 Typora 用户的痛点。另外这次 PicGo 能够作为官方指定的上传工具，我觉得非常开心，同时它也是 Typora 三个平台都支持的上传工具（uPic 和 iPic 都很棒，不过只支持 macOS），希望有了这个功能以后能够给你们带来更好码字体验~
