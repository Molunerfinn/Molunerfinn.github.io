---
title: PicGo的star数破1000的心路历程
tags: 
  - 前端
  - Nodejs
  - Electron
  - Electron-vue
  - Vue
categories:
  - Web
  - 开发
date: 2018-06-14 15:28:00
---
大概半年前（2017年11月28日）我在GitHub上开源了一个基于[electron-vue](https://github.com/SimulatedGREG/electron-vue)的开源桌面应用[PicGo](https://github.com/Molunerfinn/PicGo)。其出发点是为了改善我在写博客的时候贴图困难的问题。在经过了半年的持续维护和一些宣传（[《PicGo：基于 Electron 的图片上传工具》](https://sspai.com/post/42310)、[《图床上传工具PicGo v1.5更新：支持腾讯云COSv5版本、支持GitHub图床、支持上传前重命名文件等等》](https://sspai.com/post/44495)等等）后，6月12日，它的star数也终于突破了1000的关卡。在这过程中我也学习了不少东西。在和大家交流的过程中，我才发现原来大家都有着这些需求，才发现我一开始的实现思路并非到位等等。谨以此文记录与PicGo有关的我的心路历程。

> 赶巧前不久也有一个开发者chyingp的开源项目破了1000star，也有着类似的[文章](https://juejin.im/post/5b1717a86fb9a01e3e5ce540)，祝贺！

![](https://ws1.sinaimg.cn/large/8700af19ly1fs892cewamj21ks0emq5n.jpg)

<!--more -->

## 项目诞生

我以前写博客的时候，由于一开始用的是七牛的图床，所以遇到要在markdown里贴图的时候就必须登录七牛，然后手动上传图片，再找到按钮来复制链接，然后复制到markdown里。要在markdown里显示一张图片我得经过上述4个步骤。

我自己的笔记本用的是mac，所以我就接触到了一款叫做[iPic](https://toolinbox.net/iPic/)的图床神器。在用它的时候我也知道了微博图床。iPic的功能和体验真的特别好。不过如果需要使用七牛等其他图床的时候，我就需要付费了。其实如果iPic支持windows的话，我可能就真的去付费了。（因为实验室的电脑是windows，所以我平时在实验室里得用windows来写东西）。仅为mac一个平台付费我有点不能接受。

于是我就在想能否我自己写一个工具来简化我的上传图片的流程呢，这个应用可以实现拖拽图片就上传，然后上传完自动复制链接到剪贴板里，我就只要粘贴到markdown里就好了。一开始想用swift写mac的应用，用C#写windows的应用。后来发现工作量不仅大，而且学习成本也很高。于是最后还是选择投入electron的怀抱。

一开始不选择electron主要是因为我的印象中electron的应用体积都挺大的（100MB以上），所以感觉体积可能有点不友好。不过后来我在用了`electron-vue`打包出来后发现体积是可以接受的范围（mac端大概50M，windows端大概38M），于是就决定用它来写了。用`electron-vue`的主要原因是我写vue比较多，想要学习成本低一些，这样开发只要学electron的部分就好了。

说干就干，在去年年底（11月下旬）我开始写这个应用。

## 项目发布

> 文末会给出electron-vue开发的系列经验教程

经过20多天的间断性地开发，我在12月12号发布了1.0.0版本。由于一开始是在mac上开发的，所以1.0.0版本也只支持了macOS。一开始支持的图床也不多，只支持了微博和七牛两个图床。

1.0.0版本的截图如下：

![](https://user-images.githubusercontent.com/12621342/34242310-b5056510-e655-11e7-8568-60ffd4f71910.gif)

![](https://user-images.githubusercontent.com/12621342/34242857-d177930a-e658-11e7-9688-7405851dd5e5.gif)

基本实现了我预期的功能，类似iPic能够通过拖拽到顶部栏图标上传。并且为了今后支持windows平台（windows平台的任务栏图标不支持拖拽事件），我就做了一个主窗口，在主窗口里也有拖拽上传的区域。因为有了主窗口，我就顺便把图床的配置也放到了主窗口里。

应用做出来了，我也想让更多的人用到。于是我在北邮人论坛、cnode、v2ex还有掘金都发了文章。不过一开始看到的人寥寥无几，发了文章也没多少人看到和使用。后来我在少数派上发了同样的文章，意外地被推荐到了首页。

![](https://ws1.sinaimg.cn/large/8700af19ly1fmvr6uah8rj21z20vk7wh)

这次的契机让PicGo意外地有了些用户和star数。在跟使用者交流的过程中我也开始逐步往PicGo里加功能和修复bug。在1月10日的时候，PicGo更新v1.3.1版本支持了windows系统。

因为开始有用户了，PicGo早期确实存在着不少功能的缺失，比如`快捷键上传`，其他常用图床的缺失等等。所以那时候是PicGo迭代最快的一段时间。通过大家在issue里的反馈，我也在不断打磨PicGo。可以看到截止6月14日，已经有61个被关闭的issue了。

![](https://i.loli.net/2018/06/14/5b2223c52853f.png)

## 项目改进

用户体验这个东西真的并不是开发者在开发的时候能够立马就想到的。这点在开发PicGo上我体会很深。

比如增加`快捷键上传`这个功能，我一开始觉得自定义快捷键写起来比较麻烦，干脆我定一个大家基本用不到的快捷键吧。于是我默认给了一个【command/ctrl+shift+p】的快捷键。我自己用的时候没有什么问题。结果有人给我反馈说，快捷键跟某某某软件冲突了，能否给一个快捷键自定义的功能。这是我无法回避的一个问题。于是我就开始去学习如何加入自定义快捷键。并在不久之后实现了个这个[功能](https://github.com/Molunerfinn/PicGo/commit/37a784225e90c9d115367f056957dac88ebcf816)。

比如自定义链接格式的问题。我一开始给了大家4种复制链接的格式，分别是`markdown`、`HTML`、`URL`、`UBB`。本来以为这4种格式就足够大家平时使用了。后来有人提了一个[issue](https://github.com/Molunerfinn/PicGo/issues/25)，问PicGo能否自定义链接格式，因为他想基于HTML增加一些属性，比如大小居中等。我觉得这个使用场景确实是有的，于是我便在后来的某个[提交](https://github.com/Molunerfinn/PicGo/commit/4010a09fe48d8109456c3c1b37695f177336f2e4)里实现了这个功能。

当然并不是大家有这个需求我就一定要做。还有一些需求我觉得并不符合我对于PicGo的定位的，那么我就会给予回绝。比如[后期能否支持上传视频文件？](https://github.com/Molunerfinn/PicGo/issues/53)，由于PicGo的开发初衷只针对图片，所以在流程上（图片->base64）就不允许上传视频文件。于是我拒绝了这个需求。

还有一个对我以及PicGo这个项目影响深远的[issue](https://github.com/Molunerfinn/PicGo/issues/26)，ZetaoYang提出了一个想法：

![](https://i.loli.net/2018/06/14/5b2228f31219a.png)

这个建议改变了我对PicGo开发的后续想法。我思考了好久，发现确实一步步增加默认的图床支持是不长远的。一个是重复性劳动太多（图床上传除了协议和加密方式不同之外，接收文件，转成base64和最后上传成功后存到本地的流程是一样的），一个是无止尽的图床支持其实也不应该。相比之下，把PicGo做成一个Core+Plugin模式的应用会更好。其中Core的部分可以单独只做图片接收和转码，并预留一些生命周期，供上传过程中不同的需求来调用。Core的部分可以单独发布成一个npm包。Plugin可以实现接入Core的生命周期，可以实现自己的上传逻辑，可以实现图片压缩、加水印等等其他功能。而PicGo只是在Core+Plugin的基础上套了一层electron的皮方便普通用户使用，而Core和Plugin可以独立拆出方便开发者使用和开发。这个也是PicGo的2.0版本将要做的事。

## 总结

在开发PicGo的过程中我也深刻了解到，写一个DEMO不难，给这个DEMO注入你自己的思想和灵魂是难的。PicGo从一个一开始只是我想简化上传图片流程的玩具应用，发展到现在已经是不少用户的效率工具而言，其实一路走来也并不容易。现在大家对用户体验的要求越来越高，如果只沉醉在自己的DEMO里无法自拔，只会被更好的产品所淘汰。

开发PicGo也是一件很开心的事。大家给予我的赞赏和感谢，都是给我继续开发的动力。而我也发现越来越多的文章里，都提到了PicGo。如下：

- [《老司机的神兵利器-效率工具》](https://juejin.im/post/5af0021e518825671547926e)
- [《PicGo 强大的免费图床工具》](https://imwyc.com/picgo/)
- [《PicGo：开源的图片管理工具》](https://lai.yuweining.cn/archives/2035/)
- [《图床神器PicGo》](https://blog.csdn.net/weixin_39200308/article/details/80644336)
- [《提升生活品质——个人效率工具与资讯网站推荐》](https://zhuanlan.zhihu.com/p/37873730)
- [《7 款 Windows 国产软件推荐》](https://sspai.com/post/44150)

我想，得到你们的认可，把它写进你们的文章，这是对我最大的肯定，这个比star数更令我感到开心。

我在开发PicGo的过程中，也写了一个系列文章[Electron-vue开发实战](https://molunerfinn.com/tags/Electron-vue/)。如果你也想学习electron或者electron-vue的开发的话，希望我的文章能够给你带来帮助。如果你之前没有听说过PicGo，那么不妨试试；如果你觉得它挺好用的，不妨点个star~