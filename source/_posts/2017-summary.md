---
title: 2017小结
tags: 
  - 随笔
categories:
  - 日志
date: 2017-12-27 21:01
---
年末了，赶着刚考完两门考试，在最后4门考试来临之前抽空写一下今年的小结。

<!-- more -->

今年格外忙。忙完本科毕设，又马上投入了研究生实验室的搬砖生涯。跟去年一样，列个今年的学习成果清单：

# 过去的一年

## 技术成果

**2017.03~2017.05.07** 开坑学习Three.js，完成了一个简单的[机械装置展示平台](https://github.com/Molunerfinn/Gear-system)（我的本科毕设）【Three.js+dat.gui】

**2017.05.23~2017.07.15** 基于vue2+koa2重构了[福建北邮人服务系统](https://fj.teamsz.xyz/)，这是我自己的项目。开始引入eslint（以前嫌麻烦233），以后的项目也一并引入。期间在手写一些常用Vue组件的时候学习了不少东西，写了一篇[Vue组件的三种调用方式](https://molunerfinn.com/vue-components/)【Vue2+Koa2】

**2017.05.26** 为了上面那个项目简单做了一个基于`stylus`的栅格系统css——[Melody.css](https://github.com/Molunerfinn/Melody.css)，用来快速做响应式开发。【stylus】

**2017.06.07** 协助解决实验室Vue项目里webpack的Hot Reload速度太慢的问题，做了个webpack的开发模式的插件[webpack-dev-compile-optimize](https://github.com/Molunerfinn/webpack-dev-compile-optimize)提升热重载速度（只在自己内部项目测试过），同期总结了一篇基于[vue-cli项目的webpack构建优化文章](https://molunerfinn.com/Webpack-Optimize/)。【webpack】

**2017.07.07** 博客开启持久化构建，依赖于github-page，不过加上了https以及进入了HSTS列表。第一次接触了[Travis-CI](https://travis-ci.org/)，发表了一篇[经验文](https://molunerfinn.com/hexo-travisci-https/)。【Travis-CI】

**2017.08.09** 开坑[hexo-theme-melody](https://github.com/Molunerfinn/hexo-theme-melody)，写一个送给妹子的hexo主题，效果见[我博客](https://molunerfinn.com)即是。【hexo hexo-theme】

**2017.10.09** 写每周电影推荐的时候因为嫌弃获取电影信息步骤繁杂，于是改造了一下早期写的node小爬虫[dbmovie-spider](https://github.com/Molunerfinn/dbmovie-spider)支持读取命令行信息了。【node】

**2017.10.28** 开始[练习算法](https://github.com/Molunerfinn/FE-Learning)，并借机学习TypeScript和前端测试（采用了Jest）。 不过后来一直有其他事压着，没有持续，等考完试要继续。【TypeScript Jest】

**2017.11.02** 开坑[vue-koa-demo](https://github.com/Molunerfinn/vue-koa-demo)项目的前端测试。同期写了一篇[Jest 全栈测试的经验](https://molunerfinn.com/Use-Jest-To-Test-Vue-Koa/)博客。【Jest】

**2017.11.18** 开坑[PicGo](https://github.com/Molunerfinn/PicGo)，学习electron的基本开发流程，边写边学。最终完成了一个我现在写博客贴图片时很方便的工具。并于12月中发布正式版，还上了少数派首页推荐。【electron】

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fmvr6uah8rj21z20vk7wh)

> PS，在掘金也发了一遍[推荐](https://juejin.im/post/5a30e4755188256e7a06cc3e)不过没有被推荐到首页T T

之后应该会发几篇electron开发的文章。

**2017.11.30** 抽空把vue-koa-demo的[ssr](https://github.com/Molunerfinn/vue-koa-demo/tree/ssr)版本做了一下。踩了一些ssr的坑。

对比去年给自己立的目标：

```
**算法**

**数据结构**

**Three.js -> 浏览器3D建模**

**回归JS语言基础**

**学会玩Webpack2**

**持续的项目开源**

**Python简单入门**
```

感觉除了Python没怎么学之外(尴尬)，其他的目标大致都有所建树，算是完成地还不错吧！

## 期望、目标

依然要写下2018年需要学习的东西：

- 算法、数据结构
- Parcel
- TypeScript
- Puppeteer自动化测试
- PWA
- 给开源库提PR
- github robot
- 如果可以，学习一下react

## 随笔

这一年来的前端的学习之路，收获还是不少的。比起去年来说，我自己觉得收获最大的就是在开源社区跟开发者、使用者的交流更多了。因为自己也有开源项目，所以很多时候一些情况也是第一次见：比如第一次遇到PR（开心不已），第一次给开源库提issue，第一次跟开发者讨论项目细节等等。今年还没有给开源库提过PR，所以明年的目标是来一个吧~

今年也是前端框架、库井喷的一年。各种新的技术涌现、较新的技术逐渐走向成熟、成熟的项目走向稳定。这种感觉似乎从我两年半前学习前端的时候就有了，不过今年真的特别强烈。也因此才有那篇流传甚广的《2017年学JavaScript是怎样的一种体验》。前端要学的东西太多了啊。不过我觉得虽然看似多，作为前端工程师，还是要有自己的大体学习路线。

我认为如今前端工程师应当分成两类，

1. 结合Node的偏向全栈的前端，他们更注重网站的访问优化、性能提升、毫秒级别的用户体验。
2. 结合CSS\JS的偏向用户端特效的“纯”前端工程师。这部分的前端工程师通常来说必须要有自己的设计认知。

很多优秀的前端工程师都是设计师出身。比如TJ，比如尤雨溪。但是却不是很常听说优秀的设计师是前端工程师出身。这就是因为现在很多学前端的人还是在认为自己能够写个页面、套个模板，厉害点的还原个页面就行了。殊不知，你要学习的不仅仅是前端配套的HTML\CSS\JS，你还需要知道结合了Nodejs后带来的一系列现代开发工具和工程化的流程。不再是只会用个bootstrap+jquery做个页面就完事的年代了。刀耕火种的年代已经过去，可是还是有人在抓着旧石器不放。

不过还是需要强调一下，基础真的很重要。我身边遇到太多半路“出家”，自愿也好，被迫也罢来学前端的同学，他们很多都是草草几天看完HTML\CSS\JS基础，然后就直接用上Vue、React来写项目了。连npm都不知道是什么东西的他们，很多时候写起前端来非常痛苦。前端不再是以前那样认为的是一门可以速成的技术了啊，现在而言，至少入门门槛高了不少。

前端圈还是太浮躁了点。还是沉下心来，好好钻研自己喜欢的技术吧。

另外，由于最近出现的诸如PWA、Electron、RN、微信小程序等由前端主导的新技术，很多人就说了“啊iOS开发要完啦”、“啊安卓开发要完啦”、“要转行前端啦”等，我觉得其实还没有必要恐慌到那个程度。诚然如今前端能做的事不少，但是局限性还是很强。PWA由于依赖高版本Chrome在一般安卓机器上体验依然不怎么样，想做出像原生一样的效果还是受限于机能，iOS就更别说了，虽然safari开始支持service worker，但支持PWA还有待时日；Electron虽然能开发跨端应用，不过还有很多的局限，比如应用体积实在大，比如无法获取外部当前鼠标选中的文件等等。所以对于新技术应该理性看待，自己亲手实践一下，而不应盲目从众。

## 总结一下

**今年的技术栈成长：**

- 更加深入Vue的开发
- 开始学习Three.js
- 开始用上ESLint
- 开始学习TypeScript
- 开始使用前端测试（Jest）
- 开始学习Electron
- 开始练习算法
- 对前端工程化+自动化有更多的实践和体会
- 持续维护三个开源项目：[vue-koa-demo](https://github.com/Molunerfinn/vue-koa-demo)、[hexo-theme-melody](https://github.com/Molunerfinn/hexo-theme-melody)、[PicGo](https://github.com/Molunerfinn/PicGo)
- 学习持续集成

希望我的2018年能够继续有所收获！
