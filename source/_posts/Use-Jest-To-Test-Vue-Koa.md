---
title: 全栈测试实战：用Jest测试Vue+Koa全栈应用
tags: 
  - 前端
  - Nodejs
  - Vue
categories:
  - Web
  - 开发
  - Nodejs
date: 2017-11-15 21:34:00
---

## 前言

今年一月份的时候我写了一个[Vue+Koa的全栈应用](https://github.com/Molunerfinn/vue-koa-demo)，以及相应的[配套教程](https://molunerfinn.com/Vue+Koa/)，得到了很多的好评。同时我也在和读者交流的过程中不断认识到不足和缺点，于是也对此进行了不断的更新和完善。本次带来的完善是加入和完整的前后端测试。相信对于很多学习前端的朋友来说，`测试`这个东西似乎是个熟悉的陌生人。你听过，但是你未必做过。如果你对前端（以及nodejs端）测试很熟悉，那么本文的帮助可能不大，不过我很希望能得到你们提出的宝贵意见！

<!--more-->

## 简介

和上一篇[全栈开发实战：用Vue2+Koa1开发完整的前后端项目](https://molunerfinn.com/Vue+Koa/)一样，本文从测试新手的角度出发（默认了解Koa并付诸实践，了解Vue并付诸实践，但是并无测试经历），在已有的项目上从0开始构建我们的全栈测试系统。可以了解到测试的意义，[Jest](https://facebook.github.io/jest/)测试框架的搭建，前后端测试的异同点，如何写测试用例，如何查看测试结果并提升我们的测试覆盖率，100%测试覆盖率是否是必须，以及在搭建测试环境、以及测试本身过程中遇到的各种疑难杂症。希望可以作为入门前端以及Node端测试的文章吧。

## 项目结构

有了之前的项目结构作为骨架，加入Jest测试框架就很简单了。

```
.
├── LICENSE
├── README.md
├── .env  // 环境变量配置文件
├── app.js  // Koa入口文件
├── build // vue-cli 生成，用于webpack监听、构建
│   ├── build.js
│   ├── check-versions.js
│   ├── dev-client.js
│   ├── dev-server.js
│   ├── utils.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   └── webpack.prod.conf.js
├── config // vue-cli 生成&自己加的一些配置文件
│   ├── default.conf
│   ├── dev.env.js
│   ├── index.js
│   └── prod.env.js
├── dist // Vue build 后的文件夹
│   ├── index.html // 入口文件
│   └── static // 静态资源
├── env.js // 环境变量切换相关 <-- 新
├── .env // 开发、上线时的环境变量 <-- 新
├── .env.test // 测试时的环境变量 <-- 新
├── index.html // vue-cli生成，用于容纳Vue组件的主html文件。单页应用就只有一个html
├── package.json // npm的依赖、项目信息文件、Jest的配置项 <-- 新
├── server // Koa后端，用于提供Api
│   ├── config // 配置文件夹
│   ├── controllers // controller-控制器
│   ├── models // model-模型
│   ├── routes // route-路由
│   └── schema // schema-数据库表结构
├── src // vue-cli 生成&自己添加的utils工具类
│   ├── App.vue // 主文件
│   ├── assets // 相关静态资源存放
│   ├── components // 单文件组件
│   ├── main.js // 引入Vue等资源、挂载Vue的入口js
│   └── utils // 工具文件夹-封装的可复用的方法、功能
├── test
│   ├── sever // 服务端测试 <-- 新
│   └── client // 客户端（前端）测试 <-- 新
└── yarn.lock // 用yarn自动生成的lock文件
```

可以看到新增的或者说更新的东西只有几个：

1. 最主要的test文件夹，包含了客户端（前端）和服务端的测试文件
2. env.js以及配套的`.env`、`.env.test`，是跟测试相关的环境变量
3. package.json，更新了一些依赖以及Jest的配置项

> 主要环境：Vue2，Koa2，Nodejs v8.9.0

## 测试用到的一些关键依赖

以下依赖的版本都是本文所写的时候的版本，或者更旧一些

1. jest: ^21.2.1
2. babel-jest: ^21.2.0
3. supertest: ^3.0.0
4. dotenv: ^4.0.0

剩下依赖可以项目[demo仓库](https://github.com/Molunerfinn/vue-koa-demo)。

## 搭建Jest测试环境

对于测试来说，我也是个新手。至于为什么选择了[Jest](https://facebook.github.io/jest/)，而不是其他框架（例如mocha+chai、jasmine等），我觉得有如下我自己的观点（当然你也可以不采用它）：

1. 由Facebook开发，保证了更新速度以及框架质量
2. 它有很多集成的功能（比如断言库、比如测试覆盖率）
3. 文档完善，配置简单
4. 支持typescript，我在学习typescript的时候也用了Jest来写测试
5. Vue官方的单元测试框架[vue-test-utils](https://github.com/vuejs/vue-test-utils)专门有配合Jest的测试说明
6. 支持快照功能，对前端单元测试是一大利好
7. 如果你是React技术栈，Jest天生就适配React

### 安装

```bash
yarn add jest -D

#or

npm install jest --save-dev
```

很简单对吧。

### 配置

由于我项目的Koa后端用的是ES modules的写法而不是Nodejs的Commonjs的写法，所以是需要babel的插件来进行转译的。否则你运行测试用例的时候，将会出现如下问题：

```
 ● Test suite failed to run

    /Users/molunerfinn/Desktop/work/web/vue-koa-demo/test/sever/todolist.test.js:1
    ({"Object.<anonymous>":function(module,exports,require,__dirname,__filename,global,jest){import _regeneratorRuntime from 'babel-runtime/regenerator';import _asyncToGenerator from 'babel-runtime/helpers/asyncToGenerator';var _this = this;import server from '../../app.js';
                                                                                             ^^^^^^

    SyntaxError: Unexpected token import

      at ScriptTransformer._transformAndBuildScript (node_modules/jest-runtime/build/script_transformer.js:305:17)
          at Generator.next (<anonymous>)
          at new Promise (<anonymous>)
```

看了官方github的[README](https://github.com/facebook/jest#using-babel)发现应该是`babel-jest`没装。

```bash
yarn add babel-jest -D

#or

npm install babel-jest --save-dev
```

> 但是奇怪的是，文档里说：Note: babel-jest is automatically installed when installing Jest and will automatically transform files if a babel configuration exists in your project. 也就是babel-jest在jest安装的时候便会自动安装了。这点需要求证。

然而发现运行测试用例的时候还是出了上述问题，查阅了相关[issue](https://github.com/facebook/jest/issues/2081)之后，我给出两种解决办法：

都是修改项目目录下的`.babelrc`配置文件，增加`env`属性，配置`test`环境如下：

**1.** 增加`presets`

```json
"env": {
  "test": {
    "presets": ["env", "stage-2"] // 采用babel-presents-env来转译
  }
}
```

**2.** 或者增加`plugins`

```json
"env": {
  "test": {
    "plugins": ["transform-es2015-modules-commonjs"] // 采用plugins来讲ES modules转译成Commonjs modules
  }
}
```

再次运行，编译通过。

> 通常我们将测试文件（\*.test.js或\*.spec.js）放置在项目的test目录下。Jest将会自动运行这些测试用例。值得一提的是，通常我们将基于`TDD`的测试文件命名为`*.test.js`，把基于`BDD`的测试文件命名为`*.spec.js`。这二者的区别可以看这篇[文章](http://www.cnblogs.com/ustbwuyi/archive/2012/10/26/2741223.html)

我们可以在`package.json`的`scripts`字段里加入`test`的命令（如果原本存在则换一个名字，不要冲突）

```json
"scripts": {
  // ...其他命令
  "test": "jest"
  // ...其他命令
},
```

这样我们就可以在终端直接运行`npm test`来执行测试了。下面我们先来从后端的Api测试开始写起。

## Koa后端Api测试

重现一下之前的应用的操作流程，可以发现应用分为登录前和登录后两种状态。

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fl97i5zzb0g20z30ixk77.gif)

可以根据操作流程或者后端api的结构来写测试。如果根据操作流程来写测试就可以分为登录前和登录后。如果根据后端api的结构的话，就可以根据routes或者controllers的结构、功能来写测试。

由于本例登录前和登录后的api基本上是分开的，所以我主要根据上述后者（routes或controllers）来写测试。

到此需要解释一下一般来说（写）测试的步骤：

1. 写测试说明，针对你的每条测试说明测试了什么功能，预期结果是什么。
2. 写测试主体，通常是 输入 -> 输出。
3. 判断测试结果，拿输出和预期做对比。如果输出和预期相符，则测试通过。反之，不通过。

在`test`文件夹下新建一个`server`文件夹。然后创建一个`user.spec.js`文件。

我们可以通过

```js
import server from '../../app.js'
```

的方式将我们的Koa应用的主入口文件引入。但是此时遇到了一个问题。我们如何对这个server发起http请求，并对其的返回结果做出判断呢？

在阅读了[Async testing Koa with Jest](https://hackernoon.com/async-testing-koa-with-jest-1b6e84521b71)以及[A clear and concise introduction to testing Koa with Jest and Supertest](https://www.valentinog.com/blog/testing-api-koa-jest/)这两篇文章之后，我决定使用[supertest](https://github.com/visionmedia/supertest)这个工具了。它是专门用来测试nodejs端HTTP server的测试工具。它内封了[superagent](https://github.com/visionmedia/superagent)这个著名的Ajax请求库。并且支持Promise，意味着我们对于异步请求的结果也能通过`async await`的方式很好的控制了。

安装：

```bash
yarn add supertest -D

#or

npm install supertest --save-dev
```

现在开始着手写我们第一个测试用例。先写一个针对登录功能的吧。当我们输入了错误的用户名或者密码的时候将无法登录，后端返回的参数里，success会是false。

```js
// test/server/user.spec.js

import server from '../../app.js'
import request from 'supertest'

afterEach(() => {
  server.close() // 当所有测试都跑完了之后，关闭server
})

// 如果输入用户名为Molunerfinn，密码为1234则无法登录。正确应为molunerfinn和123。
test('Failed to login if typing Molunerfinn & 1234', async () => { // 注意用了async
  const response = await request(server) // 注意这里用了await
                    .post('/auth/user') // post方法向'/auth/user'发送下面的数据
                    .send({
                      name: 'Molunerfinn',
                      password: '1234'
                    })
  expect(response.body.success).toBe(false) // 期望回传的body的success值是false（代表登录失败）
})
```

> 上述例子中，test()方法能接受3个参数，第一个是对测试的描述(string)，第二个是回调函数(fn)，第三个是延时参数(number)。本例不需要延时。然后expect()函数里放输出，再用各种[match](https://facebook.github.io/jest/docs/en/expect.html)方法来将预期和输出做对比。

在终端执行`npm test`，紧张地希望能跑通也许是人生的第一个测试用例。结果我得到如下关键的报错信息：

```
 ● Post todolist failed if not give the params

    TypeError: app.address is not a function
 ...

 ● Post todolist failed if not give the params

    TypeError: _app2.default.close is not a function
```


这是怎么回事？说明我们import进来的server看来并没有close、address等方法。原因在于我们在`app.js`里最后一句：

```js
export default app
```

此处export出来的是一个对象。但我们实际上需要一个function。

在谷歌的过程中，找到两种解决办法：

> 参考[解决办法1](https://segmentfault.com/q/1010000006906863)和[解决办法2](https://hackernoon.com/async-testing-koa-with-jest-1b6e84521b71)

**1.** 修改`app.js`

将

```js
app.listen(8889, () => {
  console.log(`Koa is listening in 8889`)
})

export default app
```

改为

```js
export default app.listen(8889, () => {
  console.log(`Koa is listening in 8889`)
})
```

即可。

**2.** 修改你的test文件：

在里要用到`server`的地方都改为`server.callback()`：

```js
const response = await request(server.callback())
                    .post('/auth/user')
                    .send({
                      name: 'Molunerfinn',
                      password: '1234'
                    })
```

我采用的是第一种做法。

改完之后，顺利通过：

```
 PASS  test/sever/user.test.js
  ✓ Failed to login if typing Molunerfinn & 1234 (248ms)
```

然而此时发现一个问题，为何测试结束了，jest还占用着终端进程呢？我想要的是测试完jest就自动退出了。查了一下文档，发现它的cli有个参数`--forceExit`能解决这个问题，于是就把`package.json`里的`test`命令修改一下（后续我们还将修改几次）加上这个参数：

```json
"scripts": {
  // ...其他命令
  "test": "jest --forceExit"
  // ...其他命令
},
```

再测试一遍，发现没问题。这样一来我们就可以继续依葫芦画瓢，把`auth/*`这个路由的功能都测试一遍：

```js
// server/routes/auth.js

import auth from '../controllers/user.js'
import koaRouter from 'koa-router'
const router = koaRouter()

router.get('/user/:id', auth.getUserInfo) // 定义url的参数是id
router.post('/user', auth.postUserAuth)

export default router
```

测试用例如下：

```js
import server from '../../app.js'
import request from 'supertest'

afterEach(() => {
  server.close()
})

test('Failed to login if typing Molunerfinn & 1234', async () => {
  const response = await request(server)
                    .post('/auth/user')
                    .send({
                      name: 'Molunerfinn',
                      password: '1234'
                    })
  expect(response.body.success).toBe(false)
})

test('Successed to login if typing Molunerfinn & 123', async () => {
  const response = await request(server)
                    .post('/auth/user')
                    .send({
                      name: 'Molunerfinn',
                      password: '123'
                    })
  expect(response.body.success).toBe(true)
})

test('Failed to login if typing MARK & 123', async () => {
  const response = await request(server)
                    .post('/auth/user')
                    .send({
                      name: 'MARK',
                      password: '123'
                    })
  expect(response.body.info).toBe('用户不存在！')
})

test('Getting the user info is null if the url is /auth/user/10', async () => {
  const response = await request(server)
                    .get('/auth/user/10')
  expect(response.body).toEqual({})
})

test('Getting user info successfully if the url is /auth/user/2', async () => {
  const response = await request(server)
                    .get('/auth/user/2')
  expect(response.body.user_name).toBe('molunerfinn')
})
```

都很简洁易懂，看描述+预期你就能知道在测试什么了。不过需要注意一点的是，我们用到了`toBe()`和`toEqual()`两个方法。乍一看好像没有区别。实际上有大区别。

简单来说，`toBe()`适合`===`这个判断条件。比如`1 === 1`，`'hello' === 'hello'`。但是`[1] === [1]`是错的。具体原因不多说，js的基础。所以要判断比如数组或者对象相等的话需要用`toEqual()`这个方法。

OK，接下去我们开始测试`api/*`这个路由。

在`test`目录下创建一个叫做`todolits.spec.js`的文件：

有了上一个测试的经验，测试这个其实也不会有多大的问题。首先我们来测试一下当我们没有携带上JSON WEB TOKEN的header的话，服务端是不是返回401错误：

```js
import server from '../../app.js'
import request from 'supertest'

afterEach(() => {
  server.close()
})

test('Getting todolist should return 401 if not set the JWT', async () => {
  const response = await request(server)
                    .get('/api/todolist/2')
  expect(response.status).toBe(401)
})
```

一切看似没问题，但是运行的时候却报错了：

```
console.error node_modules/jest-jasmine2/build/jasmine/Env.js:194
    Unhandled error

console.error node_modules/jest-jasmine2/build/jasmine/Env.js:195
  Error: listen EADDRINUSE :::8888
      at Object._errnoException (util.js:1024:11)
      at _exceptionWithHostPort (util.js:1046:20)
      at Server.setupListenHandle [as _listen2] (net.js:1351:14)
      at listenInCluster (net.js:1392:12)
      at Server.listen (net.js:1476:7)
      at Application.listen (/Users/molunerfinn/Desktop/work/web/vue-koa-demo/node_modules/koa/lib/application.js:64:26)
      at Object.<anonymous> (/Users/molunerfinn/Desktop/work/web/vue-koa-demo/app.js:60:5)
      at Runtime._execModule (/Users/molunerfinn/Desktop/work/web/vue-koa-demo/node_modules/jest-runtime/build/index.js:520:13)
      at Runtime.requireModule (/Users/molunerfinn/Desktop/work/web/vue-koa-demo/node_modules/jest-runtime/build/index.js:332:14)
      at Runtime.requireModuleOrMock (/Users/molunerfinn/Desktop/work/web/vue-koa-demo/node_modules/jest-runtime/build/index.js:408:19)
```

看来是因为同时运行了两个Koa实例导致了监听端口的冲突。所以我们需要让Jest按顺序执行。查阅官方文档，发现了[runInBand](http://facebook.github.io/jest/docs/en/cli.html#runinband)这个参数正是我们想要的。

所以修改`package.json`里的`test`命令如下：

```json
"scripts": {
  // ...其他命令
  "test": "jest --forceExit --runInBand"
  // ...其他命令
},
```

再次运行，成功通过！

接下来遇到一个问题。我们的JWT的token原本是登录成功后生成并派发给前端的。如今我们测试api的时候并没有经过登录那一步。所以要测试的时候要用的token的话，我觉得有两种办法：

1. 增加测试的时候的api接口，不需要经过`koa-jwt`的验证。但是这种方法对项目有入侵性的影响，如果有的时候我们需要从token获取信息的话就有问题了。
2. 后端预先生成一个合法的token，然后测试的时候用上这个测试的token即可。不过这种办法的话就需要保证token不能泄露。

我采用第二种办法。为了读者使用方便我是预先生成一个token然后用一个变量存起来的。（真正的开发环境下应对将测试的token放置在项目环境变量.env中）

接下来我们测试一下数据库的四大操作：增删改查。不过我们为了一次性将这四个接口都测试一遍可以按照这个顺序：增查改删。其实就是先增加一个todo，然后查找的时候将id记录下来。随后可以用这个id进行更新和删除。

```js
import server from '../../app.js'
import request from 'supertest'

afterEach(() => {
  server.close()
})

const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoibW9sdW5lcmZpbm4iLCJpZCI6MiwiaWF0IjoxNTA5ODAwNTg2fQ.JHHqSDNUgg9YAFGWtD0m3mYc9-XR3Gpw9gkZQXPSavM' // 预先生成的token

let todoId = null // 用来存放测试生成的todo的id

test('Getting todolist should return 401 if not set the JWT', async () => {
  const response = await request(server)
                    .get('/api/todolist/2')
  expect(response.status).toBe(401)
})

// 增
test('Created todolist successfully if set the JWT & correct user', async () => { 
  const response = await request(server)
                    .post('/api/todolist')
                    .send({
                      status: false,
                      content: '来自测试',
                      id: 2
                    })
                    .set('Authorization', 'Bearer ' + token) // header处加入token验证
  expect(response.body.success).toBe(true)
})

// 查
test('Getting todolist successfully if set the JWT & correct user', async () => {
  const response = await request(server)
                    .get('/api/todolist/2')
                    .set('Authorization', 'Bearer ' + token)
  response.body.result.forEach((item, index) => {
    if (item.content === '来自测试') todoId = item.id // 获取id
  })
  expect(response.body.success).toBe(true)
})

// 改
test('Updated todolist successfully if set the JWT & correct todoId', async () => {
  const response = await request(server)
                    .put(`/api/todolist/2/${todoId}/0`) // 拿id去更新
                    .set('Authorization', 'Bearer ' + token)
  expect(response.body.success).toBe(true)
})

// 删
test('Removed todolist successfully if set the JWT & correct todoId', async () => {
  const response = await request(server)
                    .delete(`/api/todolist/2/${todoId}`)
                    .set('Authorization', 'Bearer ' + token)
  expect(response.body.success).toBe(true)
})
```

对照着api的4大接口，我们已经将它们都测试了一遍。那是不是我们对于服务端的测试已经结束了呢？其实不是的。要想保证后端api的健壮性，我们得将很多情况都考虑到。但是人为的去排查每个条件、语句什么的必然过于繁琐和机械。于是我们需要一个指标来帮我们确保测试的全面性。这就是测试覆盖率了。

### 后端api测试覆盖率

上面说过，Jest是自带了测试覆盖率功能的（其实就是基于[istanbul](https://github.com/gotwarlost/istanbul)这个工具来生成测试覆盖率的）。要如何开启呢？这里我还走了不少坑。

通过阅读官方的[配置文档](http://facebook.github.io/jest/docs/en/configuration.html)，我确定了几个需要开启的参数：

1. coverageDirectory，指定输出测试覆盖率报告的目录
2. coverageReporters，指定输出的测试覆盖率报告的形式，具体可以参考istanbul的[说明](https://istanbul.js.org/docs/advanced/alternative-reporters/)
3. collectCoverage，是否要收集覆盖率信息，当然是。
4. mapCoverage，由于我们的代码经过babel-jest转译，所以需要开启sourcemap来让Jest能够把测试结果定位到源代码上而不是编译的代码上。
5. verbose，用于显示每个测试用例的通过与否。

于是我们需要在`package.json`里配置一个Jest字段（不是在scripts字段里配置，而是和scripts在同一级的字段），来配置Jest。

配置如下：

```json
"jest": {
  "verbose": true,
  "coverageDirectory": "coverage",
  "mapCoverage": true,
  "collectCoverage": true,
  "coverageReporters": [
    "lcov", // 会生成lcov测试结果以及HTML格式的漂亮的测试覆盖率报告
    "text" // 会在命令行界面输出简单的测试报告
  ]
}
```

然后我们再进行一遍测试，可以看到在终端里已经输出了简易的测试报告总结：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1flc95bhjwmj20y80lijus.jpg)

从中我们能看到一些字段是100%，而一些不是100%。最后一列`Uncovered Lines`就是告诉我们，测试里没有覆盖到的代码行。为了更直观地看到测试的结果报告，可以到项目的根目录下找到一个`coverage`的目录，在`lcov-report`目录里有个`index.html`就是输出的html报告。打开来看看：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fld7tv6c91j21z20h0n0f.jpg)

首页是个概览，跟命令行里输出的内容差不多。不过我们可以往深了看，可以点击左侧的File提供的目录：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fld8bgzj3gj21z20hotc4.jpg)

然后我们可以看到没有被覆盖到代码行数（50）以及有一个函数没有被测试到：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fld8erd8g6j215w05a757.jpg)

通常我们没有测试到的函数也伴随着代码行数没有被测试到。我们可以看到在本例里，app的`error`事件没有被触发过。想想也是的，我们的测试都是建立在合法的api请求的基础上的。所以自然不会触发`error`事件。因此我们需要写一个测试用例来测试这个`.on('error')`的函数。

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fld8fyapefj21z20fy0yc.jpg)

通常这样的测试用例并不是特别好写。不过好在我们可以尝试去触发server端的错误，对于本例来说，如果向服务端创建一个todo的时候，没有附上相应的信息（id、status、content），就无法创建相应的todo，会触发错误。

```js

// server/models/todolist.js

const createTodolist = async function (data) {
  await Todolist.create({
    user_id: data.id,
    content: data.content,
    status: data.status
  })
  return true
}
```

上面是server端创建todo的相关函数，下面是针对它的错误进行的测试：

```js
// test/server/todolist.spec.js
// ...
test('Failed to create a todo if not give the params', async () => {
  const response = await request(server)
            .post('/api/todolist')
            .set('Authorization', 'Bearer ' + token) // 不发送创建的参数
  expect(response.status).toBe(500) // 服务端报500错误
})
```

再进行测试，发现之前对于app.js的相关测试都已经是100%了。

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fld8s0m0g5j20xy0lq41t.jpg)

不过`controllers/todolist.js`里还是有未测试到的行数34，以及我们可以看到`% Branch`这列的数字显示的是50而不是100。`Branch`的意思就是分支测试。什么是分支测试呢？简单来说就是你的条件语句测试。比如一个`if...else`语句，如果测试用例只跑过`if`的条件，而没有跑过`else`的条件，那么`Branch`的测试就不完整。让我们来看看是什么条件没有测试到？

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fld8yzgta5j214w0b6di6.jpg)

可以看到是个三元表达式并没有测试完整。（三元表达式也算分支）我们测试了0的情况，但是没有测试非零的情况，所以再写一个非零的情况：

```js
test('Failed to update todolist  if not update the status of todolist', async () => {
  const response = await request(server)
                    .put(`/api/todolist/2/${todoId}/1`) // <- 这里最后一个参数改成了1
                    .set('Authorization', 'Bearer ' + token)
  expect(response.body.success).toBe(false)
})
```

再次跑测试：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fld977icalj20x40lmdj2.jpg)

哈，成功做到了100%测试覆盖率！

### 端口占用和环境变量的引入

虽然做到了100%测试覆盖率，但是有一个问题却是不容忽视的。那就是我们现在测试环境和开发环境下的服务端监听的端口是一致的。意味着你不能在开发环境下测试你的代码。比如你写完一个api之后马上要写一个测试用例的时候，如果测试环境和开发环境的服务端监听的端口一致的话，测试的时候就会因为端口被占用而无法被监听到。

所以我们需要指定一下测试环境下的端口，让它和开发乃至生产环境的端口不一样。我一开始想法很简单，指定一下`NODE_ENV=test`的时候用8888端口，开发环境下用8889端口。在`app.js`里就是这样写：

```js
// ...
let port = process.env.NODE_ENV === 'test' ? 8888 : 8889
// ...
export default app.listen(port, () => {
  console.log(`Koa is listening in ${port}`)
})
```

接下去就遇到了两个问题：

1. 需要解决跨平台env设置
2. 这样设置的话一旦在测试环境下，对于port这句话，`Branch`测试是无法完全通过的——因为始终是在test环境下，无法运行到`port = 8889`那个条件

#### 跨平台env设置

跨平台env主要涉及到windows、linux和macOS。要在三个平台在测试的时候都跑着`NODE_ENV=test`的话，我们需要借助[cross-env](https://github.com/kentcdodds/cross-env)来帮助我们。

```bash
yarn add cross-env -D

#or

npm install cross-env --save-dev
```

然后在`package.json`里修改`test`的命令如下：

```json
"scripts": {
  // ...其他命令
  "test": "cross-env NODE_ENV=test jest --forceExit --runInBand"
  // ...其他命令
},
```

这样就能在后端代码里，通过`process.env.NODE_ENV`这个变量访问到`test`这个值。这样就解决了第一个问题。

#### 端口分离并保证测试覆盖率

目前为止，我们已经能够解决测试环境和开发环境的监听端口一致的问题了。不过却带来了测试覆盖率不全的问题。

为此我找到两种解决办法：

1. 通过istanbul特殊的[`ignore`注释](https://github.com/gotwarlost/istanbul/blob/master/ignoring-code-for-coverage.md)来忽略测试环境下的一些测试分支条件
2. 通过配置环境变量文件，不同环境下采用不同的环境变量文件


第一种方法很简单，在需要忽略的地方，输入`/* istanbul ignore next */`或`/* istanbul ignore <word>[non-word] [optional-docs] */`等语法忽略代码。不过考虑到这是涉及到测试环境和开发环境下的环境变量问题，如果不仅仅是端口问题的话，那么就不如采用第二种方法来得更加优雅。（比如开发环境和测试环境的数据库用户和密码都不一样的话，还是需要写在对应的环境变量的）

此时我们需要另外一个很常用的库[dotenv](https://github.com/motdotla/dotenv)，它能默认读取`.env`文件里的值，让我们的项目可以通过不同的`.env`文件来应对不同的环境要求。

步骤如下：

##### 1. 安装dotenv

```bash
yarn add dotenv

#or

npm install dotenv --save
```

##### 2. 在项目根目录下创建`.env`和`.env.test`两个文件，分别应用于开发环境和测试环境

// .env

```bash
DB_USER=xxxx # 数据库用户
DB_PASSWORD=yyyy # 数据库密码
PORT=8889 # 监听端口
```

// .env.test

```bash
DB_USER=xxxx # 数据库用户
DB_PASSWORD=yyyy # 数据库密码
PORT=8888 # 监听端口
```

##### 3. 创建一个`env.js`文件，用于不同环境下采用不同的环境变量。代码如下：

```js
import * as dotenv from 'dotenv'
let path = process.env.NODE_ENV === 'test' ? '.env.test' : '.env'
dotenv.config({path, silent: true})
```

##### 4. 在app.js开头引入env

```js
import './env'
```

然后把原本那句port的话改成：

```js
let port = process.env.PORT
```

再把数据库连接的用户密码也用环境变量来代替：

```js
// server/config/db.js

import '../../env'
import Sequelize from 'sequelize'

const Todolist = new Sequelize(`mysql://${process.env.DB_USER}:${process.env.DB_PASSWORD}@localhost/todolist`, {
  define: {
    timestamps: false // 取消Sequelzie自动给数据表加入时间戳（createdAt以及updatedAt）
  }
})

```

**不过需要注意的是，.env和.env.js文件都不应该纳入git版本库，因为都是比较重要的内容。**

这样就能实现不同环境下用不同的变量了。慢着！这样不是还没有解决问题吗？`env.js`里的条件还是无法被测试覆盖啊——你肯定有这样的疑问。不用紧张，现在给出解决办法——给Jest指定收集测试覆盖率的范围：

修改`package.json`里`jest`字段如下：

```json
"jest": {
  "verbose": true,
  "coverageDirectory": "coverage",
  "mapCoverage": true,
  "collectCoverage": true,
  "coverageReporters": [
    "lcov",
    "text"
  ],
  "collectCoverageFrom": [ // 指定Jest收集测试覆盖率的范围
    "!env.js", // 排除env.js
    "server/**/*.js",
    "app.js"
  ]
}
```

做完这些工作之后，再跑一次测试，一次通过：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1fld977icalj20x40lmdj2.jpg)

这样我们就完成了后端的api测试。完成了100%测试覆盖率。下面我们可以开始测试Vue的前端项目了。


## Vue前端测试

Vue的前端测试我就要推荐来自官方的[vue-test-utils](https://github.com/vuejs/vue-test-utils)了。当然前端测试大致分成了单元测试（Unit test)和端对端测试(e2e test)，由于端对端的测试对于测试环境的要求比较严苛，而且测试起来比较繁琐，而且官方给出的测试框架是单元测试框架，因此本文对于Vue的前端测试也仅介绍配合官方工具的单元测试。

在Vue的前端测试中我们能够了解到jest的mock、snapshot等特性和用法和vue-test-utils提供的mount、shallow、setData等一系列操作。

### 安装vue-test-utils

根据官网的[介绍](https://vue-test-utils.vuejs.org/en/guides/testing-SFCs-with-jest.html)我们需要安装如下：

```bash
yarn add vue-test-utils vue-jest jest-serializer-vue -D

#or

npm install vue-test-utils vue-jest jest-serializer-vue --save-dev
```

其中，`vue-test-utils`是最关键的测试框架。提供了一系列对于Vue组件的测试操作。（下面会提到）。`vue-jest`用于处理`*.vue`的文件，`jest-serializer-vue`用于快照测试提供快照序列化。

### 配置vue-test-utils以及jest

**1.** 修改`.babelrc`

在`test`的`env`里增加或修改`presets`：

```json
{
  "presets": [
    ["env", { "modules": false }],
    "stage-2"
  ],
  "plugins": [
    "transform-runtime"
  ],
  "comments": false,
  "env": {
    "test": {
      "plugins": ["transform-es2015-modules-commonjs"],
      "presets": [
        ["env", { "targets": { "node": "current" }}] // 增加或修改
      ]
    }
  }
}
```

**2.** 修改`package.json`里的jest配置：

```json
"jest": {
  "verbose": true,
  "moduleFileExtensions": [
    "js"
  ],
  "transform": { // 增加transform转换
    ".*\\.(vue)$": "<rootDir>/node_modules/vue-jest",
    "^.+\\.js$": "<rootDir>/node_modules/babel-jest"
  },
  "coverageDirectory": "coverage",
  "mapCoverage": true,
  "collectCoverage": true,
  "coverageReporters": [
    "lcov",
    "text"
  ],
  "moduleNameMapper": { // 处理webpack alias
    "@/(.*)$": "<rootDir>/src/$1"
  },
  "snapshotSerializers": [ // 配置快照测试
    "<rootDir>/node_modules/jest-serializer-vue"
  ],
  "collectCoverageFrom": [
    "!env.js",
    "server/**/*.js",
    "app.js"
  ]
}
```

### 前端单元测试的一些说明

关于vue-test-utils和Jest的配合测试，我推荐可以查看这个系列的[文章](https://alexjoverm.github.io/2017/08/21/Write-the-first-Vue-js-Component-Unit-Test-in-Jest/)，讲解很清晰。

接着，明确一下前端单元测试都需要测试些什么东西。引用`vue-test-utils`的说法：

> 对于 UI 组件来说，我们不推荐一味追求行级覆盖率，因为它会导致我们过分关注组件的内部实现细节，从而导致琐碎的测试。

> 取而代之的是，我们推荐把测试撰写为断言你的组件的公共接口，并在一个黑盒内部处理它。一个简单的测试用例将会断言一些输入 (用户的交互或 prop 的改变) 提供给某组件之后是否导致预期结果 (渲染结果或触发自定义事件)。

> 比如，对于每次点击按钮都会将计数加一的 Counter 组件来说，其测试用例将会模拟点击并断言渲染结果会加 1。该测试并没有关注 Counter 如何递增数值，而只关注其输入和输出。

> 该提议的好处在于，即便该组件的内部实现已经随时间发生了改变，只要你的组件的公共接口始终保持一致，测试就可以通过。

所以，相对于后端api测试看重测试覆盖率而言，前端的单元测试是不必一味追求测试覆盖率的。（当然你要想达到100%测试覆盖率也是没问题的，只不过如果要达到这样的效果你需要撰写非常多繁琐的测试用例，占用太多时间，得不偿失。）替代地，我们只需要回归测试的本源：给定输入，我只关心输出，不考虑内部如何实现。只要能覆盖到和用户相关的操作，能测试到页面的功能即可。

和之前类似，我们在`test/client`目录下书写我们的测试用例。对于Vue的单元测试来说，我们就是针对`*.vue`文件进行测试了。由于本例里的`app.vue`无实际意义，所以就测试`Login.vue`和`Todolist.vue`即可。

运用`vue-test-utils`如何来进行测试呢？简单来说，我们需要的做的就是用`vue-test-utils`提供的`mount`或者`shallow`方法将组件在后端渲染出来，然后通过一些诸如`setData`，`propsData`、`setMethods`等方法模拟用户的操作或者模拟我们的测试条件，最后再用jest提供的`expect`断言来对预期的结果进行判断。这里的预期就很丰富了。我们可以通过判断事件是否触发、元素是否存在、数据是否正确、方法是否被调用等等来对我们的组件进行比较全面的测试。下面的例子里也会比较完整地介绍它们。

### Login.vue的测试

创建一个`login.spec.js`文件。

首先我们来测试页面里是否有两个输入框和一个登录按钮。根据官方文档，我首先注意到了[shallow rendering](https://vue-test-utils.vuejs.org/en/guides/common-tips.html#shallow-rendering)，它的说明是，对于某个组件而言，只渲染这个组件本身，而不渲染它的子组件，让测试速度提高，也符合单元测试的理念。看着好像很不错的样子，拿过来用。

#### 查找元素测试

```js
import { shallow } from 'vue-test-utils'
import Login from '../../src/components/Login.vue'

let wrapper

beforeEach(() => {
  wrapper = shallow(Login) // 每次测试前确保我们的测试实例都是是干净完整的。返回一个wrapper对象
})

test('Should have two input & one button', () => {
  const inputs = wrapper.findAll('.el-input') // 通过findAll来查找dom或者vue实例
  const loginButton = wrapper.find('.el-button') // 通过find查找元素
  expect(inputs.length).toBe(2) // 应该有两个输入框
  expect(loginButton).toBeTruthy() // 应该有一个登录按钮。 只要断言条件不为空或这false，toBeTruthy就能通过。
})
```

一切看起来很正常。运行测试。结果报错了。报错是`input.length`并不等于2。通过debug断点查看，确实并没有找到元素。

这是怎么回事？哦对，我想起来，形如`el-input`、`el-button`其实也相当于是子组件啊，所以`shallow`并不能将它们渲染出来。在这种情况下，用`shallow`来渲染就不合适了。所以还是需要用`mount`来渲染，它会将页面渲染成它应该有的样子。

```js
import { mount } from 'vue-test-utils'
import Login from '../../src/components/Login.vue'

let wrapper

beforeEach(() => {
  wrapper = mount(Login) // 每次测试前确保我们的测试实例都是是干净完整的。返回一个wrapper对象
})

test('Should have two input & one button', () => {
  const inputs = wrapper.findAll('.el-input') // 通过findAll来查找dom或者vue实例
  const loginButton = wrapper.find('.el-button') // 通过find查找元素
  expect(inputs.length).toBe(2) // 应该有两个输入框
  expect(loginButton).toBeTruthy() // 应该有一个登录按钮。 只要断言条件不为空或这false，toBeTruthy就能通过。
})
```

测试，还是报错！还是没有找到它们。为什么呢？再想想。应该是我们并没有将`element-ui`引入我们的测试里。因为`.el-input`实际上是`element-ui`的一个组件，如果没有引入它，vue自然无法将一个`el-input`渲染成`<div class="el-input"><input></div>`这样的形式。想通了就好说了，把它引进来。因为我们的项目里在`webpack`环境下是有一个`main.js`作为入口文件的，在测试里可没有这个东西。所以Vue自然也不知道你测试里用到了什么依赖，需要我们单独引入：

```js
import Vue from 'vue'
import elementUI from 'element-ui'
import { mount } from 'vue-test-utils'
import Login from '../../src/components/Login.vue'

Vue.use(elementUI)

// ...
```

再次运行测试，通过！

#### 快照测试

接下来，使用Jest内置的一个特别棒的特性：快照（snapshot）。它能够将某个状态下的html结构以一个快照文件的形式存储下来，以后每次运行快照测试的时候如果发现跟之前的快照测试的结果不一致，测试就无法通过。

当然如果是以后页面确实需要发生改变，快照需要更新，那么只需要在执行jest的时候增加一个`-u`的参数，就能实现快照的更新。

说完了原理来实践一下。对于登录页，实际上我们只需要确保html结构没问题那么所有必要的元素自然就存在。因此快照测试写起来特别方便：

```js
test('Should have the expected html structure', () => {
  expect(wrapper.element).toMatchSnapshot() // 调用toMatchSnapshot来比对快照
})
```

如果是第一次进行快照测试，那么它会在你的测试文件所在目录下新建一个`__snapshots__`的目录存放快照文件。上面的测试就生成了一个`login.spec.js.snap`的文件，如下：

```html
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`Should have the expected html structure 1`] = `
<div
  class="el-row content"
>
  <div
    class="el-col el-col-24 el-col-xs-24 el-col-sm-6 el-col-sm-offset-9"
  >
    <span
      class="title"
    >
      
     欢迎登录
    
    </span>
     
    <div
      class="el-row"
    >
      <div
        class="el-input"
      >
        <!---->
        <!---->
        <input
          autocomplete="off"
          class="el-input__inner"
          placeholder="账号"
          type="text"
        />
        <!---->
        <!---->
      </div>
       
      <div
        class="el-input"
      >
        <!---->
        <!---->
        <input
          autocomplete="off"
          class="el-input__inner"
          placeholder="密码"
          type="password"
        />
        <!---->
        <!---->
      </div>
       
      <button
        class="el-button el-button--primary"
        type="button"
      >
        <!---->
        <!---->
        <span>
          登录
        </span>
      </button>
    </div>
  </div>
</div>
`;
```

可以看到它将整个html结构以快照的形式保存下来了。快照测试能确保我们的前端页面结构的完整性和稳定性。

#### methods测试

很多时候我们需要测试在某些情况下，Vue中的一些methods能否被触发。比如本例里的，我们点击登录按钮应对要触发`loginToDo`这个方法。于是就涉及到了`methods`的测试，这个时候`vue-test-utils`提供的`setMethods`这个方法就很有用了。我们可以通过设置（覆盖）`loginToDo`这个方法，来查看它是否被触发了。

> 注意，一旦setMethods了某个方法，那么在某个test()内部，这个方法原本的作用将完全被你的新function覆盖。包括这个Vue实例里其他methods通过`this.xxx()`方式调用也一样。

```js
test('loginToDo should be called after clicking the button', () => {
  const stub = jest.fn() // 伪造一个jest的mock funciton
  wrapper.setMethods({ loginToDo: stub }) // setMethods将loginToDo这个方法覆写
  wrapper.find('.el-button').trigger('click') // 对button触发一个click事件
  expect(stub).toBeCalled() // 查看loginToDo是否被调用
})
```

注意到这里我们用到了`jest.fn`这个方法，这个在下节会详细说明。此处你只需要明白这个是jest提供的，可以用来检测是否被调用的方法。

#### mock方法测试

接下去就是对登录这个功能的测试了。由于我们之前把Koa的后端api进行了测试，所以我们在前端测试中，可以默认后端的api接口都是返回正确的结果的。（这也是我们先进行了Koa端测试的原因，保证了后端api的健壮性回到前端测试的时候就能很轻松）

虽然道理是说得通的，但是我们如何来默认、或者说“伪造”我们的api请求，以及返回的数据呢？这个时候就需要用上Jest一个非常有用的功能`mock`了。可以说`mock`这个词对很多做前端的朋友来说，不是很陌生。在没有后端，或者后端功能还未完成的时候，我们可以通过api的mock来实现伪造请求和数据。

Jest的mock也是同理，不过它更厉害的一点是，它能伪造库。比如我们接下去要用的HTTP请求库`axios`。对于我们的页面来说，登录只需要发送post请求，判断返回的`success`是否是`true`即可。我们先来mock一下`axios`以及它的`post`请求。

```js
jest.mock('axios', () => ({
  post: jest.fn(() => Promise.resolve({
    data: {
      success: false,
      info: '用户不存在！'
    }
  }))
}))
```

然后我们可以把axios引入我们的项目了：

```js
import Vue from 'vue'
import elementUI from 'element-ui'
import { mount } from 'vue-test-utils'
import Login from '../../src/components/Login.vue'
import axios from 'axios'

Vue.use(elementUI)

Vue.prototype.$http = axios

jest.mock(....)
```

等会，你肯定会提出疑问，`jest.mock()`方法写在了`import axios from 'axios'`下面，那么不就意味着`axios`是从`node_modules`里引入的吗？其实不是的，`jest.mock()`会实现函数提升，也就是实际上上面的代码其实和下面的是一样的：

```js
jest.mock(....)
import Vue from 'vue'
import elementUI from 'element-ui'
import { mount } from 'vue-test-utils'
import Login from '../../src/components/Login.vue'
import axios from 'axios' // 这里的axios是来自jest.mock()里的axios

Vue.use(elementUI)

Vue.prototype.$http = axios
```

看起来甚至有些`var`的变量提升的味道。

不过这样的好处是很明显的，我们可以在不破坏`eslint`的规则的情况下采用第一种的写法而达到一样的目的。

然后你还会注意到我们用到了`jest.fn()`的方法，它是jest的mock方法里很重要的一部分。它本身是一个`mock function`。通过它能够实现方法调用的追踪以及后面会说到的能够实现创建复杂行为的模拟功能。

继续我们没写完的测试：

```js
test('Failed to login if not typing the correct password', async () => {
  wrapper.setData({
    account: 'molunerfinn',
    password: '1234'
  }) // 模拟用户输入数据
  const result = await wrapper.vm.loginToDo() // 模拟异步请求的效果
  expect(result.data.success).toBe(false) // 期望返回的数据里success是false
  expect(result.data.info).toBe('密码错误！')
})
```

我们通过`setData`来模拟用户在两个input框内输入了数据。然后通过`wrapper.vm.loginToDo()`来显式调用`loginTodo`的方法。由于我们返回的是一个`Promise`对象，所以可以用`async await`将resolve里的数据拿出来。然后测试是否和预期相符。我们这次是测试了输入错误的情况，测试通过，没有问题。那如果我接下去要再测试用户密码都通过的测试怎么办？我们`mock`的`axios`的`post`方法只有一个，难不成还能一个方法输出多种结果？下一节来详细说明这个问题。

#### 创建复杂行为测试

回顾一下我们的mock写法：

```js
jest.mock('axios', () => ({
  post: jest.fn(() => Promise.resolve({
    data: {
      success: false,
      info: '用户不存在！'
    }
  }))
}))
```

可以看到，采用这种写法的话，post请求始终只能返回一种结果。如何做到既能`mock`这个`post`方法又能实现多种结果测试？接下去就要用到Jest另一个杀手锏的方法：[mockImplementationOnce](https://facebook.github.io/jest/docs/en/mock-functions.html#mock-implementations)。官方的示例如下：

```js
const myMockFn = jest.fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
// > 'first call', 'second call', 'default', 'default'
```

4次调用同一个方法却能给出不同的运行结果。这正是我们想要的。

于是在我们测试登录成功这个方法的时候我们需要改写一下我们对`axios`的mock方法：

```js
jest.mock('axios', () => ({
  post: jest.fn()
        .mockImplementationOnce(() => Promise.resolve({
          data: {
            success: false,
            info: '用户不存在！'
          }
        }))
        .mockImplementationOnce(() => Promise.resolve({
          data: {
            success: true,
            token: 'xxx' // 随意返回一个token
          }
        }))
}))
```

然后开始写我们的测试：

```js
test('Succeeded to login if typing the correct account & password', async () => {
  wrapper.setData({
    account: 'molunerfinn',
    password: '123'
  })
  const result = await wrapper.vm.loginToDo()
  expect(result.data.success).toBe(true)
})
```

就在我认为跟之前的测试没有什么两样的时候，报错传来了。先来看看当`success`为true的时候，`loginToDo`在做什么：

```js
if (res.data.success) { // 如果成功
  sessionStorage.setItem('demo-token', res.data.token) // 用sessionStorage把token存下来
  this.$message({ // 登录成功，显示提示语
    type: 'success',
    message: '登录成功！'
  })
  this.$router.push('/todolist') // 进入todolist页面，登录成功
}
```

很快我就看到了错误所在：我们的测试环境里并没有`sessionStorage`这个原本应该在浏览器端的东西。以及我们并没有使用`vue-router`，所以就无法执行`this.$router.push()`这个方法。

关于前者，很容易找到[问题](https://stackoverflow.com/questions/30792076/mocking-sessionstorage-when-using-jestjs)的[解决办法](https://github.com/letsrock-today/mock-local-storage)。

首先安装一下`mock-local-storage`这个库（也包括了sessionStorage）

```bash
yarn add mock-local-storage -D

#or

npm install mock-local-storage --save-dev
```

然后配置一下`package.json`里的`jest`参数：

```json
"jest": {
  // ...
  "setupTestFrameworkScriptFile": "mock-local-storage"
}
```

对于后者，阅读过官方的[建议](https://vue-test-utils.vuejs.org/en/guides/common-tips.html#dealing-with-routing)，我们不应该引入`vue-router`，这样会破坏我们的单元测试。相应的，我们可以mock它。不过这次是用`vue-test-utils`自带的`mocks`特性了：

```js
const $router = { // 声明一个$router对象
  push: jest.fn()
}

beforeEach(() => {
  wrapper = mount(Login, {
    mocks: {
      $router // 在beforeEach钩子里挂载进mount的mocks里。
    }
  })
})
```

通过这个方式，会把`$router`这个对象挂载到实例的`prototype`上，就能实现在组件内部通过`this.$router.push()`的方式来调用了。

上述两个问题解决之后，我们的测试也顺利通过了：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1flj15nzsg0j215c06aq4c.jpg)

接下去开始测试`Todolist.vue`这个组件了。

### Todolist.vue的测试

#### 键盘事件测试以及隐式事件触发

类似的我们在`test/client`目录下创建一个叫做`todolist.spec.js`的文件。

先把上例中的一些环境先预置进来：

```js
import Vue from 'vue'
import elementUI from 'element-ui'
import { mount } from 'vue-test-utils'
import Todolist from '../../src/components/Todolist.vue'
import axios from 'axios'

Vue.use(elementUI)

jest.mock(...) // 后续补充

Vue.prototype.$http = axios

let wrapper

beforeEach(() => {
  wrapper = mount(Todolist)
  wrapper.setData({
    name: 'Molunerfinn', // 预置数据
    id: 2
  })
})
```

先来个简单的，测试数据是否正确：

```js
// test 1
test('Should get the right username & id', () => {
  expect(wrapper.vm.name).toBe('Molunerfinn')
  expect(wrapper.vm.id).toBe(2)
})
```
不过需要注意的是，`todolist`这个页面在`created`阶段就会触发`getUserInfo`和`getTodolist`这两个方法，而我们的wrapper是相当于在`mounted`阶段之后的。所以在我们拿到wrapper的时候，`created`、`mounted`等生命周期的钩子其实已经运行了。本例里`getUserInfo`是从`sessionStorage`里取值，不涉及ajax请求。但是`getTodolist`涉及请求，因此需要在jest.mock方法里为其配置一下，否则将会报错：

```js
jest.mock('axios', () => ({
  get: jest.fn()
        // for test 1
        .mockImplementationOnce(() => Promise.resolve({
          status: 200,
          data: {
            result: []
          }
        }))
}))
```

上面说到的`getTodolist`和`getUserInfo`就是在测试中需要注意的隐式事件，它们并不受你测试的控制就在组件里触发了。

接下来开始进行键盘事件测试。其实跟鼠标事件类似，键盘事件的触发也是以事件名来命名的。不过对于一些常见的事件，`vue-test-utils`里给出了一些别名比如：

`enter, tab, delete, esc, space, up, down, left, right`。你在书写测试的时候可以直接这样：

```js
const input = wrapper.find('.el-input')
input.trigger('keyup.enter')
```

当然如果你需要指定某个键也是可以的，只需要提供keyCode就行：

```js
const input = wrapper.find('.el-input')
input.trigger('keyup'， {
  which: 13 // enter的keyCode为13
})
```

于是我们把这个测试完善一下，这个测试是测试当我在输入框激活的情况下按下回车键能否触发`addTodos`这个事件：

```js
test('Should trigger addTodos when typing the enter key', () => {
  const stub = jest.fn()
  wrapper.setMethods({
    addTodos: stub
  })
  const input = wrapper.find('.el-input')
  input.trigger('keyup.enter')
  expect(stub).toBeCalled()
})
```

没有问题，一次通过。

注意到我们在实际开发时，在组件上调用原生事件是需要加`.native`修饰符的：

```html
<el-input placeholder="请输入待办事项" v-model="todos" @keyup.enter.native="addTodos"></el-input>
```

但是在`vue-test-utils`里你是可以直接通过原生的`keyup.enger`来触发的。

#### wrapper.update()的使用

很多时候我们要跟异步打交道。尤其是异步取值，异步赋值，页面异步更新。而对于使用Vue来做的实际开发来说，异步的情况简直太多了。

还记得`nextTick`么？很多时候，我们要获取一个变更的数据结果，不能直接通过`this.xxx`获取，相应的我们需要在`this.$nextTick()`里获取。在测试里我们也会遇到很多需要异步获取的情况，但是我们不需要`nextTick`这个办法，相应的我们可以通过`async await`配合`wrapper.update()`来实现组件更新。例如下面这个测试添加todo成功的例子：

```js
test('Should add a todo if handle in the right way', async () => {
  wrapper.setData({
    todos: 'Test',
    stauts: '0',
    id: 1
  })

  await wrapper.vm.addTodos()
  await wrapper.update()
  expect(wrapper.vm.list).toEqual([
    {
      status: '0',
      content: 'Test',
      id: 1
    }
  ])
})
```

在本例中，从进页面到添加一个todo并显示出来需要如下步骤：

1. getUserInfo -> getTodolist
2. 输入todo并敲击回车
3. addTodos -> getTodolist
4. 显示添加的todo

可以看到总共有3个ajax请求。其中第一步不在我们test()的范围内，2、3、4都是我们能控制的。而addTodos和getTodolist这两个ajax请求带来的就是异步的操作。虽然我们mock方法，但是本质上是返回了Promise对象。所以还是需要用`await`来等待。

> 注意你在jest.mock()里要加上相应的mockImplementationOnce的get和post请求。

所以第一步`await wrapper.vm.addTodos()`就是等待`addTodos()`的返回。
第二步`await wrapper.update()`实际是在等待`getTodolist`的返回。

缺一不可。两步等待之后我们就可以通过断言数据`list`的方式测试我们是否拿到了返回的todo的信息。

接下去的就是对todo的一些增删改查的操作，采用的测试方法已经和前文所述相差无几，不再赘述。至此所有的独立测试用例的说明就说完了。看看这测试通过的成就感：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1flj2hpjeprj213606udhc.jpg)

不过在测试中我还有关于调试的一些经验想分享一下，配合调试能更好的判断我们的测试的时候发生的不可预知的问题所在。

### 用VSCode来调试测试

由于我自己是使用VSCode来做的开发和调试，所以一些用其他IDE或者编辑器的朋友们可能会有所失望。不过没关系，可以考虑加入VSCode阵营嘛！

本文撰写的时候采用的nodejs版本为`8.9.0`，VSCode版本为`1.18.0`，所以所有的debug测试的配置仅保证适用于目前的环境。其他环境的可能需要自行测试一下，不再多说。

关于jest的调试的配置如下：（注意配置路径为VScode关于本项目的`.vscode/launch.json`）

```json
{
  // Use IntelliSense to learn about possible Node.js debug attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Jest",
      "type": "node",
      "request": "launch",
      "program": "${workspaceRoot}/node_modules/jest-cli/bin/jest.js",
      "stopOnEntry": false,
      "args": [
        "--runInBand",
        "--forceExit"
      ],
      "cwd": "${workspaceRoot}",
      "preLaunchTask": null,
      "runtimeExecutable": null,
      "runtimeArgs": [
        "--nolazy"
      ],
      "env": {
        "NODE_ENV": "test"
      },
      "console": "integratedTerminal",
      "sourceMaps": true
    }
  ]
}
```

配置完上面的配置之后，你可以在`DEBUG`面板里（不要跟我说你不知道什么是DEBUG面板~）找到名为`Debug Jest`的选项：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1flj2s0qhxkj20og09kq3p.jpg)

然后你可以在你的测试文件里打断点了：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1flj2uakvi4j21bk09oq5s.jpg)

然后运行debug模式，按那个绿色启动按钮，就能进入DEBUG模式，当运行到断点处就会停下：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1flj2y2hpeaj21fa09u41u.jpg)

于是你可以在左侧面板的`Local`和`Closure`里找到当前作用域下你所需要的变量值、变量类型等等。充分运用VSCode的debug模式，开发的时候查错和调试的效率都会大大加大。

## 总结

本文用了很大的篇幅描述了如何搭建一个Jest测试环境，并在测试过程中不断完善我们的测试环境。讲述了Koa后端测试的方法和测试覆盖率的提高，讲述了Vue前端单元测试环境的搭建以及许多相应的测试实例，以及在测试过程中不停地遇到问题并解决问题。能够看到此处的都不是一般有耐心的人，为你们鼓掌~也希望你们通过这篇文章能过对本文在开头提出的几个重点在心中有所体会和感悟：

> 可以了解到测试的意义，[Jest](https://facebook.github.io/jest/)测试框架的搭建，前后端测试的异同点，如何写测试用例，如何查看测试结果并提升我们的测试覆盖率，100%测试覆盖率是否是必须，以及在搭建测试环境、以及测试本身过程中遇到的各种疑难杂症。

本文所有的测试用例以及整体项目实例你都可以在我的[vue-koa-demo](https://github.com/Molunerfinn/vue-koa-demo)的github项目中找到源代码。如果你喜欢我的文章以及项目，欢迎点个star~如果你对我的文章和项目有任何建议或者意见，欢迎在文末评论或者在本项目的[issues](https://github.com/Molunerfinn/vue-koa-demo/issues)跟我探讨！

## 参考链接

> Koa相关

[Supertest搭配koa报错](https://segmentfault.com/q/1010000006906863)

[测试完自动退出](http://facebook.github.io/jest/docs/en/cli.html#forceexit)

[Async testing Koa with Jest](https://hackernoon.com/async-testing-koa-with-jest-1b6e84521b71)

[How to use Jest to test Express middleware or a funciton which consumes a callback?](http://www.albertgao.xyz/2017/06/10/how-to-use-jest-to-test-express-middleware-or-a-function-which-consumes-a-callback/)

[A clear and concise introduction to testing Koa with Jest and Supertest](https://www.valentinog.com/blog/testing-api-koa-jest/)

[Debug jest with vscode](https://github.com/Microsoft/vscode/issues/28007)

[Test port question](https://stackoverflow.com/questions/12236890/run-mocha-tests-in-test-environment)
[Coverage bug](https://github.com/facebook/jest/issues/4777)

[Eaddrinuse bug](https://stackoverflow.com/questions/41733634/eaddrinuse-127-0-0-15858-during-jest-test-debugging)

[Istanbul ignore](https://github.com/gotwarlost/istanbul/blob/master/ignoring-code-for-coverage.md)

> Vue相关

[vue-test-utils](https://vue-test-utils.vuejs.org/en/)

[Test Methods and Mock Dependencies in Vue.js with Jest](https://alexjoverm.github.io/2017/09/25/Test-Methods-and-Mock-Dependencies-in-Vue-js-with-Jest/)

[Storage problem](https://stackoverflow.com/questions/30792076/mocking-sessionstorage-when-using-jestjs)
