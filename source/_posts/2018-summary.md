---
title: 2018小结
tags: 
  - 随笔
categories:
  - 日志
date: 2019-01-18 09:22
---

终于把研究生开题的事情弄得差不多了，可以抽空写一下2018年的小结了。

<!-- more -->

今年和去年一样，也是格外忙。不仅实验室活多，还要兼顾研究生的开题等。跟去年一样，列一个今年学习成果清单：

# 过去的一年

## 技术成果

- **2019.01.13**（插播） [PicGo](https://github.com/Molunerfinn/PicGo) 发布v2.0版本，正式支持插件系统。star数破3200，下载量破26k。【Electron】

- **2018.08.28** [PicGo](https://github.com/Molunerfinn/PicGo) star数破2000，下载量破12k。【Electron】

- **2018.07.19** [PicGo-Core](https://github.com/Molunerfinn/PicGo-Core) 开坑PicGo底层流程系统，将支持插件系统【Node+TypeScript】

- **2018.07.11** [PicGo](https://github.com/Molunerfinn/PicGo) 更新v1.6版本，支持阿里云OSS，imgur，mini窗口，批量删除等功能。【Electron】

- **2018.05.23** 为VSCode的[amVim-for-VSCode](https://github.com/aioutecism/amVim-for-VSCode)插件提交的支持`:`呼出`Command Palette`并实现部分Vim命令的[PR](https://github.com/aioutecism/amVim-for-VSCode/pull/199)被合并。【TypeScript】

- **2018.05.17** [PicGo](https://github.com/Molunerfinn/PicGo) star数破800，下载数破5k。【Electron】

- **2018.05.15** 开发推来推趣3期后台时遇到微信二维码支付相关功能的开发，总结了一篇[《基于Koa2开发微信二维码扫码支付相关流程》](https://molunerfinn.com/koa2-wechatpay/)的经验文。【Koa】

- **2018.05.09** [PicGo](https://github.com/Molunerfinn/PicGo) 更新v1.5版本，支持腾讯云COSv5、GitHub图床、重命名等新功能。【Electron】

- **2018.03.28** [node-github-profile-summary](https://github.com/Molunerfinn/node-github-profile-summary)和[vue-koa-demo](https://github.com/Molunerfinn/vue-koa-demo)的Docker话。【Docker】

- **2018.03.10~2018.05.31** 推来推趣3期后台（全栈）迭代。【Vue+Koa+Graphql】

- **2018.03.06** [hexo-theme-melody](https://github.com/Molunerfinn/hexo-theme-melody) 更新v1.5版本，支持iframe、支持slides等特性。【hexo+hexo-theme】

- **2018.01.17~2018.03.28** 开坑[node-github-profile-summary](https://github.com/Molunerfinn/node-github-profile-summary)，可以生成漂亮的GitHub总结报告。【Vue+Koa+Chart.js+Graphql】

- **2018.01.11~2018.05.08** 写了[Electron-vue开发实战系列教程](https://molunerfinn.com/tags/Electron-vue/)，用于记录自己开发PicGo的坑以及帮助新人入门Electron开发。【Electron】

对比去年给自己立的目标：

- 算法、数据结构 【一部分】
- Parcel 【没有】
- TypeScript 【用上了】
- Puppeteer自动化测试 【没有】
- PWA 【有新的体验】
- 给开源库提PR 【完成】
- github robot 【没有】
- 如果可以，学习一下react 【碰了皮毛】

感觉完成度不够高，不及去年同期对2016年的目标的实现。主要是没有预料到下半年研究生的开题的战线耗时这么久。从2018年8月开始我就没有发过笔记或者技术文章了，真的非常惭愧。

# 期望、目标

依然要写下2019年需要学习的东西：

- 算法、数据结构
- Flutter入门
- PWA
- 学习react
- Puppeteer使用

感觉把目标缩小点应该完成度会更高。毕竟19年要开始找实习和正式工作+写研究生毕设了。

# 小结

这一年来的前端的学习之路，收获还是不少的。比起2017年来说，我感觉最大的收获就是阅读源码的能力提高了。虽然不是什么高深的源码，不过相比之前对阅读源码有恐惧心理的自己，还是好了不少。

5月份的时候，那段时间我的Mac上的VSCode的Vim插件变得异常卡，可以参考这个[issue](https://github.com/VSCodeVim/Vim/issues/2021)。无奈之下只能把官方的Vim插件替换掉，换成了[amVim-for-VSCode](https://github.com/aioutecism/amVim-for-VSCode)，当初刚换上的时候，操作如丝般顺滑！不过当时发现它不支持`:`带来的一系列操作，比如`:w`保存，`:q`退出等。于是我萌生了一个想法，能不能把VSCodeVim的操作移植到amVim上？在阅读了VSCodeVim的源码之后，我也模仿了它的实现，把一部分常用的命令移植到了amVim上，并最终成功被作者[合并](https://github.com/aioutecism/amVim-for-VSCode/pull/199)。

这次提交PR的过程，我也发了一篇[文章](https://molunerfinn.com/vscode-extension-develop-1/)作为记录。应该说这次经历过后我对阅读源码的恐惧感减轻了不少，这也为之后的[PicGo-Core](https://github.com/PicGo/PicGo-Core)的开发带来很大的帮助。

8月份之后很长的一段时间里，除了在做研究生开题相关的东西，我基本就是在开发[PicGo-Core](https://github.com/PicGo/PicGo-Core)了。如果你有用过[PicGo](https://github.com/Molunerfinn/PicGo)，那么你应该知道它的1.x版本是不支持插件系统的。而且内置的只有有限的8个图床。（如果你不知道[PicGo](https://github.com/Molunerfinn/PicGo)，欢迎使用，对你的文章写作有很大帮助~）。`PicGo`里我收到最多的issue，应该就是`能否支持XXX图床`。如果是一开始写PicGo的时候，我一般会在下一个版本里更新新的图床支持。但是支持到第8个的时候我发现这样无限地支持下去不是一个办法。正巧有个用户提出一个[想法](https://github.com/Molunerfinn/PicGo/issues/26#issuecomment-370105520)：能否将对各种图床的支持，做成插件化的管理，类似 Core + Plugins 这样的模式。

我为此思考了好久，发现这样是可行而且非常合理的。于是我开始找相关的资料——我一开始的想法只是在Electron内部实现一个插件系统。为此我去找了不少例子，比如VSCode、Kap、Atom、Hyper等用Electron写的工具，想看看他们的插件系统是如何实现的。发现他们的实现相对比较复杂。对我来说我是想要实现一个底层的上传流程系统。

后来我想到了Hexo也是有插件系统的，于是就去阅读了Hexo的插件系统如何实现。在看Hexo插件系统实现的同时，我还发现了另外一个工具[feflow的插件系统实现](https://segmentfault.com/a/1190000013362598)。不过我后来发现，feflow的插件体系其实底层大部分是「抄」的hexo的源码的，尤其一个很经典的例子...

![20190118101110.png](https://i.loli.net/2019/01/18/5c4135bf942d9.png)

于是我就把feflow的文章当做hexo插件系统实现的解析文章了哈哈。

在充分理解了hexo插件如何实现了之后，我也开始着手我自己的[PicGo-Core](https://github.com/PicGo/PicGo-Core)了。当然我并没有完全照搬hexo的实现，因为我发现那样的话不利于插件开发者开发插件（主要是语法提示），hexo的插件机制是暴露全局的`hexo`变量去实现的。

`PicGo-Core`的流程大概如下：

![flow](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/picgo-core-fix.jpg)

输入路径或者变量等->经过转换器转换->上传器上传->输出结果。中间包含着三个生命周期钩子。这样的话用户开发插件可以只实现其中的某个部分，也可以实现其中的某几个部分，来实现`PicGo`原先不能实现的一些功能：

1. 比如上传非图片文件
2. 比如上传图片前压缩、加水印
3. 比如通过已知URL上传图片

等等。

我也正式使用了`TypeScript`作为`PicGo-Core`的开发语言，使用起来一开始确实很不习惯，但是后来越用越顺手，学习新东西的过程大概都是这样吧！

在开发`PicGo-Core`的过程中，我也做了很多除了上面流程系统之外的工作。比如：

1. 要让用户在命令行和Node里都能使用，我为此基于[commander.js](https://github.com/tj/commander.js/)和[Inquirer.js](https://github.com/SBoudrias/Inquirer.js/)给`PicGo-Core`加上了命令行支持，同样插件也能支持注册命令等操作。
2. 为了方便其他开发者开发插件，首先我得写好一个插件模板[picgo-template-plugin](https://github.com/PicGo/picgo-template-plugin)，并学习了`vue-cli2`和`vue-cli3`对于模板生成的实现，写了一个下载模板、生成模板的命令[init](https://github.com/PicGo/PicGo-Core/blob/dev/src/plugins/commander/init.ts)，好让插件开发者能够快速创建插件模板进行插件开发。
3. 为了让使用者方便下载使用插件，我写了一个[PluginHandler](https://github.com/PicGo/PicGo-Core/blob/dev/src/lib/PluginHandler.ts)用于调用`npm`命令来下载插件。
4. 除了写代码，还得写文档，没有文档怎么能有其他开发者为你开发插件呢？所以还花了很大的精力写了`PicGo-Core`的[文档](https://picgo.github.io/PicGo-Core-Doc/zh/)，配图、示例一应俱全。

开发完Node版本的`PicGo-Core`之后，我还要将它和Electron版本的`PicGo`整合起来，使得Electron版本的`PicGo`也能拥有插件系统。并且还得通过`ipcMain`等方式，将主进程的信息通知给渲染进程，从而渲染出插件页面里的插件列表：

![](https://user-images.githubusercontent.com/12621342/50515434-bc9e8180-0adf-11e9-8c71-0e39973c06b1.png)

为了让插件开发者能够更好地利用GUI版本的优势，我还为GUI版本的PicGo插件加了GUI插件特有的`guiApi`、`guiMenu`等功能：

![](https://i.loli.net/2019/01/12/5c39a2f60a32a.png)

这样插件拥有自己的菜单，可以执行自己的操作，那么能做的事就更多了，比如：

1. 结合GitHub刚刚开放的免费私人仓库，可以通过插件实现PicGo的相册以及配置文件同步。
2. 结合TinyPng等工具实现上传前给图片瘦身。（不过可能挺影响上传速度的。）
3. 结合一些Canvas工具，可以在上传图片前给图片加水印。
4. 通过指定文件夹，将文件夹内部的markdown里的图片地址进行图床迁移。

等等。。

终于，在2019年1月13号，PicGo迎来了2.0版本的[更新](https://github.com/Molunerfinn/PicGo/releases/)。

回顾这些工作，都是我一个人在半年的时间里通过课余的时间做出来的，其实还是很自豪的。更关键的是，通过开放了插件系统，可以让更多的人参与到PicGo软件的完善中来，通过插件可以实现很多本体不提供或者不足的功能，也是让PicGo更加强大的一个条件。我也希望它日后也能形成自己的一个小生态。

实际上，PicGo-Core以及PicGo2.0发布之后，就已经有第三方开发者开发插件了，速度之快让我始料未及。为此我也迅速加上了[Awesome-PicGo](https://github.com/PicGo/Awesome-PicGo)的仓库，这样能让更多的开发者的作品让用户看到：

![20190118103930.png](https://i.loli.net/2019/01/18/5c413c6300681.png)

你已经可以在VSCode里搜索PicGo，就能发现VSCode版的PicGo扩展了，实现了三种在Markdown里快速上传图片的方式：

- 通过截图上传

![](https://camo.githubusercontent.com/e7898449cadc72bb7045319e4195a5210fef60cf/68747470733a2f2f692e6c6f6c692e6e65742f323031392f30312f31362f356333656430333335373761302e676966)

- 通过文件浏览器上传

![](https://camo.githubusercontent.com/955c32665b55b1ac85ec9696cc51fddcb740076d/68747470733a2f2f692e6c6f6c692e6e65742f323031392f30312f31362f356333656433366430643964332e676966)

- 通过输入文件路径上传

![](https://camo.githubusercontent.com/f2cb528b4fcca4e64f6e6bf80d1e25ea47b85483/68747470733a2f2f692e6c6f6c692e6e65742f323031392f30312f31362f356333656432333836623761632e676966)

2019年，我会更新几篇文章，主要讲讲如何实现一个插件系统，如何将Node端实现的插件系统整合到Electron端，如何实现一个模板下载、生成功能，如何实现良好的命令行交互等等。

2019年也是我找实习、找正式工作的一年，希望今年一切都顺利吧！
