---
title: hexo持久化构建以及给自有域名github-page上HTTPS
tags: 
  - hexo
categories:
  - hexo
date: 2017-07-27 18:44:00
---

过一段时间博客的主机要到期了。我看了一下才发现，主机上其实我也只是放着我hexo的静态博客而已。觉得每个月要交几十块钱其实并不值当，遂决定把hexo博客从云主机上转移出来到github-page上。

本文主要记录如何运用`Travis-CI`对hexo博客进行持久化构建，以及通过`Cloudflare`给自有域名的github-page加上`https`。**文末给出如何将自己的博客加入`HSTS`网站列表。**

<!-- more -->

之前在自己的主机上写博客的时候，写完执行一下`hexo g`就可以了。不过后来想想其实还是不太安全。这个方式万一在主机dang掉之后，数据就有可能找不回来了。如果每次写完既要`hexo g`又要`hexo d`的话又过于麻烦，而且只能推送构建后的页面而不能保存文章源(markdown)文件。于是趁此机会来改造一下，也是一件快意之事。

## 持久化构建

持久化构建，也就是`持久化`和`构建`。

`持久化`的意思就是能保证数据能够长久保留下来。对应于我的情况来说，github就是一个非常优秀的持久化存储数据的地方。github的仓库能够持久化保留我的md文件+构建后的hexo博客页面。

而`构建`就是把源(md)文件编译成博客页的步骤。

能够做到构建的同时持久化的，目前来说最方便而且最省心的就是`Travis-CI`了。

简单来说，在你push到github仓库后，通过配置文件，它能够执行一系列的脚本（比如编译、构建、测试、推送等）。对于我来说，就可以实现：只要我写完hexo的md博客，推送到github仓库里后，它就能自动帮我构建并推送到github相应的分支上。从而实现了持久化构建，以及我博客的更新。

### 注册Travis-CI

