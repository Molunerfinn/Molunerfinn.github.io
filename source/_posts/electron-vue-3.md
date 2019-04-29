---
title: Electron-vue开发实战2——引入基于Lodash的JSON数据库lowdb
tags: 
  - 前端
  - Vue
  - Electron
  - Electron-vue
categories:
  - Web
  - 开发
date: 2018-02-12 21:04:00
---
## 前言

前段时间，我用[electron-vue](https://github.com/SimulatedGREG/electron-vue)开发了一款跨平台（目前支持Mac和Windows）的免费开源的图床上传应用——[PicGo](https://github.com/Molunerfinn/PicGo)，在开发过程中踩了不少的坑，不仅来自应用的业务逻辑本身，也来自electron本身。在开发这个应用过程中，我学了不少的东西。因为我也是从0开始学习electron，所以很多经历应该也能给初学、想学electron开发的同学们一些启发和指示。故而写一份Electron的开发实战经历，用最贴近实际工程项目开发的角度来阐述。希望能帮助到大家。

预计将会从几篇系列文章或方面来展开：

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

如果之前的文章没阅读的朋友可以先从之前的文章跟着看。

<!-- more -->

## 数据持久化存储的必要性

不像平时很多人写的一些demo，就是请求一下api然后把web页面展示出来就了事了。electron应用毕竟是个桌面级应用，如果思维还留在纯web开发的思路上，那么也就失去了用electron的意义了吧。

数据持久化存储实际上对于后端很熟悉。通常是指的是把内存里的数据以不同的存储模型存储到磁盘上，在需要的时候再从存储模型里读取读入内存中的整个流程。这里面的存储模型通常就是我们熟悉的数据库。说到数据库，很多人会想到MySQL，Mongodb，SQLite等等。常见的这些数据库都是Server-Client模式的，需要启动服务端——通常我们装的就是这个。但是你一般很少见到叫别人装个桌面软件的同时，叫别人配数据库的吧。

因为有些数据我们必须在本地存下来，方便下次使用的时候读取。而对于electron来说，既然让用户装MySQL、Mongodb是不太优雅的解决办法的话，那么如果能用其他方式，将数据存到本地而不用用户操心如何存储的，对我们和用户来说都是一件好事。

## 纯JavaScript数据库的选择

既然是JS技术栈的，于是我就找了一些纯JavaScript实现的数据库。经过初步筛选，我找到如下两个：

1. [nedb](https://github.com/louischatriot/nedb) 7800star（2018-02-12）
2. [lowdb](https://github.com/typicode/lowdb) 7269star（2018-02-12）

### 比较

其中就目前来看，nedb用的更为广泛，star数更多（截止2018-02-12），而且有很多讲到nedb和electron配合使用的文章。不过，nedb已经有快两年没有维护了，而且原生不支持Promise，采用的是异步回调（虽然可以通过第三方插件实现Promise）。

lowdb是用JSON为基本存储结构基于lodash开发的，有lodash的加持，用起来很顺手。优势在于它在持续的维护，有不少好用的插件。并且很关键的是同步操作，采用链式调用的写法，写起来有种jQuery的感觉。再者，用JSON存储的数据，不管是调用还是备份都很方便，这也是让我很喜欢的一点。

综上，PicGo采用的是lowdb。

## lowdb的初始化

由于electron给main进程和renderer进程都置入了Node的`fs`模块，所以我们可以很方便的在两端都使用跟`fs`相关的操作。而lowdb本质上就是通过`fs`来读写JSON文件实现的，正好符合我们的要求。所以根据官方给出的文档，我们首先先初始化一下。

**为了操作`fs`更方便，不妨安装一个[fs-extra](https://github.com/jprichardson/node-fs-extra)。**

创建一个`datastore.js`文件：

```js
import Datastore from 'lowdb'
import FileSync from 'lowdb/adapters/FileSync'
import path from 'path'
import fs from 'fs-extra'
import { app } from 'electron'

const STORE_PATH = app.getPath('userData') // 获取electron应用的用户目录

const adapter = new FileSync(path.join(STORE_PATH, '/data.json')) // 初始化lowdb读写的json文件名以及存储路径

const db = Datastore(adapter) // lowdb接管该文件

export default db // 暴露出去

```

接着我们在main进程和renderer进程里就可以这样引入：

```js
import db from '../datastore' // 取决于你的datastore.js的位置
```

### 踩坑

如果仅仅是上面的基本操作，那么这篇文章未免也太简单了。关于electron引入lowdb的踩坑之路现在才开始。

#### 1. renderer进程要使用remote模块

首先由上面的初始化能明显看到一个问题。`app`模块是main进程里特有的，renderer进程应该使用`remote.app`模块。所以上面的代码在`renderer`进程里会报错。

因此第一次修改，使其既能跑在main进程也能跑在renderer进程：

```js
import Datastore from 'lowdb'
import FileSync from 'lowdb/adapters/FileSync'
import path from 'path'
import fs from 'fs-extra'
import { app, remote } from 'electron' // 引入remote模块

const APP = process.type === 'renderer' ? remote.app : app // 根据process.type来分辨在哪种模式使用哪种模块

const STORE_PATH = APP.getPath('userData') // 获取electron应用的用户目录

const adapter = new FileSync(path.join(STORE_PATH, '/data.json')) // 初始化lowdb读写的json文件名以及存储路径

const db = Datastore(adapter) // lowdb接管该文件

export default db // 暴露出去

```

#### 2. 开发模式和生产模式初始化路径问题

在开发模式的时候，通过`APP.getPath('userData')`获取到的路径形如：`/Users/molunerfinn/Library/Application Support/Electron`（macOS下）。这个是一个已经自动创建好的路径。所以在开发模式的时候，初始化路径是已经存在的。

然而在生产模式下不是这样。生产模式下，第一次打开应用的过程中，`APP.getPath('userData')`获取的路径并未创建，而`datastore.js`却已经被加载。所以这个时候初始化路径并不存在。用户在第一次打开应用的时候就会遇到如下报错：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fodwgwq9k6j20nc176dov)

所以我们必须在`datastore.js`里做一次路径是否存在的判断：

> 此处的fs是来自fs-extra模块

```js
if (process.type !== 'renderer') {
  if (!fs.pathExistsSync(STORE_PATH)) { // 如果不存在路径
    fs.mkdirpSync(STORE_PATH) // 就创建
  }
}
```

#### 3. 初始化数据

因为有的时候我们需要预先指定数据库的基本结构，比如是个数组，这样我们就初始化为`[]`。如果是个Object，有具体值，就指定为具体值。而初始化数据结构不应该在每次对数据读写的时候来判断，应该在数据库一开始创建的时候就初始化，所以写在`datastore.js`里是合适的。

比如我要初始化上传列表应该是一个数组，具体如下：

```js
if (!db.has('uploaded').value()) { // 先判断该值存不存在
  db.set('uploaded', []).write() // 不存在就创建
}
```

#### 4. 唯一标识的id字段

用过MySQL的人大多都会在表里初始化一个自增的id字段作为数据的唯一标识。而lowdb虽然无法很方便地创建一个自增的id字段，但是通过[lodash-id](https://github.com/typicode/lodash-id)这个插件可以很方便地为每个新增的数据自动加上一个唯一标识的id字段。

形如：

```json
{
  "height": 514,
  "type": "weibo",
  "width": 514,
  "id": "7f247aa7-ffeb-4bb1-87f1-a0d69824ec78"
}
```

初始化也很方便：

```js
// ...
import LodashId from 'lodash-id'
// ...

const db = Datastore(adapter)
db._.mixin(LodashId) // 通过._mixin()引入
```

### 初始化完整代码

通过上述的踩坑，PicGo的初始化代码如下，仅供参考：

```js
import Datastore from 'lowdb'
import LodashId from 'lodash-id'
import FileSync from 'lowdb/adapters/FileSync'
import path from 'path'
import fs from 'fs-extra'
import { remote, app } from 'electron'

const APP = process.type === 'renderer' ? remote.app : app
const STORE_PATH = APP.getPath('userData')

if (process.type !== 'renderer') {
  if (!fs.pathExistsSync(STORE_PATH)) {
    fs.mkdirpSync(STORE_PATH)
  }
}

const adapter = new FileSync(path.join(STORE_PATH, '/data.json'))

const db = Datastore(adapter)
db._.mixin(LodashId)

if (!db.has('uploaded').value()) {
  db.set('uploaded', []).write()
}

if (!db.has('picBed').value()) {
  db.set('picBed', {
    current: 'weibo'
  }).write()
}

if (!db.has('shortKey').value()) {
  db.set('shortKey', {
    upload: 'CommandOrControl+Shift+P'
  }).write()
}

export default db
```

## lowdb的基本操作

数据库的基本操作无非就是CURD。

> 它代表创建（Create）、更新（Update）、读取（Retrieve）和删除（Delete）操作。

下面介绍lowdb的基本使用方法。

### 创建

主要通过`set()`或者`defaults()`方法。其中`defaults()`专门针对空JSON文件进行初始化。（不过用set也是可以实现类似的，如上一小节说到的初始化）

```js
db.defaults({ posts: [], user: {}, count: 0 })
  .write() // 一定要显式调用write方法将数据存入JSON
```

**注意任何写的操作，都必须显式的使用`write()`方法来保存。**

### 读取

```js
db.get('posts').value() // []
```

当然还可以用lodash的一些方法来查询你的JSON。

比如`find()`

```js
db.get('posts')
  .find({ id: 1 })
  .value()
```

**注意任何读的操作，都必须显式使用`value()`方法来获取值。**

### 更新

通过不同的方法对不同的结构来更新。

比如针对对象就用赋值，针对数组就用`push()`或者`insert()`（lowdb-id提供的方法）

```js
db.get('posts').insert({ // 对数组进行insert操作
  title: 'xxx',
  content: 'xxxx'
}).write()
```

针对对象可以直接用`set()`来更新：

```js
db.set('user.name', 'typicode') // 通过set方法来对对象操作
  .write()
```

还可以这么写：

```js
db.set('user', {
  name: 'typicode'
}).write()
```

很灵活对吧。

针对原有的数据进行更新的可以用update。

```js
db.update('count', n => n + 1) // update方法使用已存在的值来操作
  .write()
```

### 删除

可以通过`remove()`方法删除一个符合条件的项：

```js
db.get('posts')
  .remove({ title: 'low!' })
  .write()
```

可以通过`unset`来删除一个属性：

```js
db.unset('user.name')
  .write()
```

还可以通过`lodash-id`提供的`removeById()`来删除指定id的项：

```js
db.get('posts')
  .removeById(id)
  .write()
```

## lowdb实际使用的坑

lowdb在使用的过程中会遇到一个大坑在于，如果就按照基本操作，那么有可能出现我在`main`进程里存入的值，在`renderer`进程里读不到。

为啥？因为直接引用的`db`实际上只是那个时刻在内存里的数据。lowdb在使用过程中会把JSON数据读入内存中。只有在需要写操作的时候才会将新的数据写入磁盘。

main进程和renderer进程拿到的db都是应用打开时所读取的。在没有额外处理的情况下，在main进程拿到的内存里的db，和renderer拿到的内存里的db不是同一个db，也就是所谓的不是一个db的两份引用，而是一个db的两份拷贝。main进程对其进行的操作，renderer进程是不知道的。换句话说，main进程对db进行了任何读写操作，renderer拿到的db依然是当初应用打开时所读取的db。所以就会遇到main进程更新了数据，而renderer进程依然无法拿到新的数据。

那有没有办法解决呢？有的。就是有点麻烦。那就是在所有的db操作的最开始，都重新读取一遍db的最新状态：

比如：

```js
db.read().get('xxx').value()

db.read().set('xxx', 'xxx')
```

强制在每个db操作前，都通过read()刷新一遍内存区，这样就能保证拿到的数据都是最新的啦。

## Vue里使用lowdb的便捷方法

类似于很多人会在Vue里把axios挂在vue的原型链上一样，我们也可以用类似的方法来方便我们在Vue里使用lowdb。

打开Vue项目的入口文件，通常是`main.js`

```js
// ...
import db from '../datastore'
import Vue from 'vue'
// ...

Vue.prototype.$db = db
```

这样我们就可以在项目里，用`this.$db`的方法来使用lowdb啦。

## 总结

本文详细地介绍了lowdb以及lowdb在electron里的使用。很多都是我在开发`PicGo`的时候碰到的问题、踩的坑。也许文中简单的几句话背后就是我无数次的查阅和调试。希望这篇文章能够给你的`electron-vue`开发带来一些启发。文中相关的代码，你都可以在[PicGo](https://github.com/Molunerfinn/PicGo)的项目仓库里找到。如果本文能够给你带来帮助，那么将是我最开心的地方。如果喜欢，欢迎关注我的[博客](https://molunerfinn.com)以及本系列文章的后续进展。

> **注：文中的图片除未特地说明之外均属于我个人作品，需要转载请私信**
