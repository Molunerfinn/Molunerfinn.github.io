---
title: Electron-vue开发实战6——开发插件系统之GUI部分
tags: 
  - 前端
  - Vue
  - Electron
  - Electron-vue
categories:
  - Web
  - 开发
date: 2019-03-17 11:30:00
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
8. 想到再写...

## 说明

`PicGo`是采用`electron-vue`开发的，所以如果你会`vue`，那么跟着一起来学习将会比较快。如果你的技术栈是其他的诸如`react`、`angular`，那么纯按照本教程虽然在render端（可以理解为页面）的构建可能学习到的东西不多，不过在main端(`Electron`的主进程）应该还是能学习到相应的知识的。

如果之前的文章没阅读的朋友可以先从[之前的文章](https://molunerfinn.com/tags/Electron-vue/)跟着看。并且如果没有看过前一篇CLI插件系统构建的朋友，需要先行阅读，本文涉及到的部分内容来自上一篇文章。

<!-- more -->

## 运行时的require

我们之前构建的插件系统是基于`Node.js`端的。对于`Electron`而言，main进程可以认为拥有`Node.js`环境，所以我们首先要在main进程里将其引入。而对于PicGo而言，由于上传流程已经完全抽离到`PicGo-Core`这个库里了，所以原本存在于Electron端的上传部分就可以精简整合成调用`PicGo-Core`的api来实现上传部分的逻辑了。

而在引入`PicGo-Core`的时候会遇到一个问题。在`Electron`端，由于我使用的脚手架是`Electron-vue`，它会将`main`进程和`renderer`进程都通过`Webapck`进行打包。由于`PicGo-Core`用于加载插件的部分使用的是`require`，在Node.js端很正常没问题。但是Webpack并不知道这些`require`是在运行时才需要调用的，它会认为这是构建时的「常规」`require`，也就会在打包的时候把你`require`的插件也打包进来。这样明显是不合理的，我们是运行时才`require`插件的，所以需要做一些手段来「绕开」`Webpack`的打包机制：

```js
// eslint-disable-next-line
const requireFunc = typeof __webpack_require__ === 'function' ? __non_webpack_require__ : require
const PicGo = requireFunc('picgo')
```

> 关于`__non_webpack_require__`的说明，可以查看[文档](https://webpack.docschina.org/api/module-variables/#__non_webpack_require__-webpack-特有变量-)。

打包之后会变成如下：

```js
const requireFunc = true ? require : require
const PicGo = requireFunc('picgo')
```

这样就可以避免PicGo-Core内部的`require`被`Webpack`也打包进去了。

## 「前后端」分离

`Electron`的`main`进程和`renderer`进程实际上你可以把它们看成我们平时Web开发的后端和前端。二者交流的工具也不再是`Ajax`，而是`ipcMain`和`ipcRenderer`。当然`renderer`本身能做的事情也不少，只不过这样说一下可能会好理解一点。相应的，我们的插件系统原本实现在`Node.js`端，是一个没有界面的工具，想要让它拥有「脸面」，其实也不过是在`renderer`进程里调用来自`main`进程里的插件系统暴露出来的api而已。这里我们举几个例子来说明。

### 简化原有流程

在以前PicGo上传图片需要经过很多步骤：

1. 通过[uploader](https://github.com/Molunerfinn/PicGo/blob/v1.6.2/src/main/utils/uploader.js)来接收图片，并通过[pic-bed-handler](https://github.com/Molunerfinn/PicGo/blob/v1.6.2/src/datastore/pic-bed-handler.js)来指定上传的图床。
2. 通过[img2base64](https://github.com/Molunerfinn/PicGo/blob/v1.6.2/src/main/utils/img2base64.js)来把图片统一转成`Base64`编码。
3. 通过指定的`imgUploader`（比如`qiniu`比如`weibo`等）来上传到指定的图床。

而如今整个底层上传流程系统已经被抽离出来，因此我们可以直接使用PicGo-Core实现的api来上传图片，只需定义一个[Uploader](https://github.com/Molunerfinn/PicGo/blob/dev/src/main/utils/uploader.js)类即可（下面的代码是简化版本）：

```js
import {
  app,
  Notification,
  BrowserWindow,
  ipcMain
} from 'electron'
import path from 'path'

// eslint-disable-next-line
const requireFunc = typeof __webpack_require__ === 'function' ? __non_webpack_require__ : require
const PicGo = requireFunc('picgo')
const STORE_PATH = app.getPath('userData')
const CONFIG_PATH = path.join(STORE_PATH, '/data.json')

class Uploader {
  constructor (img, webContents, picgo = undefined) {
    this.img = img
    this.webContents = webContents
    this.picgo = picgo
  }

  upload () {
    const win = BrowserWindow.fromWebContents(this.webContents) // 获取上传的窗口
    const picgo = this.picgo || new PicGo(CONFIG_PATH) // 获取上传的picgo实例
    picgo.config.debug = true // 方便调试
    // for picgo-core
    picgo.config.PICGO_ENV = 'GUI'
    let input = this.img // 传入的this.img是一个数组

    picgo.upload(input) // 上传图片，只用了一句话

    picgo.on('notification', message => { // 上传成功或者失败提示信息
      const notification = new Notification(message)
      notification.show()
    })

    picgo.on('uploadProgress', progress => { // 上传进度
      this.webContents.send('uploadProgress', progress)
    })

    return new Promise((resolve) => { // 返回一个Promise方便调用
      picgo.on('finished', ctx => { // 上传完成的事件
        if (ctx.output.every(item => item.imgUrl)) {
          resolve(ctx.output)
        } else {
          resolve(false)
        }
      })
      picgo.on('failed', ctx => { // 上传失败的事件
        const notification = new Notification({
          title: '上传失败',
          body: '请检查配置和上传的文件是否符合要求'
        })
        notification.show()
        resolve(false)
      })
    })
  }
}

export default Uploader
```

可以看出，由于在设计CLI插件系统的时候我们有考虑到设计好插件的生命周期，所以很多功能都可以通过生命周期的钩子、以及相应的一些事件来实现。比如图片上传完成就是通过`picgo.on('finished'， callback)`监听`finished`事件来实现的，而上传的进度与进度条显示就是通过`picgo.on('progress')`来实现的。它们的效果如下：

![upload-process](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/picgo-2.0.gif)

而且我们还可以通过接入`picgo`的生命周期，实现一些以前实现起来比较麻烦的功能，比如上传前重命名：

```js
picgo.helper.beforeUploadPlugins.register('renameFn', {
  handle: async ctx => {
    const rename = picgo.getConfig('settings.rename')
    const autoRename = picgo.getConfig('settings.autoRename')
    await Promise.all(ctx.output.map(async (item, index) => {
      let name
      let fileName
      if (autoRename) {
        fileName = dayjs().add(index, 'second').format('YYYYMMDDHHmmss') + item.extname
      } else {
        fileName = item.fileName
      }
      if (rename) { // 如果要重命名
        const window = createRenameWindow(win) // 创建重命名窗口
        await waitForShow(window.webContents) // 等待窗口打开
        window.webContents.send('rename', fileName, window.webContents.id) // 给窗口发送相应信息
        name = await waitForRename(window, window.webContents.id) // 获取重新命名后的文件名
      }
      item.fileName = name || fileName
    }))
  }
})
```

通过注册一个`beforeUploadPlugin`，在上传前判断是否需要「上传前重命名」，如果是，就创建窗口并等待用户输入重命名的结果，然后将重命名的`name`赋值给`item.fileName`供后续的流程使用。

我们还可以在`beforeTransform`阶段通知用户当前正在准备上传了：

```js
picgo.on('beforeTransform', ctx => {
  if (ctx.getConfig('settings.uploadNotification')) {
    const notification = new Notification({
      title: '上传进度',
      body: '正在上传'
    })
    notification.show()
  }
})
```

等等。所以实际上我们只需要在`main`进程完成相应的api，那么`renderer`进程做的事只不过是通过`ipcRenderer`来通过`main`进程调用这些api而已了。比如：

- 当用户拖动图片到上传区域，通过`ipcRenderer`通知`main`进程：

```js
this.$electron.ipcRenderer.send('uploadChoosedFiles', sendFiles)
```

- `main`进程监听事件并调用`Uploader`的`upload`方法：

```js
ipcMain.on('uploadChoosedFiles', async (evt, files) => {
  const input = files.map(item => item.path)
  const imgs = await new Uploader(input, evt.sender).upload() // 由于upload返回的是Promise
  // ...
})
```

就完成了一次「前后端」交互。其他方式上传（比如剪贴板上传）也同理，就不再赘述。

## 实现插件管理界面

光有插件系统没有插件也不行，所以我们需要实现一个插件管理的界面。而插件管理的功能（比如安装、卸载、更新）已经在CLI版本里实现了，所以这些功能我们只需要通过向上一节里说的调用`ipcRenderer`和`ipcMain`来调用相应api即可。

### 第三方插件搜索

在GUI界面我们需要一个很重要的功能就是「插件搜索」的功能。由于PicGo的插件统一是发布到[npm](https://www.npmjs.com/)的，所以其实我们可以通过npm的api来打到搜索插件的目的：

```js
getSearchResult (val) {
  // this.$http.get(`https://api.npms.io/v2/search?q=${val}`)
  this.$http.get(`https://registry.npmjs.com/-/v1/search?text=${val}`) // 调用npm的搜索api
    .then(res => {
      this.pluginList = res.data.objects.map(item => {
        return this.handleSearchResult(item) // 返回格式化的结果
      })
      this.loading = false
    })
    .catch(err => {
      console.log(err)
      this.loading = false
    })
},
handleSearchResult (item) {
  const name = item.package.name.replace(/picgo-plugin-/, '')
  let gui = false
  if (item.package.keywords && item.package.keywords.length > 0) {
    if (item.package.keywords.includes('picgo-gui-plugin')) {
      gui = true
    }
  }
  return {
    name: name,
    author: item.package.author.name,
    description: item.package.description,
    logo: `https://cdn.jsdelivr.net/npm/${item.package.name}/logo.png`,
    config: {},
    homepage: item.package.links ? item.package.links.homepage : '',
    hasInstall: this.pluginNameList.some(plugin => plugin === item.package.name.replace(/picgo-plugin-/, '')),
    version: item.package.version,
    gui,
    ing: false // installing or uninstalling
  }
}
```

通过搜索然后把结果显示到界面上就是如下：

![](https://raw.githubusercontent.com/Molunerfinn/test/master/test/search-plugin.gif)

没有安装的插件就会在右下角显示「安装」两个字样。


### 本地插件列表

当我们安装好插件之后，需要从本地获取插件列表。这个部分需要做一些处理。由于插件是安装在Node.js端的，所以我们需要通过`ipcRenderer`去向`main`进程发起获取插件列表的「请求」：

```js
this.$electron.ipcRenderer.send('getPluginList') // 发起获取插件的「请求」
this.$electron.ipcRenderer.on('pluginList', (evt, list) => { // 获取插件列表
  this.pluginList = list
  this.pluginNameList = list.map(item => item.name)
  this.loading = false
})
```

而获取插件列表以及相应信息我们需要在`main`端进行，并发送回去：

```js
ipcMain.on('getPluginList', event => {
  const picgo = new PicGo(CONFIG_PATH)
  const pluginList = picgo.pluginLoader.getList()
  const list = []
  for (let i in pluginList) {
   // 处理插件相关的信息
  }
  event.sender.send('pluginList', list) // 将插件信息列表发送回去
})
```

注意到由于`ipcMain`和`ipcRenderer`里收发数据的时候会自动经过`JSON.stringify`和`JSON.parse`，所以对于原来的一些属性是`function`之类无法被序列化的属性，我们要做一些处理，比如先执行它们得到结果：

```js
const handleConfigWithFunction = config => {
  for (let i in config) {
    if (typeof config[i].default === 'function') {
      config[i].default = config[i].default()
    }
    if (typeof config[i].choices === 'function') {
      config[i].choices = config[i].choices()
    }
  }
  return config
}
```

这样，在`renderer`进程里才能拿到完整的数据。

### 插件配置相关

当然光有安装、查看还不够，还需要让插件管理界面拥有其他功能，比如「卸载」、「更新」或者是配置功能，所以在每个安装成功后的插件卡片的右下角有个配置按钮可以弹出相应的菜单：

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/20190113160001.png)

菜单这个部分就是用`Electron`的`Menu`模块去实现了（我在之前的文章里已经有涉及，不再赘述），并没有特别复杂的地方。而这里比较关键的地方，就是当我点击`配置plugin-xxx`的时候，会弹出一个配置的对话框：

![](https://raw.githubusercontent.com/Molunerfinn/test/master/test/20190317161148.png)

这个配置对话框内的配置内容来自前文《开发CLI插件系统》里我们要求开发者定义好的`config`方法返回的配置项。由于插件开发者定义的`config`内容是[Inquirer.js](https://github.com/SBoudrias/Inquirer.js/)所要求的格式，便于在CLI环境下使用。但是它和我们平时使用的`form`表单的一些格式可能有些出入，所以需要「转义」一下，通过原始的`config`动态生成表单项：

```html
<div id="config-form">
  <el-form
    label-position="right"
    label-width="120px"
    :model="ruleForm"
    ref="form"
    size="mini"
  >
    <el-form-item
      v-for="(item, index) in configList"
      :label="item.name"
      :required="item.required"
      :prop="item.name"
      :key="item.name + index"
    >
      <el-input
        v-if="item.type === 'input' || item.type === 'password'"
        :type="item.type === 'password' ? 'password' : 'input'"
        v-model="ruleForm[item.name]"
        :placeholder="item.message || item.name"
      ></el-input>
      <el-select
        v-else-if="item.type === 'list'"
        v-model="ruleForm[item.name]"
        :placeholder="item.message || item.name"
      >
        <el-option
          v-for="(choice, idx) in item.choices"
          :label="choice.name || choice.value || choice"
          :key="choice.name || choice.value || choice"
          :value="choice.value || choice"
        ></el-option>
      </el-select>
      <el-select
        v-else-if="item.type === 'checkbox'"
        v-model="ruleForm[item.name]"
        :placeholder="item.message || item.name"
        multiple
        collapse-tags
      >
        <el-option
          v-for="(choice, idx) in item.choices"
          :label="choice.name || choice.value || choice"
          :key="choice.value || choice"
          :value="choice.value || choice"
        ></el-option>
      </el-select>
      <el-switch
        v-else-if="item.type === 'confirm'"
        v-model="ruleForm[item.name]"
        active-text="yes"
        inactive-text="no"
      >
      </el-switch>
    </el-form-item>
    <slot></slot>
  </el-form>
</div>
```

上面是针对`config`里不同的`type`转换成不同的Web表单控件的代码。下面是初始化的时候处理`config`的一些工作：

```js
watch: {
  config: {
    deep: true,
    handler (val) {
      this.ruleForm = Object.assign({}, {})
      const config = this.$db.read().get(`picBed.${this.id}`).value()
      if (val.length > 0) {
        this.configList = cloneDeep(val).map(item => {
          let defaultValue = item.default !== undefined
            ? item.default : item.type === 'checkbox'
              ? [] : null // 处理默认值
          if (item.type === 'checkbox') { // 处理checkbox选中值
            const defaults = item.choices.filter(i => {
              return i.checked
            }).map(i => i.value)
            defaultValue = union(defaultValue, defaults)
          }
          if (config && config[item.name] !== undefined) { // 处理默认值
            defaultValue = config[item.name]
          }
          this.$set(this.ruleForm, item.name, defaultValue)
          return item
        })
      }
    },
    immediate: true // 立即执行
  }
}
```

经过上述处理，就可以将原本用于CLI的配置项，近乎「无缝」地迁移到Web（GUI）端了。其实这也是vue-cli3的ui版本实现的思路，大同小异。

## 实现特有的guiApi

不过既然是GUI软件了，只通过调用CLI实现的功能明显是不够丰富的。因此我也为`PicGo`实现了一些特有的`guiApi`提供给插件的开发者，让插件的可玩性更强。当然不同的软件给予插件的GUI能力是不一样的，因此不能一概而论。我仅以`PicGo`为例，讲述我对于`PicGo`所提供的`guiApi`的理解和看法。下面我就来说说这部分是如何实现的。

由于PicGo本质是一个上传系统，所以用户在上传图片的时候，很多插件底层的东西和功能实际上是看不到的。如果要让插件的功能更加丰富，就需要让插件有自己的「可视化」入口让用户去使用。因此对于PicGo而言，我给予插件的「可视化」入口就放在插件配置的界面里——除了给插件默认的配置菜单之外，还给予插件自己的菜单项供用户使用：

![](https://i.loli.net/2019/01/12/5c39a2f60a32a.png)

这个实现也很容易，只要插件在自己的`index.js`文件里暴露一个`guiMenu`的选项，就可以生成自己的菜单：

```js
const guiMenu = ctx => {
  return [
    {
      label: '打开InputBox',
      async handle (ctx, guiApi) {
        // do something...
      }
    },
    {
      label: '打开FileExplorer',
      async handle (ctx, guiApi) {
        // do something...
      }
    },
    // ...
  ]
}
```

可以看到菜单项可以自定义，点击之后的操作也可以自定义，因此给予了插件很大的自由度。可以注意到，在点击菜单的时候会触发`handle`函数，这个函数里会传入一个`guiApi`，这个就是本节的重点了。就目前而言，`guiApi`实现了如下功能：

1. `showInputBox([option])` 调用之后打开一个输入弹窗，可以用于接受用户输入。
2. `showFileExplorer([option])` 调用之后打开一个文件浏览器，可以得到用户选择的文件（夹）路径。
3. `upload([file])` 调用之后使用PicGo底层来上传，可以实现自动更新相册图片、上传成功后自动将URL写入剪贴板。
4. `showNotificaiton(option)` 调用之后弹出系统通知窗口。

上面api我们可以通过诸如`guiApi.showInputBox()`、`guiApi.showFileExplorer()`等来实现调用。这里面的例子实现思路都差不多，我简单以`guiApi.showFileExplorer()`来做讲解。

当我们在`renderer`界面点击插件实现的某个菜单之后，实际上是通过调用`ipcRenderer`向`main`进程传播了一次事件：

```js
if (plugin.guiMenu) {
  menu.push({
    type: 'separator'
  })
  for (let i of plugin.guiMenu) {
    menu.push({
      label: i.label,
      click () { // 当点击的时候，发送当前的插件名和当前菜单项的名字
        _this.$electron.ipcRenderer.send('pluginActions', plugin.name, i.label)
      }
    })
  }
}
```

于是在`main`进程，我们通过监听这个事件，来调用相应的`guiApi`：

```js
const handlePluginActions = (ipcMain, CONFIG_PATH) => {
  ipcMain.on('pluginActions', (event, name, label) => {
    const picgo = new PicGo(CONFIG_PATH)
    const plugin = picgo.pluginLoader.getPlugin(`picgo-plugin-${name}`)
    const guiApi = new GuiApi(ipcMain, event.sender, picgo) // 实例化guiApi
    if (plugin.guiMenu && plugin.guiMenu(picgo).length > 0) {
      const menu = plugin.guiMenu(picgo)
      menu.forEach(item => {
        if (item.label === label) { // 找到相应的label，执行插件的`handle`
          item.handle(picgo, guiApi)
        }
      })
    }
  })
}
```

而`guiApi`的实现类[GuiApi](https://github.com/Molunerfinn/PicGo/blob/dev/src/main/utils/guiApi.js)其实特别简单：

```js
import {
  dialog,
  BrowserWindow,
  clipboard,
  Notification
} from 'electron'
import db from '../../datastore'
import Uploader from './uploader'
import pasteTemplate from './pasteTemplate'
const WEBCONTENTS = Symbol('WEBCONTENTS')
const IPCMAIN = Symbol('IPCMAIN')
const PICGO = Symbol('PICGO')
class GuiApi {
  constructor (ipcMain, webcontents, picgo) {
    this[WEBCONTENTS] = webcontents
    this[IPCMAIN] = ipcMain
    this[PICGO] = picgo
  }

  /**
   * for plugin show file explorer
   * @param {object} options
   */
  showFileExplorer (options) {
    if (options === undefined) {
      options = {}
    }
    return new Promise((resolve, reject) => {
      dialog.showOpenDialog(BrowserWindow.fromWebContents(this[WEBCONTENTS]), options, filename => {
        resolve(filename)
      })
    })
  }
}
```

实际上就是去调用一些`Electron`的方法，甚至是你自己封装的一些方法，返回值是一个新的`Promise`对象。这样插件开发者就可以通过`async`和`await`来方便获取这些方法的返回值了：

```js
const guiMenu = ctx => {
  return [
    {
      label: '打开文件浏览器',
      async handle (ctx, guiApi) {
        // 通过await获取用户所选的文件路径
        const files = await guiApi.showFileExplorer({
          properties: ['openFile', 'multiSelections']
        })
        console.log(files)
      }
    }
  ]
}
```

## 小结

至此，一个GUI插件系统的关键部分我们就基本实现了。除了整合了CLI插件系统的几乎所有功能之外，我们还提供了独特的`guiApi`给插件开发者无限的想象空间，也给用户带来更好的插件体验。可以说插件系统的实现，让`PicGo`有了更多的可玩性。关于`PicGo`目前的插件，欢迎查看[Awesome-PicGo](https://github.com/PicGo/Awesome-PicGo)的列表。以下罗列一些我觉得比较有用或者有意思的插件：

1. [vs-picgo](https://github.com/Spades-S/vs-picgo) 在VSCode里使用PicGo（无需安装GUI！）
2. [picgo-plugin-pic-migrater](https://github.com/PicGo/picgo-plugin-pic-migrater) 可以迁移你的Markdown里的图片地址到你默认指定的图床，哪怕是本地图片也可以迁移到云端！
3. [picgo-plugin-github-plus](https://github.com/zWingz/picgo-plugin-github-plus) 增强版GitHub图床，支持了同步图床以及同步删除操作（删除本地图片也会把GitHub上的图片删除）
4. [picgo-plugin-web-uploader](https://github.com/yuki-xin/picgo-plugin-web-uploader) 支持[PicUploader](https://github.com/xiebruce/PicUploader)配置的图床插件
5. [picgo-plugin-qingstor-uploader](https://github.com/chengww5217/picgo-plugin-qingstor-uploader) 支持青云云存储的图床插件
6. [picgo-plugin-blog-uploader](https://github.com/chengww5217/picgo-plugin-blog-uploader) 支持掘金、简书和CSDN来做图床的图床插件

如果你也想为PicGo开发插件，欢迎阅读[开发文档](https://picgo.github.io/PicGo-Core-Doc/zh/)，PicGo有你更精彩哈哈！

本文很多都是我在开发`PicGo`的时候碰到的问题、踩的坑。也许文中简单的几句话背后就是我无数次的查阅和调试。希望这篇文章能够给你的`electron-vue`开发带来一些启发。文中相关的代码，你都可以在[PicGo](https://github.com/Molunerfinn/PicGo)和[PicGo-Core](https://github.com/PicGo/PicGo-Core)的项目仓库里找到，欢迎star~如果本文能够给你带来帮助，那么将是我最开心的地方。如果喜欢，欢迎关注我的[博客](https://molunerfinn.com)以及[本系列文章](https://molunerfinn.com/tags/Electron-vue/)的后续进展。

> **注：文中的图片除未特地说明之外均属于我个人作品，需要转载请私信**

## 参考文献

感谢这些高质量的文章：

1. [用Node.js开发一个Command Line Interface (CLI)](https://zhuanlan.zhihu.com/p/38730825)
2. [Node.js编写CLI的实践](https://zhuanlan.zhihu.com/p/26895282)
3. [Node.js模块机制](http://www.infoq.com/cn/articles/nodejs-module-mechanism)
4. [前端插件系统设计与实现](https://onetwo.ren/%E5%89%8D%E7%AB%AF%E6%8F%92%E4%BB%B6%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1/)
5. [Hexo插件机制分析](https://blog.csdn.net/kyfxbl/article/details/47787827)
6. [如何实现一个简单的插件扩展](http://blog.yunplus.io/%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E6%8F%92%E4%BB%B6%E6%89%A9%E5%B1%95/)
7. [使用NPM发布与维护TypeScript模块](https://fenying.net/2017/12/02/publish-to-npm/)
8. [typescript npm 包例子](https://github.com/basarat/ts-npm-module)
9. [通过travis-ci发布npm包](https://docs.travis-ci.com/user/deployment/npm/)
10. [Dynamic load module in plugin from local project node_modules folder](https://discuss.atom.io/t/dynamically-load-module-in-plugin-from-local-project-node-modules-folder/42930/2)
11. [跟着老司机玩转Node命令行](https://aotu.io/notes/2016/08/09/command-line-development/index.html)
12. 以及没来得及记录的那些好文章，感谢你们！
