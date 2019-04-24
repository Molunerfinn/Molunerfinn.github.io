---
title: Electron-vue开发实战7——命令行调用与系统级别右键菜单的实现
tags: 
  - 前端
  - Vue
  - Electron
  - Electron-vue
categories:
  - Web
  - 开发
date: 2019-04-16 15:50:00
---
## 前言

前段时间，我用[electron-vue](https://github.com/SimulatedGREG/electron-vue)开发了一款跨平台（目前支持主流三大桌面操作系统）的免费开源的图床上传应用——[PicGo](https://github.com/Molunerfinn/PicGo)，在开发过程中踩了不少的坑，不仅来自应用的业务逻辑本身，也来自electron本身。在开发这个应用过程中，我学了不少的东西。因为我也是从0开始学习electron，所以很多经历应该也能给初学、想学electron开发的同学们一些启发和指示。故而写一份Electron的开发实战经历，用最贴近实际工程项目开发的角度来阐述。希望能帮助到大家。

预计将会从几篇[系列文章](https://molunerfinn.com/tags/Electron-vue/)或方面来展开：

1. [electron-vue入门](https://molunerfinn.com/electron-vue-1/)
2. [Main进程和Renderer进程的简单开发](https://molunerfinn.com/electron-vue-2/)
3. [引入基于Lodash的JSON database——lowdb](https://molunerfinn.com/electron-vue-3/)
4. [跨平台的一些兼容措施](https://molunerfinn.com/electron-vue-4/)
5. [通过CI发布以及更新的方式](https://molunerfinn.com/electron-vue-5/)
6. [开发插件系统——CLI部分](https://molunerfinn.com/electron-vue-6/)
7. [开发插件系统——GUI部分](https://molunerfinn.com/electron-vue-7/)
8. [命令行调用与系统级别右键菜单的实现](https://molunerfinn.com/electron-vue-8/)
9. 想到再写...

## 说明

`PicGo`是采用`electron-vue`开发的，所以如果你会`vue`，那么跟着一起来学习将会比较快。如果你的技术栈是其他的诸如`react`、`angular`，那么纯按照本教程虽然在render端（可以理解为页面）的构建可能学习到的东西不多，不过在main端(`Electron`的主进程）应该还是能学习到相应的知识的。

如果之前的文章没阅读的朋友可以先从[之前的文章](https://molunerfinn.com/tags/Electron-vue/)跟着看。本文主要是基于PicGo v2.1.0版本更新的重要内容做的讲述。

<!-- more -->

## 命令行调用

我们在使用一些`Electron`开发的应用程序的时候，可以发现有些程序是可以通过命令行唤起的。比如`VSCode`，在macOS的`.bash_profile`里可以设置`alias code='/Applications/Visual\ Studio\ Code.app/Contents/Resources/app/bin/code'`，这样就可以在命令行里通过`code xxx.js`来调用VSCode打开文件了。如果想打开当前目录，可以通过`code .`，如果想打开某个目录`code xxx`等等。

命令行调用里其实还涉及到一个问题，有的时候我们的应用是个「单例应用」，也就是不能「多开」。如何在只能单开的应用里，也实现命令行调用呢？比如`PicGo`，在软件打开的时候，命令行调用它也能上传图片，而不是打开一个新的`PicGo`窗口。没事，下面会详细说明。

### 实现命令行调用

首先我们要来实现命令行调用。其实`Electron`的命令行调用没有什么特殊的地方，与在`Node.js`端很类似。我以`PicGo`举例：

当我们在Windows下安装好了`PicGo`之后，可以在安装目录里找到`PicGo.exe`。你有没有想过在命令行里运行这个`exe`会怎么样呢？在安装目录里打开`powershell`，输入`.\PicGo.exe`，你会发现`PicGo`已经被打开了。如果我是加了一些参数打开会怎么样呢`.\PicGo.exe upload`

我们可以在`main`进程里的`ready`事件里把命令行参数打印出来：

```js
app.on('ready', () => {
  console.log(process.argv) // ['D:\\PicGo.exe', 'upload']
})
```

关键出现了，我们可以通过`process.argv`这个在`Node.js`端获取命令行参数的关键变量同样获得`Electron`被命令行打开后的命令行参数。那么我们就可以在`main`进程的`ready`阶段通过获取的`process.argv`参数来实现我们对应的功能。

对于PicGo而言，如果通过命令行打开它，并且传递了`upload xxx.jpg`的话，我们就可以认为用户需要调用PicGo来实现上传一张图片。那么我们可以这么做（以下是实例代码）：

```js
import path from 'path'
import fs from 'fs-extra'
const getUploadFiles = (argv = process.argv, cwd = process.cwd()) => {
   files = argv.slice(2) // 过滤['D:\\PicGo.exe', 'upload']这两个参数，直接获取需要上传的图片路径
   let result = []
   if (files.length > 0) { // 如果图片列表不为空
     result = files.map(item => {
       if (path.isAbsolute(item)) { // 如果是绝对路径
         return {
           path: item
         }
       } else {
         let tempPath = path.join(cwd, item) // 如果是相对路径，就拼接
         if (fs.existsSync(tempPath)) { // 判断文件是否存在
           return {
             path: tempPath
           }
         } else {
           return null
         }
       }
     }).filter(item => item !== null) // 排除为null的路径
   }
   return result // 返回结果
}
```

拿到图片列表后就执行自带的上传逻辑即可。下面说说单开应用的命令行调用注意事项。

### 实现单例应用的命令行调用

`Electron`的发展很快，本文讲述的`Electron`版本为当前最新的`v4.1.4`，所以关于实现单例应用的`api`也是跟随[官方文档](https://electronjs.org/docs/api/app)走的，如果你的Electron版本不是`v4.x`，那么需要找对应版本的`Electron`文档。

当前版本下实现单例应用的官方例子是：

```js
const { app } = require('electron')
let myWindow = null

const gotTheLock = app.requestSingleInstanceLock() // 拿到单例锁

if (!gotTheLock) { // 如果一个应用二次打开，那么getTheLock为false
  app.quit() // 立即退出二次打开的应用
} else {
  app.on('second-instance', (event, commandLine, workingDirectory) => { // 一个应用尝试打开第二个实例时触发
    // Someone tried to run a second instance, we should focus our window.
    if (myWindow) {
      if (myWindow.isMinimized()) myWindow.restore()
      myWindow.focus()
    }
  })

  // Create myWindow, load the rest of the app, etc...
  app.on('ready', () => {
  })
}
```

注意有个`second-instance`事件。当我们试图在打开一个单例应用之后再打开这个应用的时候，就会触发这个事件。并且这个事件的回调函数里，有`commandLine`和`workingDeirectory`，实际上它们就是`process.argv`和对应的`cwd`（执行路径）。因此我们可以在这个事件里书写当应用试图被二次打开的时候应该做的事的逻辑。以下依然以PicGo举例：

```js
app.on('second-instance', (event, commandLine, workingDirectory) => {
 let files = getUploadFiles(commandLine, workingDirectory)
 if (files === null || files.length > 0) { // 如果有文件列表作为参数，说明是命令行启动
   if (files === null) { // 如果为null说明是让PicGo上传剪贴板的图片
     uploadClipboardFiles()
   } else { // 否则说明是让PicGo上传具体的图片文件
     // ...
     uploadChoosedFiles(win.webContents, files)
   }
 } else { // 如果files === [] 说明并不是命令行启动或者并没有带额外参数
   if (settingWindow) { // 说明用户是点击了PicGo图标启动，那么这个时候把原有的窗口调出来并focus即可
     if (settingWindow.isMinimized()) {
       settingWindow.restore()
     }
     settingWindow.focus()
   }
 }
})
```

这里我们通过读取`commandLine`参数，来判断用户是用命令行来调用`PicGo`上传图片的，还是仅仅是通过`PicGo`的图标再次打开`PicGo`的。关键的逻辑就是判断`commandLine`里有没有关键的参数，从而得出是否是从命令行调用我们的应用的。如果用户仅仅是通过`PicGo`图标再次打开`PicGo`，那么我们应该把之前打开过的窗口复原并激活，告诉用户你之前已经打开过这个应用了。当然具体的业务逻辑不能一概而论，这里只是我对`PicGo`的一点理解，只需知道核心是监听`second-instance`事件即可。

以下是上述实现的截图，注意命令行输出都只在第一个终端进程里，说明我们实现了单例应用的命令行调用：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/commandline-picgo.gif)

#### macOS的命令行调用

其实这个章节到上面基本结束。不过我想起我演示的是在Windows下做的，相对简单。而macOS下的命令行调用`Electron`应用会有个坑，所以还是要说一下为好。（由于我没有Linux机器，所以Linux部分就不说明了，有兴趣的朋友可以测试一下跟我反馈！）

大家都知道`macOS`的应用基本是放在`Application`下的，所以我们会很自然想到直接命令行调用它们：

```bash
open /Applications/PicGo.app
```

但是这样做并不能传递参数进去，因为执行命令的是`open`。

所以我们需要到更深层次的路径启动`PicGo`并传递参数进去：

```bash
/Applications/PicGo.app/Contents/MacOS/PicGo upload xxx.jpg
```

只有这样才能像Windows那样类似`PicGo.exe`来实现调用。

值得注意的是，`Electron`的macOS应用想要在生产阶段打开`debug`模式查看`console`的输出也是到上述应用的对应目录下：

```bash
/Applications/PicGo.app/Contents/MacOS/PicGo --debug
```

而`Widnows`相对简单，只需要：

```bash
.\PicGo.exe --debug
```

（Linux请自测）

## 系统级别右键菜单

在实现了命令行调用的功能之后，我就在考虑给PicGo加上原生的系统右键菜单。这样做的好处是用户可以直接在一张图片上右键->通过PicGo上传。例如：

Windows下：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/windows-context-menu.png)

macOS下：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/macos-context-menu.png)

接下来说说二者在实现上不同的地方。（Linux没有测试，欢迎有兴趣的小伙伴测试一下跟我说说~）

### Windows

Windows的右键菜单的原理其实很简单，在注册表里写入值就行。篇幅原因不会对Windows注册表的知识做过多的展开。我们只关注往哪里写值，写哪些值才能实现我们要的效果。

首先我们可以看看VScode是如何实现右键菜单「Open with Code」的。

![VScode的右键菜单](https://blog-1251750343.cos.ap-beijing.myqcloud.com/vscode-context-menu.png)

在系统里按快捷键`WIN+R`然后输入`regedit`打开注册表编辑器，我们来找到`VSCode`的右键菜单所在地：

`HKEY_CLASSES_ROOT` → `*` → `shell` → `VSCode`:

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/vscode-reg.png)

可以看到一个「默认」的属性下的数据为「Open w&ith Code」，这个就是我们看到的菜单名。而一个叫「Icon」的属性下的数据为`VSCode`的`exe`安装路径。所以可以认为这个`Icon`可以获取`exe`的`Icon`并显示到菜单上。

不过这里还没有看到如何将文件路径作为参数传入`VScode`的。继续看：

`HKEY_CLASSES_ROOT` → `*` → `shell` → `VSCode` → `command`:

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/vscode-reg-2.png)

在`command`目录下我们看到了如下数据：

`"C:\Users\PiEgg\AppData\Local\Programs\Microsoft VS Code\Code.exe" "%1"`

可以看出这个`%1`就是作为参数传给`Code.exe`的。有了`VSCode`作为参考，给自己的`Electron`应用实现一个系统级别的右键菜单也不难了。有人可能会说我可以在应用启动阶段通过某些`npm`包（比如[windows-registry](https://www.npmjs.com/package/windows-registry)）来实现对注册表的写入。

不过实际上，在`Windows`平台，如果你是用`electron-builder`打包的话有一个更简洁的解决方案，那就是编写`NSIS`脚本来实现，对此`electron-builder`官方给出的[文档](https://www.electron.build/configuration/nsis#custom-nsis-script)可以一看。

本文不对`NSIS`脚本做过多的描述，你只需要知道它是用来生成`Windows`安装界面的一门脚本语言，你可以通过它来控制安装（卸载）界面都有哪些元素。并且它可以接入安装的生命周期，做一些操作，比如写入注册表。我们利用这个特性，来给PicGo做一个安装阶段写入注册表的操作，实现系统级别的右键菜单。

`electron-builder`给`NSIS`暴露的钩子主要有`customHeader`, `preInit`, `customInit`, `customInstall`, `customUnInstall`，等等。

我们可以在`customInstall`阶段通过获取用户安装PicGo的路径`$INSTDIR`来实现对注册表关键值的写入。自己书写的`installer.nsh`默认放在项目的`build`目录下，那么`electron-builder`在构建`Windows`应用的时候将会自动读取这个文件以及`package.json`里的配置来生成安装界面。

写入注册表的格式大概是这样：
```
WriteRegStr <reg-path> <your-reg-path> <attr-name> <value>
```

以下是PicGo的`installer.nsh`，仅供参考：

```nsh
!macro customInstall
   WriteRegStr HKCR "*\shell\PicGo" "" "Upload pictures w&ith PicGo"
   WriteRegStr HKCR "*\shell\PicGo" "Icon" "$INSTDIR\PicGo.exe"
   WriteRegStr HKCR "*\shell\PicGo\command" "" '"$INSTDIR\PicGo.exe" "upload" "%1"'
!macroend
!macro customUninstall
   DeleteRegKey HKCR "*\shell\PicGo"
!macroend
```

注意`HKCR`即是注册表目录`HKEY_CLASSES_ROOT`的缩写。在写`value`的时候如果要写多个参数，可以用单引号包起来。`attr-name`不写即为默认。相信有了`VSCode`的右键菜单注册表说明，你也能看得懂上面的PicGo的脚本了。同时注意我们应该在卸载阶段将之前写的注册表删除，以免用户卸载了应用之后菜单还在，上述脚本的后面部分是是在做这个事情。

因为上一章实现了命令行调用，所以我们的菜单就可以通过`'"$INSTDIR\PicGo.exe" "upload" "%1"'`来实现菜单调用命令了。

### macOS

macOS的话可以通过实现自动化脚本来生成右键菜单。打开`automator`：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/mac-automator.png)

然后新建一个`快速操作`:

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/automator-quick.png)

将快速操作的工作流程限制到`图像文件`，并且只作用于`访达.app`里，同时在左侧菜单里找到`shell`组件，将其拖拽到右侧编辑区：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/automator-quick-1.png)

将`shell`选择成`/bin/bash`，传递输入选成`作为自变量`。

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/automator.png)

然后将默认的内容改成如下(实际上就差不多是之前说的`macOS`下如何命令行调用`Electron`应用的写法)：

```bash
/Applications/PicGo.app/Contents/MacOS/PicGo upload "$@" > /dev/null 2>&1 &
```

其中macOS的快捷操作里，是通过`"$@"`来作为参数传递的。

如何作为右键菜单？只要把你生成的这个workflow文件（夹），放到`~/Library/Services`这个目录下就行了。

这样你就在你右键菜单里看到它：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/macos-context-menu.png)

如果你的服务项过多的话，会在服务的二级菜单里看到它：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/macOS-context-menu-2.png)

其中，菜单名就是你生成的这个workflow的文件（夹）名。

那么生成了这个workflow之后，我们如何实现不让用户手动创建，而是自动帮他们放到`~/Library/Services`目录下呢？macOS没有Windows那么方便的安装工具脚本语言，那么我们可以在`main`进程里手动来实现这个功能。下面是PicGo的[beforeOpen.js](https://github.com/Molunerfinn/PicGo/blob/dev/src/main/utils/beforeOpen.js)，其中我们将我们生成的`workflow`文件（夹）放到项目的`static`目录下。

```js
import fs from 'fs-extra'
import path from 'path'
import os from 'os'
if (process.env.NODE_ENV !== 'development') {
  global.__static = path.join(__dirname, '/static').replace(/\\/g, '\\\\')
}
if (process.env.DEBUG_ENV === 'debug') {
  global.__static = path.join(__dirname, '../../../static').replace(/\\/g, '\\\\')
}
function beforeOpen () {
  const dest = `${os.homedir}/Library/Services/Upload pictures with PicGo.workflow`
  if (fs.existsSync(dest)) { // 判断是否存在
    return true
  } else { // 如果不存在就复制过去
    try {
      fs.copySync(path.join(__static, 'Upload pictures with PicGo.workflow'), dest)
    } catch (e) {
      console.log(e)
    }
  }
}

export default beforeOpen
```

然后在主进程里加入这个方法，并判断是否在macOS下运行：

```js
// main/index.js
import beforeOpen from './utils/beforeOpen'
// ...
if (process.platform === 'darwin') {
  beforeOpen()
}
// ...
```

这样用户在安装PicGo之后，打开软件之后，他的右键菜单就多了一个「Upload pictures with PicGo」项了。

## 小结

至此，一个`Electron`应用的命令行调用以及系统级别右键菜单的实现就讲述完了。当然可能还有其他实现的方式，以及更细致的实现（比如还能支持文件夹右键等等）。我在这里也只是一个抛砖引玉，其他的实现或者更好的实现方式需要自己摸索啦。当然本文没有Linux的相关内容，主要是我时间有限并且没有Linux机器，所以也希望有兴趣的朋友自己在Linux下实现了本文的功能后也能跟我说说~

本文很多都是我在开发`PicGo`的时候碰到的问题、踩的坑。也许文中简单的几句话背后就是我无数次的查阅和调试。希望这篇文章能够给你的`electron-vue`开发带来一些启发。文中相关的代码，你都可以在[PicGo](https://github.com/Molunerfinn/PicGo)和[PicGo-Core](https://github.com/PicGo/PicGo-Core)的项目仓库里找到，欢迎star~如果本文能够给你带来帮助，那么将是我最开心的地方。如果喜欢，欢迎关注我的[博客](https://molunerfinn.com)以及[本系列文章](https://molunerfinn.com/tags/Electron-vue/)的后续进展。（PS: 下一篇文章应该会讲述一下如何构建一个Electron应用 **可扩展的快捷键系统** 。）

> **注：文中的图片除未特地说明之外均属于我个人作品，需要转载请私信**

## 参考文献

感谢这些高质量的文章、问题等：

1. [一个还不错的图床工具-PicUploader](https://www.xiebruce.top/17.html)
2. [Passing command line arguments to electron executable (after installing an already packaged app)](https://stackoverflow.com/questions/49552703/passing-command-line-arguments-to-electron-executable-after-installing-an-alrea)
3. [Command Line Arguments in Dev Mode](https://github.com/SimulatedGREG/electron-vue/issues/581)
4. [Electron app Docs](https://electronjs.org/docs/api/app#apprequestsingleinstancelock)
5.  以及没来得及记录的那些好文章，感谢你们！
