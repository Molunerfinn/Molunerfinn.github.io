title: 图床「神器」PicGo v2.0更新，插件系统终于来了
tags: 
  - 前端
  - Vue
  - Electron
  - Electron-vue
categories:
  - Web
  - 开发
date: 2019-01-13 11:30:00
---

## 前言

距离上次更新(v1.6.2)已经过去了5个月，很抱歉2.0版本来得这么晚。本来想着在18年12月（PicGo一周年的时候）发布2.0版本，但是无奈正值研究生开题期间，需要花费不少时间（不然毕不了业了T T），所以这个大版本姗姗来迟。不过从这个版本开始，正式支持插件系统，发挥你们的无限想象，PicGo也能成为一个极致的效率工具。

除了发布PicGo 2.0[本体](https://github.com/Molunerfinn/PicGo/releases/)，一同发布的还有[PicGo-Core](https://picgo.github.io/PicGo-Core-Doc/)（PicGo 2.0的底层，支持CLI和API调用），以及VSCode的PicGo插件[vs-picgo](https://github.com/Spades-S/vs-picgo)等。

<!-- more -->

### 插件系统

PicGo的底层核心其实是`PicGo-Core`。这个核心主要就是一个流程系统。(它支持在Node.js环境下全局安装，可以通过命令行上传图片文件、也可以接入Node.js项目中调用api实现上传。)

`PicGo-Core`的上传流程如下：

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/picgo-core-fix.jpg)

`Input`一般是文件路径，经过`Transformer`读取信息，传入`Uploader`进行上传，最后通过 `Output` 输出结果。而插件可以接入三个生命周期（`beforeTransform`、`beforeUpload`、`afterUpload`）以及两种部件（`Transformer`和`Uploader`）。

换句话说，如果你书写了合适的`Uploader`，那么可以上传到不同的图床。如果你书写了合适的`Transformer`，你可以通过URL先行下载文件再通过`Uploader`上传等等。

另外，如果你不想下载PicGo的electron版本，也可以通过npm安装picgo来实现命令行一键上传图片的快速体验。

PicGo除了`PicGo-Core`提供的核心功能之外，额外给GUI插件给予一些自主控制权。

比如插件可以拥有自己的菜单项：

![](https://i.loli.net/2019/01/12/5c39a2f60a32a.png)

因此GUI插件除了能够接管`PicGo-Core`给予的上传流程，还可以通过PicGo提供的guiApi等接口，在插件页面实现一些以前单纯通过`上传区`实现不了的功能：

比如可以通过打开一个`InputBox`获取用户的输入：

![](https://i.loli.net/2019/01/12/5c39aa4dab0b4.png)

可以通过打开一个路径来执行其他功能（而非只是上传文件）：

![](https://i.loli.net/2019/01/12/5c39aea61e80d.gif)

甚至还可以直接在插件面板通过调用api实现上传。

另外插件可以监听相册里图片删除的事件：

![](https://i.loli.net/2019/01/12/5c39b3c8746cf.png)

这个功能就可以写一个插件来实现相册图片和远端存储里的同步删除了。

通过如上介绍，我现在甚至就已经能想到插件系统能做出哪些有意思的插件了。

比如：

1. 结合GitHub刚刚开放的免费私人仓库，可以通过插件实现PicGo的相册以及配置文件同步。
2. 结合TinyPng等工具实现上传前给图片瘦身。（不过可能挺影响上传速度的。）
3. 结合一些Canvas工具，可以在上传图片前给图片加水印。
4. 通过指定文件夹，将文件夹内部的markdown里的图片地址进行图床迁移。
5. 等等。。

希望这个插件系统能够给PicGo带来更强大的威力，也希望它能够成为你的极致的效率工具。

**需要注意的是，想要使用PicGo 2.0的插件系统，需要先行安装[Node.js](https://nodejs.org/en/)环境，因为PicGo的插件安装依赖`npm`。**

## 2.0其他更新内容

除了上面说的插件系统，PicGo 2.0还更新了如下内容：

- 底层重构了之后，某些图床上传不通过`base64`值的将会提升不少速度。比如`SM.MS`图床等。而原本就通过`base64`上传的图床速度不变。
- 增加一些配置项，比如打开配置文件（包括了上传的图片列表）、mini窗口置顶、代理设置等。
![image](https://user-images.githubusercontent.com/12621342/50515474-ea83c600-0adf-11e9-8022-52f4ab9e0ea5.png)
- 在相册页可以选择复制的链接格式，不用再跑去上传页改了。
![image](https://user-images.githubusercontent.com/12621342/50515502-17d07400-0ae0-11e9-80b9-c38f25b64922.png)
- 增加不同页面切换的淡入淡出动画。
- macOS版本配色小幅更新。（Windows版本配色更新Fluent Design效果预计在2.1版本上线）
- 更新electron版本从1.8->4.0，启动速度更快了，性能也更好了。

## Bug Fixed

- 修复：macOS多屏下打开详细窗口时位置错误的[问题](https://github.com/Molunerfinn/PicGo/issues/128)。
- 修复：多图片上传重命名一致的[问题](https://github.com/Molunerfinn/PicGo/issues/136)。
- 修复：拖拽图片到软件会自动在软件内部打开这张图片的[bug](https://github.com/Molunerfinn/PicGo/issues/140)。
- 修复：重命名窗口只出现在屏幕中央而不是跟随主窗口的[bug](https://github.com/Molunerfinn/PicGo/issues/145)。

## 结语

PicGo第一个稳定版本是在少数派上发布的，详见[PicGo：基于 Electron 的图片上传工具](https://sspai.com/post/42310)。支持macOS、Windows、Linux三平台，开源免费，界面美观，也得到了很多朋友的认可。如果你对它有什么意见或者建议，也欢迎在[issues](https://github.com/Molunerfinn/PicGo/issues)里指出。如果你喜欢它，不妨给它点个star。如果对你真的很有帮助，不妨请我喝杯咖啡（PicGo的GitHub[首页](https://github.com/Molunerfinn/PicGo)有赞助的二维码）？

> 下载地址：https://github.com/Molunerfinn/PicGo/releases

> Windows用户请下载`.exe`文件，macOS用户请下载`.dmg`文件，Linux用户请下载`.AppImage`文件。

Happy uploading！






