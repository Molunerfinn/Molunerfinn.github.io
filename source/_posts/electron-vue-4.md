title: Electron-vue开发实战3——跨平台的一些兼容措施
tags: 
  - 前端
  - Vue
  - Electron
  - Electron-vue
categories:
  - Web
  - 开发
date: 2018-03-20 14:40:00
---
## 前言

前段时间，我用[electron-vue](https://github.com/SimulatedGREG/electron-vue)开发了一款跨平台（目前支持Mac和Windows）的免费开源的图床上传应用——[PicGo](https://github.com/Molunerfinn/PicGo)，在开发过程中踩了不少的坑，不仅来自应用的业务逻辑本身，也来自electron本身。在开发这个应用过程中，我学了不少的东西。因为我也是从0开始学习electron，所以很多经历应该也能给初学、想学electron开发的同学们一些启发和指示。故而写一份Electron的开发实战经历，用最贴近实际工程项目开发的角度来阐述。希望能帮助到大家。

预计将会从几篇系列文章或方面来展开：

1. [electron-vue入门](https://molunerfinn.com/electron-vue-1/)
2. [Main进程和Renderer进程的简单开发](https://molunerfinn.com/electron-vue-2/)
3. [引入基于Lodash的JSON database——lowdb](https://molunerfinn.com/electron-vue-3/)
4. [跨平台的一些兼容措施](https://molunerfinn.com/electron-vue-4/)
5. 通过CI发布以及更新的方式
6. ...（想到再写）

## 说明

`PicGo`是采用`electron-vue`开发的，所以如果你会`vue`，那么跟着一起来学习将会比较快。如果你的技术栈是其他的诸如`react`、`angular`，那么纯按照本教程虽然在render端（可以理解为页面）的构建可能学习到的东西不多，不过在main端（electron的主进程）应该还是能学习到相应的知识的。

如果之前的文章没阅读的朋友可以先从之前的文章跟着看。

<!-- more -->

## 跨平台的重要性

虽然electron在大多数情况下的跨平台措施已经帮我们做得很好了。不过需要注意的是，不同平台必然存在细节上的差异。我们在书写跨平台应用的时候，如果只在自己书写平台下测试通过的话是不足以说明我们的应用是健壮的。（当然如果你只想提供给某个平台那另当别论）所以针对不同的发布平台，就需要做一些兼容性措施。

就我自己的感受而言，macOS平台支持的特性相对比较多，而这里面又很多是独有的，所以很多能在macOS上实现的功能却不一定能在windows上实现。所以对于windows用户而言，在保证整体应用的可用性的情况下，就有可能要相应地做一些妥协和牺牲。不过在windows上的一些操作习惯也可以反过来服务于macOS平台。这点我会在下面给出一个例子详细说明。

## 留意不同平台的独有功能

在开发electron应用的时候，很多时候我们只注意去查找api名，却容易忽视这个api能够使用的平台。在官方文档里，对于一些独占的api，大多都会有标识标出：

![](https://ws1.sinaimg.cn/large/8700af19ly1fpifmc1muoj21mo0ti0wa.jpg)

不过需要注意的是一些未有平台标识的api里的配置项，也有可能是某个平台的独占：

![](https://ws1.sinaimg.cn/large/8700af19ly1fpifrhg1r8j21k804w0tt.jpg)

平时开发的过程中，用到文档的地方还是需要细细留心，避免后续不必要的麻烦。

## 跨平台措施入门

上面讲了这么多，该到实例的时候了。在electron应用中，通常来说`renderer`进程的东西不需要做太多的跨平台措施——毕竟不管是哪个平台，都是跑在Chrome里的页面。所以大多数情况下，这个方面的工作会放在`main`进程里。不过也有例外：

### title-bar的操作区处理

下面是PicGo的windows版：

![](https://ws1.sinaimg.cn/large/8700af19ly1fpig60gzw6j20m80ciwf1.jpg)

下面是PicGo的macOS版：

![](https://ws1.sinaimg.cn/large/8700af19ly1fpig71431kj218g0p0tav.jpg)

可以发现除了颜色有些区别之外，顶部的`title-bar`操作栏也有些区别。macOS的程序窗口习惯将窗口的缩放、关闭按钮放在窗口的左上角。而windows程序则相反，它们喜欢放在窗口的右上角。所以为了迎合用户的操作习惯，我们在开发electron程序的时候也应该注意到这一点。

当然，如果是通过普通的`BrowserWindow`创建的窗口，那么将会自动拥有常见的macOS、windows的顶部栏，以及默认的样式。

<!--双平台的图片-->

我在这里想说的是如果想要更加美观的界面，通常我们喜欢「沉浸式」的顶部栏。对于macOS而言，沉浸式的顶部栏就是将顶部栏的三个操作按钮直接「嵌入」窗口主题的左上角。而对于windows而言，只能删去顶部的三个操作按钮，自己用前端的方式来实现了。所以这个地方两个平台的差异性就出来了。

在`main`进程里创建该窗口的时候，主要代码如下：

```js
const createSettingWindow = () => {
  const options = {
    height: 450,
    width: 800,
    show: false, // 当window创建的时候不用打开
    center: true,
    fullscreenable: false,
    resizable: false,
    title: 'PicGo',
    vibrancy: 'ultra-dark', // 窗口模糊的样式
    transparent: true,
    titleBarStyle: 'hidden', // title-bar的样式——隐藏顶部栏的横条，把操作按钮嵌入窗口
    webPreferences: {
      backgroundThrottling: false
    }
  }
  if (process.platform === 'win32') { // 如果平台是win32，也即windows
    options.show = true // 当window创建的时候打开
    options.frame = false // 创建一个frameless窗口，详情：https://electronjs.org/docs/api/frameless-window
    options.backgroundColor = '#3f3c37'
  }
  settingWindow = new BrowserWindow(options)

  settingWindow.loadURL(settingWinURL)

  settingWindow.on('closed', () => {
    settingWindow = null
  })
}
```

主要的工具是通过`process.platform`来判断不同的平台。当前可能的值有：

- 'aix'
- 'darwin'
- 'freebsd'
- 'linux'
- 'openbsd'
- 'sunos'
- 'win32'

在这里我们基本上只需要关心`darwin`（macOS）、`win32`（windows）、`linux`（Linux）这三个平台即可。注意，由于electron的对于`renderer`进程的加持，在`renderer`进程里也能直接使用`process.platform`来判断当前的操作系统。这是一个很方便的特性。

针对windows平台，由于采用了[frameless-window](https://electronjs.org/docs/api/frameless-window)，所以我们需要手动「绘制」顶部的缩放和关闭按钮，并配上相应的事件来模拟真实的按钮。

```html
<div class="fake-title-bar">
  PicGo - {{ version }}
  <div class="handle-bar" v-if="process.platform === 'win32'"><!-- 如果是windows平台 -->
    <i class="el-icon-minus" @click="minimizeWindow"></i>
    <i class="el-icon-close" @click="closeWindow"></i>
  </div>
</div>
```

相应的事件如下：

```js
minimizeWindow () {
  const window = BrowserWindow.getFocusedWindow()
  window.minimize()
},
closeWindow () {
  const window = BrowserWindow.getFocusedWindow()
  window.close()
},
```

简单来说就是调用了`BrowserWindow`的方法来获取当前激活的窗口，然后再对这个窗口进行缩小或关闭的操作。其实也不难对吧！

### 任务栏图标交互

针对不同的平台，我对PicGo的任务栏图标交互也有所区别。对于macOS而言，点击顶部菜单栏的时候会弹出一个小窗口：

![](https://ws1.sinaimg.cn/large/8700af19ly1fma907llb5j20m30ed46a)

由于macOS的顶部栏图标可以接受拖拽事件，所以就针对macOS的顶部栏制作了顶部栏图标对应的小窗口。让大部分操作不经过主窗口也能实现。而对于windows而言，没有顶部栏，取而代之的是位于底部栏的右侧的任务栏，通常点击任务栏里的图标就会把应用的主窗口调出来。所以为了迎合不同平台的操作习惯，我对于这个地方也做了相应的兼容性适配：

```js
tray.on('click', () => { // 不管是顶部栏的图标还是任务栏的图标都是Tray组件生成的
  if (process.platform === 'darwin') { // 如果是macOS平台
    let img = clipboard.readImage()
    let obj = []
    if (!img.isEmpty()) {
      // 从剪贴板来的图片默认转为png
      const imgUrl = 'data:image/png;base64,' + Buffer.from(img.toPNG(), 'binary').toString('base64')
      obj.push({
        width: img.getSize().width,
        height: img.getSize().height,
        imgUrl
      })
    }
    toggleWindow() // 打开小窗口
    setTimeout(() => {
      window.webContents.send('clipboardFiles', obj)
    }, 0)
  } else {
    window.hide()
    if (settingWindow === null) { // 如果主窗口未创建
      createSettingWindow() // 创建
      settingWindow.show() // 并打开
    } else {
      settingWindow.show() // 如果已存在，打开
      settingWindow.focus() // 并激活
    }
  }
})
```

### 窗口关闭与应用退出

在windows平台上，通常我们把应用的窗口都关了之后也就默认把这个应用给退出了。而如果在macOS系统上却不是这样。我们把应用的窗口关闭了，但是并非完全退出这个应用。所以为了实现这个操作习惯，我们也可以增加一个情况判断：

```js
app.on('window-all-closed', () => { // 当窗口都被关闭了
  if (process.platform !== 'darwin') { // 如果不是macOS
    app.quit() // 应用退出
  }
})
```

## 总结

本文简要地讲述了electron应用在跨平台开发的时候的一些注意事项。可能很多人会觉得奇怪我为啥把这个章节单独拎出来讲。很多时候我们只关注于应用的开发过程，把应用的功能实现是很多情况下的「终极」目标。然而真实情况是，应用的功能实现只是「基本」目标。一个应用要给用户使用的话必然不仅要考虑到应用的功能，还必须考虑用户的使用习惯。要站在用户的角度来做应用。而不是做自嗨型的应用。所以这篇文章也希望能够帮助想要开发electron应用的你。

本文很多都是我在开发`PicGo`的时候碰到的问题、踩的坑。也许文中简单的几句话背后就是我无数次的查阅和调试。希望这篇文章能够给你的`electron-vue`开发带来一些启发。文中相关的代码，你都可以在[PicGo](https://github.com/Molunerfinn/PicGo)的项目仓库里找到，欢迎star~如果本文能够给你带来帮助，那么将是我最开心的地方。如果喜欢，欢迎关注我的[博客](https://molunerfinn.com)以及本系列文章的后续进展。

> **注：文中的图片除未特地说明之外均属于我个人作品，需要转载请私信**