去官网注册一下[Travis-CI](https://travis-ci.org/)，关联上你的github账号，给予权限后就可以把你的github仓库同步到它这里来。

![Travis-CI](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi42yvwwfzj21460le418.jpg)

在需要`Travis-CI`构建的仓库打开同步设置。

![打开同步设置](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi430jb981j20w404ct8w.jpg)

### 设定Token

去github的个人设置那边找到Token设置。

![Github-token](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi433d1zlkj21mi0q4799.jpg)

设定一个Token给`Travis-CI`，名字自己取让它对我们的仓库能够拥有读写权限（就够了）。

![Token](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi434caiywj216o08otaa.jpg)

**然后保存一下这个Token——它只显示一次，之后就不再显示了。所以找个地方把它记下来先。**

然后回到`Travis-CI`里，对于开启同步的仓库进行设置，我们把刚才的这个Token存储为一个叫做`GH_TOKEN`的环境变量，可以在`.travis.yml`这个配置文件里通过`${GH_TOKEN}`的方式获取。这样就不会将你的TOKEN暴露出去了。

![GH_TOKEN](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi438l5g9sj218a0wmn1u.jpg)

然后在所在github项目里创建一个叫做`.travis.yml`的文件。这个文件就是`Travis-CI`的执行脚本。它会根据里面定义的环境、步骤一步一步执行直到最后输出结果。

### 编写.travis.yml

`.travis.yml`是`Travis-CI`读取的配置文件。我的配置文件如下：

```yml
language: node_js # 声明环境为node
node_js: stable

# Travis-CI Caching
cache:
  directories:
    - node_modules # 缓存node_modules文件夹

# S: Build Lifecycle
install:
  - npm install # 下载依赖

script:
  - hexo algolia # 我装了algolia的搜索工具。这一步正常可以是 hexo g


after_script: # 推送到github的部分
  - cd ./public
  - git init
  - git config user.name "Molunerfinn"
  - git config user.email "marksz@teamsz.xyz"
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master # 通过之前存在Travis-CI里的token以及github仓库的地址推送到相应的master分支
# E: Build LifeCycle

branches:
  only:
    - hexo # 只对hexo分支构建

env: # 环境变量
 global:
   - GH_REF: github.com/Molunerfinn/Molunerfinn.github.io.git # 我的仓库地址
```

### 推送

我原本设定的是master分支作为`github-page`的展示分支。所以我切出了一个`hexo`分支，用来放置博客原文。`hexo`分支只需要留下如下的目录结构即可（其实就是除了public文件夹以外，其他文件夹都可以留）

```
.
├── _config.yml
├── .travis.yml
├── package-lock.json
├── package.json
├── scaffolds
│   ├── draft.md
│   ├── page.md
│   └── post.md
├── source
│   ├── CNAME
│   ├── _posts
│   ├── about
│   ├── favicon.ico
│   └── tags
└── themes
    └── next
```

做完上述工作后，将`.travis.yml`这个文件添加进`git`仓库，然后推送到远端的`hexo`分支，就会发现`Travis-CI`已经接收到相应的请求正在进行构建了。构建完成后，会将`public`文件夹内的内容推送到`master`分支。至此我们完成了持久化构建的部分。

## 给自有域名的github-page上HTTPS

从Chrome56左右开始，对于没有HTTPS的网站，不符合要求的，都不会出现一把小绿锁。反之，有了小绿锁的网站，标志着这个网站是HTTPS安全的。

假如你没有自己的域名，而是使用着github的子域名（形如`xxx.github.io`）那么能够自动拥有github的https，无需操心。

但是如果有自己的域名，想要实现自己的域名通过CNAME指向github的page，并加上小绿锁的话，就比较麻烦了。首先我们需要将自己的域名通过CNAME指向github-page。在hexo的`source`文件夹里创建一个叫做CNAME的文件，内容只需要写上你自己的域名即可。对于我来说就是`molunerfinn.com`。

通过CNAME指向github-page的页面之后，我们发现，原本github自带的https已经不能再使用了。我们必须给自己的域名想办法弄上https。一开始并无头绪，不过好在我找到了`Cloudflare`这个解决方案。

### 注册Cloudflare

第一步当然是注册。`Cloudflare`是国外非常有名的一家网络服务提供商。它提供的其中一项免费服务就是给我们自有域名加上HTTPS。正好符合我们的需求。

注册成功后添加域名。

然后需要增加几个记录，其中A记录就是指向这`192.30.252.153`和`192.30.252.154`这两个IP地址，它们是github-page的ip地址。然后建一个CNAME将`www`的网址指向我们非`www`的网址

![DNS Records](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi43icnw1jj21l80wajwm.jpg)

然后需要将我们的域名的DNS服务商的地址改成`Cloudflare`要求的两个`DNS`服务器地址。每个人分配的不一样，而且必须用分配的否则会失效。

![DNS Server](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi43n3bs7yj21ig0g4wgj.jpg)

这个操作需要在自己的域名服务提供那边修改。一般是48小时内生效。

### 开启HTTPS

找到`Crypto`选项，这里我们需要开启`Flexible`的HTTPS选项。

![https](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi45c5avd6j21jq0s4tcc.jpg)

其实`Cloudflare`做的事就是，当访问我们的域名的时候，实际上走的是`Cloudflare`的服务器，这个时候这个阶段的访问是有HTTPS的。然后`Cloudflare`再去请求我们实际的内容，再将内容返回给用户。这一段是没有HTTPS的。也就是实际上是半HTTPS。不过对于我们静态博客来说，这种半HTTPS实际上已经够我们使用了。

可以看见开启HTTPS真的非常简单，基本不需要额外操作。

### 重定向

这个时候我们访问`https://molunerfinn.com`自然走的是HTTPS。但是如果有人访问了`http://molunerfinn.com`，那要如何跳转到HTTPS的页面呢？`CloudFare`另一个很棒的功能`Page Rules`就派上用场了。我们可以指定我们的域名强制使用HTTPS，并且当访问是HTTP的时候重定向到HTTPS。这样就能保证用户访问我们的页面都是通过HTTPS的了。

![Rewrite](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi45ezb4b2j21k20xotdz.jpg)

### 附录

#### DNS解析

刚开始用`CloudFare`的DNS服务器，国内域名解析一开始会时断时续。我自己大概是过了24小时之后开始稳定的。所以一开始有可能访问不到自己的博客这是正常的。一开始我还以为是Cloudflare那边问题比较严重还提了一个issue。后来第三天就正常了。

#### 加入HSTS的列表

上面说到，我们有可能访问自己的网站是走HTTP->304重定向->HTTPS。这个是浏览器跟服务器进行了一次通信之后才发生的跳转。那有没有可能做到，访问的是HTTP，但是浏览器识别之后自动转成HTTPS访问，而不经过重定向那一层操作呢？有的。通过HSTS的Preload List。

可以参考这篇文章对[HSTS](http://www.jianshu.com/p/caa80c7ad45c)进行更深入的了解。简单来说，HSTS能够使我们的网站安全性更上一层楼。

还是`CloudFare`，它家自有的HSTS功能，开启之后就能很好的满足我们的需要。（真是完美了）还是在`Crypto`选项下，开启`HSTS`

![HSTS](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi45gia6pcj21kc0iugny.jpg)

建议都使用默认的选项。

然后可以去[HSTS Preload List](https://hstspreload.org/)的网站把我们的域名进行检查并收录（不能是子域名，必须是一级域名），如果没通过会给出修改建议，按照建议修改就行。如果通过了，就会放入审核列表。之后可以时不时回来看看自己的网站被收录了没有。我是等了快一周才被收录。网上的说法普遍是几周内。所以耐心等待收录。一旦被收录就会应用到主流浏览器上，这样你的网站就是更加安全的网站啦。

![HSTS Preload List](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi45hvxt3oj21fu0f8gn5.jpg)

## 记录总结

至此，我的博客迁移工作就做完了。用的因为是`Cloudflare`的cdn加速，所以在国外访问速度很快，在国内访问的速度会稍慢一些。不过也无伤大雅。最关键的是通过上述的办法，让我的博客能够实现持久化构建，加上了HTTPS的小绿锁，并且成功加入HSTS的Preload List，还是比较满意。

![HTTPS](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi45ityzzrj20yo01smxb.jpg)

最后由衷感谢GitHub+Travis-CI+Cloudflare提供的这么优质的服务。