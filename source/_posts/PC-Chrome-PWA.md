---
title: 用新版的Chrome把PWA网站添加到桌面，获得媲美原生应用的体验
tags: 
  - 前端
  - PWA
categories:
  - Web
  - 开发
date: 2018-03-03 16:12:00
---

**2018.03.06 更新：**非PWA网站也能通过独立窗口而非浏览器打开。具体看「注意事项」。

## PWA是什么

引用自[Harttle.Land](http://harttle.land/2017/01/28/pwa-explore.html)的说法：

> [PWA](https://developers.google.com/web/progressive-web-apps/)(Progressive Web Apps)是 Google 最近在提的一种 Web App 形态 （或者如 Wikipedia 所称的“软件开发方法”）。 Harttle 能找到的关于 PWA 最早的一篇文章是 2015年6月 Alex Russell 的一篇博客： [Progressive apps escaping tabs without losing our soul](https://infrequently.org/2015/06/progressive-apps-escaping-tabs-without-losing-our-soul/)， **让 Web App 从标签页跳出来，同时保持 Web 的灵魂。**

> 如 Alex 所述，PWA 意图让 Web 在保留其本质（开放平台、易于访问、可索引）的同时， 在离线、交互、通知等方面达到类似 App 的用户体验。按 [Google 官方的解释](https://developers.google.com/web/progressive-web-apps/) PWA 具有这些特性：Reliable, Fast, Engaging。

它比原生应用更轻量，但是却比现有的Web APP的功能更加丰富。最大也是最关键的区别是它能够脱离浏览器的「束缚」（虽然依然是基于浏览器的技术），能够把PWA网站添加到你的桌面上，不管是PC操作系统还是手机操作系统，类似于一个原生应用一样，并且拥有媲美原生应用的体验。

它也能拥有原生APP应用一般的启动闪屏，它也能像原生APP应用一般能有消息推送——不过要知道，它源自Web，通常只有传统APP的体积的十分之一甚至更小。它不用等待下载安装的时间，打开网页的时候就已经「下载」并且「安装」完毕。

要想体验这项技术，如果你是安卓用户，那最新版的Chrome已经支持；如果你是iOS用户，可以等待3月份的11.3版本更新；如果你是PC电脑用户，那么来看看怎么在电脑上也体验体验PWA吧。

<!-- more -->

## 配置Chrome

首先更新你的Chrome版本到64或以上。

然后在地址栏输入`chrome://flags`，找到`Desktop PWAs`的选项将其`Enabled`了，然后Chrome会提示你重启浏览器。

![](https://ws1.sinaimg.cn/large/8700af19ly1fozgv6nxloj20l7050q37)



重启浏览器后，PWA添加到桌面的特性就已经具备了。

## 将PWA网站添加到桌面

我这里使用的是我的[博客](https://molunerfinn.com)（基于自己写的[hexo-theme-melody](https://github.com/Molunerfinn/hexo-theme-melody)主题搭建的）。打开网站，然后在浏览器右侧找到设置的按钮。接下去我针对Windows平台和macOS平台做分开讲解。

### Windows平台

![](https://ws1.sinaimg.cn/large/8700af19ly1foznbd9gcdj20x10iw4mx)



Windows平台找到`添加到桌面`这个按钮，点击，然后会出现一个确认框：



![](https://ws1.sinaimg.cn/large/8700af19ly1foznebsdhvj20w50gwhba)



点击添加。然后你就可以在桌面上看到相应的图标：



![](https://ws1.sinaimg.cn/large/8700af19ly1fozng6ajhhj20bq09hwj9)



双击打开：



![](https://ws1.sinaimg.cn/large/8700af19ly1foznhijnsdj20q30gjx1u)



你会发现打开了一个没有浏览器痕迹的网页，或者说是个应用——这就是PWA了。PWA支持离线启动技术，即使在没网的情况下也能启动应用。不过在需要网络条件下才能发送的请求依然需要网络环境。

### macOS平台

相对于Windows平台比较简单的操作，macOS平台的操作相对有点绕弯，不过也大致相同。macOS的Chrome无法一次性就把PWA应用添加到桌面。需要先把PWA网站生成一个app应用，然后你再手动把这个app应用以快捷方式复制到桌面。

接下来是具体步骤：



![](https://ws1.sinaimg.cn/large/8700af19ly1fozny8dva4j20zk0m81kx)



打开一个PWA网站，此处依然以我的[博客](https://molunerfinn.com)作为例子，然后再右侧找到配置菜单，下拉选中`添加到“应用”文件夹`。然后等待几秒钟，会出现一个对话框：



![](https://ws1.sinaimg.cn/large/8700af19ly1fozo1ww7ubj20zj0cnx29)



此时这个PWA应用已经生成完毕了。我们点击添加。之后你就可以在你的应用列表看到它了：



![](https://ws1.sinaimg.cn/large/8700af19ly1fozo2gf2xij20zk0m8qcm)



不过如果你要在你的桌面上添加这个应用的话，还需要找到这个app的位置，一般是在`/Users/你的用户名/Applications/Chrome\ Apps.localized/`这个文件夹下。用finder打开：



![](https://ws1.sinaimg.cn/large/8700af19ly1fozo545tmij20h50470tp)



然后选中这个应用，按住`alt+command`键把它拖拽到桌面上，就会生成一个快捷方式啦。这个方法也同样适用于其他应用。

## 注意事项

如果是非PWA应用，也会有`添加到桌面`或者`添加到应用文件夹`的选项。不过当你双击打开它们的时候依然会调用Chrome浏览器去打开，跟以前的书签的作用无差别。

**不过**，依然可以通过一个小操作来实现。感谢@[RiiSan](https://weibo.com/5319395630)指出我原文的错误。

前置步骤跟之前说的一样，然后打开`chrome://apps`，找到你制作的应用，然后右键，选择`在窗口中打开`。那么就能获得跟PWA应用单独窗口的类似体验。不过它是不具备PWA离线打开的能力哦，只是纯粹的一个网页通过独立窗口打开而已。

![](https://ws1.sinaimg.cn/large/8700af19ly1fp32grp8nmj21ns0s8k4a)

目前`Desktop PWAs`还是实验性的功能，所以有可能出现不稳定的情况，依照自己的情况作出决定~

## 结尾

就目前来说，我能想到的比较理想的使用条件是，在一些功能性网站支持PWA的情况下，是不用再去下它们的桌面客户端了，直接通过PWA添加到桌面，就能像使用原生应用一样使用它们啦。比如推特，比如Medium等。

下面给出一个别人总结的PWA网站列表，可以去体验一波~

https://github.com/hemanth/awesome-pwa
