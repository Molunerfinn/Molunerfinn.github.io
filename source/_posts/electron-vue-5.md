---
title: Electron-vue开发实战4——通过CI发布以及更新的方式
tags: 
  - 前端
  - Vue
  - Electron
  - Electron-vue
categories:
  - Web
  - 开发
date: 2018-04-24 14:11:00
---
## 前言

前段时间，我用[electron-vue](https://github.com/SimulatedGREG/electron-vue)开发了一款跨平台（目前支持Mac和Windows）的免费开源的图床上传应用——[PicGo](https://github.com/Molunerfinn/PicGo)，在开发过程中踩了不少的坑，不仅来自应用的业务逻辑本身，也来自electron本身。在开发这个应用过程中，我学了不少的东西。因为我也是从0开始学习electron，所以很多经历应该也能给初学、想学electron开发的同学们一些启发和指示。故而写一份Electron的开发实战经历，用最贴近实际工程项目开发的角度来阐述。希望能帮助到大家。

预计将会从几篇[系列文章](https://molunerfinn.com/tags/Electron-vue/)或方面来展开：

1. [electron-vue入门](https://molunerfinn.com/electron-vue-1/)
2. [Main进程和Renderer进程的简单开发](https://molunerfinn.com/electron-vue-2/)
3. [引入基于Lodash的JSON database——lowdb](https://molunerfinn.com/electron-vue-3/)
4. [跨平台的一些兼容措施](https://molunerfinn.com/electron-vue-4/)
5. [通过CI发布以及更新的方式](https://molunerfinn.com/electron-vue-5/)
6. [开发插件系统——CLI部分](https://molunerfinn.com/electron-vue-6/)
7. [开发插件系统——GUI部分](https://molunerfinn.com/electron-vue-7/)
8. 想到再写...

## 说明

`PicGo`是采用`electron-vue`开发的，所以如果你会`vue`，那么跟着一起来学习将会比较快。如果你的技术栈是其他的诸如`react`、`angular`，那么纯按照本教程虽然在render端（可以理解为页面）的构建可能学习到的东西不多，不过在main端（electron的主进程）应该还是能学习到相应的知识的。

如果之前的文章没阅读的朋友可以先从[之前的文章](https://molunerfinn.com/tags/Electron-vue/)跟着看。

<!-- more -->

## LOGO的准备

经过前面几篇文章的实战，我相信大家已经对于构建一个基本的electron应用没有太多的问题了。本文主要阐述一下如何让我们的应用通过CI系统来自动帮我们构建应用，然后发布给用户使用。以及之后如果有更新，要如何通知用户更新。

当然，在此之前，我们还需要做一件事：给你应用加上好看的LOGO。LOGO的设计和制作不在本文的设计范围内。为了我们的应用能够跨平台地使用，不同平台上应用的LOGO尺寸和格式也不尽相同。三个平台所需的图片格式如下：

* Linux - png
* macOS - icns
* Windows - ico 

准备一张1024\*1024以下，256\*256以上（长宽一致）的png图片，(推荐512 * 512）然后我们可以用一些工具来实现从png到其他两种格式。搜索png转ico或者png转icns的话有很多在线转换的网站，可以去上面在线转换。在mac上我推荐用的是[image2icon](http://www.img2icnsapp.com/)这个工具。

然后我们将所得的三个图片文件，放到electron-vue项目根目录的`build/icons/`目录下。

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fqnr7sfuvjj20h806cglq.jpg)

## 不同平台的构建配置

本文我们主要采用electron-vue已经配置好的基于[electron-builder](https://github.com/electron-userland/electron-builder)的构建脚本来进行我们的应用构建。构建脚本会读取`package.json`里的`build`字段里的配置来进行构建。electron-vue默认的配置如下：

```json
"build": {
  "productName": "ElectronVue",
  "appId": "org.simulatedgreg.electron-vue",
  "dmg": {
    "contents": [
      {
        "x": 410,
        "y": 150,
        "type": "link",
        "path": "/Applications"
      },
      {
        "x": 130,
        "y": 150,
        "type": "file"
      }
    ]
  },
  "directories": {
    "output": "build"
  },
  "files": [
    "dist/electron",
    "node_modules/",
    "package.json"
  ],
  "mac": {
    "icon": "build/icons/icon.icns"
  },
  "win": {
    "icon": "build/icons/icon.ico"
  },
  "linux": {
    "icon": "build/icons"
  }
}
```

简单讲述一下build配置里的一些字段的含义。

首先`productName`是你的应用的名字。`appId`的作用是用于Windows平台区分应用的标识。（**注意**该配置必须配置，而且稍后会有使用该配置的地方。如果不配置不使用的话，构建出来的Windows平台的应用将无法发送eletron的桌面通知）`dmg`这个配置里描述了macOS平台里，打开`dmg`安装包后显示的界面里的信息。如下图：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fqnrlfwfcej20u00mct9y.jpg)

表示了有两个标识，一个是应用文件，坐标是`(130, 150)`， 一个是应用文件夹的快捷方式，坐标是`(410, 150)`。

`directories`的`output`字段是你应用打包完生成的文件放置的目录。

`files`指明了要打包的目录。

而`mac`，`win`，`linux`是针对三个平台的不同的配置了。可以看出默认的配置里对它们的配置都是指向了不同的icon图标（也就是上一节所说的LOGO）。

PicGo在实际开发中，针对一些情况对默认的`build`配置项做出了一些增改：

```json
"build": {
  "productName": "PicGo",
  "appId": "com.molunerfinn.picgo",
  "directories": {
    "output": "build"
  },
  "files": [
    "dist/electron/**/*"
  ],
  "dmg": {
    "contents": [
      {
        "x": 410,
        "y": 150,
        "type": "link",
        "path": "/Applications"
      },
      {
        "x": 130,
        "y": 150,
        "type": "file"
      }
    ]
  },
  "mac": {
    "icon": "build/icons/icon.icns",
    "extendInfo": {
      "LSUIElement": 1
    }
  },
  "win": {
    "icon": "build/icons/icon.ico",
    "target": "nsis"
  },
  "nsis": {
    "oneClick": false,
    "allowToChangeInstallationDirectory": true
  },
  "linux": {
    "icon": "build/icons"
  }
},
```

由于PicGo在macOS上主要是一个顶部栏应用，所以在底部docker栏我并不想拥有一个占位的图标，所以在`mac`字段里加入了：

```json
"extendInfo": {
  "LSUIElement": 1
}
```

这个属性。参考相关[issue](https://github.com/electron-userland/electron-builder/issues/1456)。

在Windows平台上，默认打包出来的安装包并没有办法选择安装的路径，只会默认装到C盘的用户目录。这个并不是我们想要的。我们想要的是让用户自己选择安装的路径。

所以需要修改`windows`的一些配置以及加上一个`nsis`的配置来实现：

```json
"win": {
  "icon": "build/icons/icon.ico",
  "target": "nsis"
},
"nsis": {
  "oneClick": false,
  "allowToChangeInstallationDirectory": true
}
```

由于目前我还没有打包过Linux平台的应用，所以Linux相关的配置暂时先不做修改。

### Windows平台打包的一个小坑

还记得前面说到的一个配置：`appId`么，这个配置需要我们在主进程`index.js`里也要使用。否则打包后的应用将失去Windows平台的应用通知功能。这个`appId`是可以任意取的，只要保证不和其他应用重复即可。对于PicGo而言，`appId`是`com.molunerfinn.picgo`。

打开你的`main/index.js`，在Windows平台的时候加上这个`appId`：

```js
// ...
import pkg from '../../package.json'

// ...
// ...

if (process.platform === 'win32') {
  app.setAppUserModelId(pkg.build.appId)
}
```

这样就解决了通知的那个问题。

## 通过CI系统自动构建与发布

### 版本相关

发布应用其实是一个比较繁琐的活，往往跟你的版本控制绑在一块，所以通常在项目开始的阶段就要有所布局。我说说我的做法吧，不一定很科学，不过简单易行。

1. 仓库主要两个分支，一个dev一个master。平时在dev上开发，只有在发布新版的时候merge到master上。
2. 书写简单的更新版本的脚本，一键打tag+push到GitHub。
3. 结合CI系统，自动构建master分支的代码，并将应用推送到GitHub仓库去。

其中简单的更新版本的脚本我是在`package.json`里写了简单的`scripts`：

```json
"scripts": {
  "patch": "npm version patch && git push origin master && git push origin --tags", // 小版本
  "minor": "npm version minor && git push origin master && git push origin --tags", // 次版本
  "major": "npm version major && git push origin master && git push origin --tags"  // 大版本
}
```

里面用到了npm的一个命令，`npm version [options]`，具体可以参考version的[文档](https://docs.npmjs.com/cli/version)。简单来说，它能够自动帮你升级版本，修改`package.json`里的version，并打上相应的git tag，很方便。

举个例子，一个符合语义的版本号通常由如下三个部分组成：`major.minor.patch`，比如`1.5.3`。如果我运行了`npm run patch`，那么将会将小版本更新：`1.5.4`，同时修改`package.json`里的`version`字段为`1.5.4`并自动打上一个git tag `1.5.4`，并将这个修改和tag推送到远端。

不过需要注意的是，一开始我是通过electron-vue自带的`npm run build`这个脚本让CI去执行构建，但是发现无法自动上传到GitHub的release里。所以通过查阅相关资料后，发现最简单的就是把对应的npm scripts命名为`release`。于是我把原本的`npm run build`的脚本复制了一遍，起了一个新名`release`:

```json
"scripts": {
  // ...
  "release": "node .electron-vue/build.js && electron-builder",
  // ...
}
```

### CI相关

说到这里都还没说到CI系统。什么是CI？可以参考阮一峰老师给出的解释[《持续集成是什么？》](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)。我们如果每次发布应用都需要我们在本地构建，然后手动上传到GitHub（或者其他地方）去，然后让别人能下载的话，未免太累了。而且通常我们开发electron应用就是为了能够跨平台，但是要构建不同的平台的应用意味着我们要在不同的平台分别构建。这也是不能忍受的。

于是网上有一些第三方的CI系统，它们能够帮我们，在某些分支（比如master）发生了某些更新（比如更新了tag）的时候帮我们执行某些脚本（比如构建、测试）。这样就省却了我们在本地、多平台构建的烦心事，而且让一些都变得「自动化」了起来。

有了CI之后，我的electron应用的发布就变成这样的流程了：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fqq9mnpumwj20rb04bt9a.jpg)

这样，我们只需要Push代码足矣。

针对Linux或者macOS的构建，我们可以使用[Travis-CI](https://travis-ci.org/)，针对Windows平台的构建，我们可以使用[AppVeyor](https://www.appveyor.com/)。一个好消息是，它们对于在GitHub上的开源项目都是可以免费构建的，并且和GitHub的仓库结合地特别好，配置也比较简单，可以说的非常良心了。

在使用它们之前，我们需要给予它们一定的权限让它们能够访问我们的GitHub仓库。所以需要：

1. 用你的GitHub账号注册它们，才能获取你的仓库列表。
2. 在GitHub上生成[token](https://github.com/settings/tokens)，赋予CI系统读写你的仓库的权限。生成token的具体操作可以查看之前我写的一篇针对hexo持久化构建的[文章](https://molunerfinn.com/hexo-travisci-https/)。
3. 针对不同的CI平台创建不同的配置文件，好让它们知道你要它们执行什么操作。不过electron-vue很友好地为我们准备了Travis-CI的配置文件模板`.travis.yml`和AppVeyor的配置文件模板`appveyor.yml`。所以我们基本上只需要在它们的基础上小修改即可。

### Travis-CI

注册并登录Travis-CI后，找到你要构建的仓库，然后打开，点击设置进入如下页面：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19gy1fi438l5g9sj218a0wmn1u.jpg)

配置一下环境变量，名为`GH_TOKEN`，token的值就是上一步我们在GitHub生成的token。等会会有用。

PicGo经过修改后的`.travis.yml`如下：

```yaml
# Commented sections below can be used to run tests on the CI server
# https://simulatedgreg.gitbooks.io/electron-vue/content/en/testing.html#on-the-subject-of-ci-testing
osx_image: xcode8.3
sudo: required
dist: trusty
language: c
matrix:
  include:
  - os: osx
  # - os: linux
    env: CC=clang CXX=clang++ npm_config_clang=1
    compiler: clang
cache:
  directories:
  - node_modules
  - "$HOME/.electron"
  - "$HOME/.cache"
addons:
  apt:
    packages:
    - libgnome-keyring-dev
    - icnsutils
before_install:
- mkdir -p /tmp/git-lfs && curl -L https://github.com/github/git-lfs/releases/download/v1.2.1/git-lfs-$([
  "$TRAVIS_OS_NAME" == "linux" ] && echo "linux" || echo "darwin")-amd64-1.2.1.tar.gz
  | tar -xz -C /tmp/git-lfs --strip-components 1 && /tmp/git-lfs/git-lfs pull
install:
- nvm install 8.9
- curl -o- -L https://yarnpkg.com/install.sh | bash
- source ~/.bashrc
- npm install -g xvfb-maybe
- yarn
script:
- npm run release
- yarn run build:docs
branches:
  only:
  - master
after_script:
  - cd docs/dist
  - git init
  - git config user.name "Molunerfinn"
  - git config user.email "marksz@teamsz.xyz"
  - git add .
  - git commit -m "Travis build docs"
  - git push  --force --quiet "https://${GH_TOKEN}@github.com/Molunerfinn/PicGo.git" master:gh-pages
```

抛去很多前置依赖（比如C++编译库之类的）和构建环境（是什么系统，是什么语言），那些都是electron-vue给我们预置好的。我们需要注意的仅仅是几个部分：

1. script
2. branches
3. after_script

`script`是当系统和环境和依赖都准备好之后，你要CI运行的命令。在这里我运行了两个命令，一个是`npm run release`，这个就是打包构建应用啦，并且执行了这个命令之后，`electron-builder`会自动将生成好的安装包推送到我们GitHub仓库的draft release里。另一个是构建PicGo[主页](https://molunerfinn.com/PicGo/)的命令`yarn run build:docs`。

`branches`声明了你要在哪些分支在GitHub接收到了代码更新之后就构建，这里我们自然选择的是master。

`after_script`是当你执行完script里的脚本之后要做的事。可以为空。对于我而言主要在这个部分将上一步构建好的PicGo主页推送到GitHub的`gh-pages`分支。当然如果你的应用有使用说明、文档之类的网站，也可以在这里进行构建和推送。

注意到，在`after_script`命令的最后一行，有个`${GH_TOKEN}`，这个就是我们之前在Travis-CI配置里配置的环境变量`GH_TOKEN`。用环境变量的好处是不会暴露你的TOKEN，只有构建系统知道。

### AppVeyor

有了之前的经验，AppVeyor就更简单了。注册登录后，我们在主页添加一个PROJECT，选中你要构建的仓库。然后找到SETTING设置：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fqqawmvu6sj21ji06e3z0.jpg)

然后在左侧的`Genral`一栏的内容区中，找到构建的分支为master，以及设置我们仅在`tag`更新的时候构建：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fqqazwrslxj21600emjse.jpg)

> 当然这个都是根据项目实际来的配置，我只是说PicGo的项目是这样配置的。

然后在左侧的`Environment`区，找到环境变量配置，我们依然写入`GH_TOKEN`:

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fqqb1ufhj7j21ow0gydhu.jpg)

**修改完配置都别忘了拉到底部去保存！**

这样就算配置完了网页端的。而现在我们来看看`appveyor.yml`这个配置文件：

```yaml
# Commented sections below can be used to run tests on the CI server
# https://simulatedgreg.gitbooks.io/electron-vue/content/en/testing.html#on-the-subject-of-ci-testing
version: 0.1.{build}

branches:
  only:
    - master

image: Visual Studio 2017
platform:
  - x64

cache:
  - node_modules
  - '%APPDATA%\npm-cache'
  - '%USERPROFILE%\.electron'
  - '%USERPROFILE%\AppData\Local\Yarn\cache'

init:
  - git config --global core.autocrlf input

install:
  - ps: Install-Product node 8 x64
  - git reset --hard HEAD
  - npm install
  - node --version

build_script:
  #- yarn test
  - npm run release

test: off
```

依然是只需要关注我们所关心的配置即可。一个是`branches`，一个是`build_script`。有了`Travis-CI`的`.travis.yml`的经验，我相信你也能很快理解它。

经过上述配置之后，你已经实现了一个简单的前端工程的自动化构建推送流程了。而今你只需要关注代码提交，应用的构建都将会由CI系统自动帮你完成。当然CI系统也不仅仅是拿来构建electron应用的，正如你所见的，你能想到的其他项目的构建、测试其实它都能帮你通过预定义好的脚本完成。

### 发布Release

当CI构建玩应用，会将其推送到你的GitHub的release页面成为一个`draf`（草稿），你可以编辑这个草稿，加上标题和更新说明，就可以点击`publish`发布你的新版本的应用啦。

## electron应用的更新

electron应用的自动更新其实社区有很好的解决方案[electron-updater](https://github.com/electron-userland/electron-builder/tree/master/packages/electron-updater)。而electron-vue也在主进程的`main/index.js`里预先帮我们写好了一段注释的代码：

```js
// import { autoUpdater } from 'electron-updater'

// autoUpdater.on('update-downloaded', () => {
//   autoUpdater.quitAndInstall()
// })

// app.on('ready', () => {
//   if (process.env.NODE_ENV === 'production') {
//     autoUpdater.checkForUpdates()
//   }
// }
```

只要引入autoUpdater就能自动帮我们检查更新和自动下载安装更新。不过，凡事都有不过。这个方式虽然很简单，但是它需要的条件比较严格，需要你拥有证书用于应用签名。而macOS平台下的证书需要你申请开发者，一年99$的费用让我望而却步。

于是我只能退而求其次，能不能通过查询GitHub的release版本号，来比对当前版本，是否需要更新，并提醒用户呢？经过尝试，发现可行。我的实现方法如下:

我首先写了一个`updateChecker`的助手：

```js
import { dialog, shell } from 'electron'
import db from '../../datastore'
import axios from 'axios'
import pkg from '../../../package.json'
const version = pkg.version
const release = 'https://api.github.com/repos/Molunerfinn/PicGo/releases/latest'
const downloadUrl = 'https://github.com/Molunerfinn/PicGo/releases/latest'

const checkVersion = async () => {
  let showTip = db.read().get('picBed.showUpdateTip').value()
  if (showTip === undefined) {
    db.read().set('picBed.showUpdateTip', true).write()
    showTip = true
  }
  // 自动更新的弹窗如果用户没有设置不再提醒，就可以去查询是否需要更新
  if (showTip) {
    const res = await axios.get(release)
    if (res.status === 200) {
      const latest = res.data.name // 获取版本号
      const result = compareVersion2Update(version, latest) // 比对版本号，如果本地版本低于远端则更新
      if (result) {
        dialog.showMessageBox({
          type: 'info',
          title: '发现新版本',
          buttons: ['Yes', 'No'],
          message: '发现新版本，更新了很多功能，是否去下载最新的版本？',
          checkboxLabel: '以后不再提醒',
          checkboxChecked: false
        }, (res, checkboxChecked) => {
          if (res === 0) { // if selected yes
            shell.openExternal(downloadUrl)
          }
          db.read().set('picBed.showUpdateTip', !checkboxChecked).write()
        })
      }
    } else {
      return false
    }
  } else {
    return false
  }
}

// if true -> update else return false
const compareVersion2Update = (current, latest) => {
  const currentVersion = current.split('.').map(item => parseInt(item))
  const latestVersion = latest.split('.').map(item => parseInt(item))
  let flag = false

  for (let i = 0; i < 3; i++) {
    if (currentVersion[i] < latestVersion[i]) {
      flag = true
    }
  }

  return flag
}

export default checkVersion
```

然后在`main/index.js`里，我在app准备启动的时候，调用这个更新助手：

```js
// ...
import uploader from './utils/uploader.js'

app.on('ready', () => {
  // ...
  updateChecker()
  // ...
})
```

这样就能在启动应用的时候弹出更新提示：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fqqbm19ptvj20nc08swf7)

## 总结

本文简要地讲述了electron应用用上CI系统帮我们自动化构建和推送，以及在没有申请开发者，没有证书用于应用的代码签名的情况下如何告知用户进行应用更新。要做一个健壮的应用就应该考虑到应用的版本发布、版本更新和对用户的更新通知。

本文很多都是我在开发`PicGo`的时候碰到的问题、踩的坑。也许文中简单的几句话背后就是我无数次的查阅和调试。希望这篇文章能够给你的`electron-vue`开发带来一些启发。文中相关的代码，你都可以在[PicGo](https://github.com/Molunerfinn/PicGo)的项目仓库里找到，欢迎star~如果本文能够给你带来帮助，那么将是我最开心的地方。如果喜欢，欢迎关注我的[博客](https://molunerfinn.com)以及[本系列文章](https://molunerfinn.com/tags/Electron-vue/)的后续进展。

> **注：文中的图片除未特地说明之外均属于我个人作品，需要转载请私信**
