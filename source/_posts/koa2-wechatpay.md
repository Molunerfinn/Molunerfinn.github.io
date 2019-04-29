---
title: 基于Koa2开发微信二维码扫码支付相关流程
tags: 
  - 前端
  - Nodejs
  - Koa
categories:
  - Web
  - 开发
  - Nodejs
date: 2018-05-15 13:50:00
---

前段时间在开发一个功能，要求是通过微信二维码进行扫码支付。这个情景我们屡见不鲜了，各种电子商城、线下的自动贩卖机等等都会有这个功能。平时只是使用者，如今变为开发者，也是有不小的坑。所以特此写一篇博客记录一下。

> **注**： 要开发微信二维码支付，你必须要有相应的商户号的权限，否则你是无法开发的。若无相应权限，本文不推荐阅读。

<!-- more -->

## 两种模式

打开微信支付的文档，我们可以看到两种支付模式：[模式一](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_4)和[模式二](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_5)。这二者的流程图微信的文档里都给出了（不过说实话画得真的有点丑）。

文档里指出了二者的区别：

> 模式一开发前，商户必须在公众平台后台设置支付回调URL。URL实现的功能：接收用户扫码后微信支付系统回调的productid和openid。

> 模式二与模式一相比，流程更为简单，不依赖设置的回调支付URL。商户后台系统先调用微信支付的统一下单接口，微信后台系统返回链接参数code_url，商户后台系统将code_url值生成二维码图片，用户使用微信客户端扫码后发起支付。注意：code_url有效期为2小时，过期后扫码不能再发起支付。

模式一是我们平时在网购的时候比较常见的，会弹出一个专门的页面用于扫码支付，然后支付成功后这个页面会再次跳转回回调页面，通知你支付成功。第二种的话想对少一些，不过第二种开发起来相对简单点。**本文主要介绍模式二的开发**。

## 搭建Koa2的简单开发环境

快速搭建Koa2的开发环境我推荐可以使用[koa-generator](https://github.com/17koa/koa-generator)。脚手架能帮我们省去Koa项目一开始的一些基本中间件的书写步骤。（如果你想学习Koa最好自己搭建一个。如果你已经会Koa了就可以使用一些快速脚手架了。）

首先全局安装`koa-generator`：

```bash
npm install -g koa-generator

#or

yarn global add koa-generator
```

然后找一个目录用来存放Koa项目，我们打算给这个项目取个名字叫做`koa-wechatpay`，然后就可以输入`koa2 koa-wechatpay`。然后脚手架会自动创建相应文件夹`koa-wechatpay`，并生成基本骨架。进入这个文件夹，安装相应的插件。输入：

```bash
npm install

#or

yarn
```

接着你可以输入`npm start` 或者 `yarn start`来运行项目（默认监听在3000端口）。

如果不出意外，你的项目跑起来了，然后我们用postman测试一下：

> 这条路由是在`routes/index.js`里。

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/8700af19ly1frc14ddfn9j21iq0r2n0p.jpg)

如果你看到了

```json
{
  "title": "koa2 json"
}
```

就说明没问题。（如果有问题，检查一下是不是端口被占用了等等。）

接下来在`routes`文件夹里我们新建一个`wechatpay.js`的文件用来书写我们的流程。

## 签名

跟微信的服务器交流很关键的一环是签名必须正确，如果签名不正确，那么一切都白搭。

首先我们需要去公众号的后台获取我们所需要的如下相应的id或者key的信息。其中`notify_url`和`server_ip`是用于当我们支付成功后，微信会主动往这个url`post`支付成功的信息。

签名算法如下：https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=4_3

为了签名正确，我们需要安装一下`md5`。

```bash
npm install md5 --save

#or

yarn add md5
```

```js
const md5 = require('md5')
const appid = 'xxx'
const mch_id = 'yyy'
const mch_api_key = 'zzz'
const notify_url = 'http://xxx/api/notify' // 服务端可访问的域名和接口
const server_ip = 'xx.xx.xx.xx' // 服务端的ip地址
const trade_type = 'NATIVE' // NATIVE对应的是二维码扫码支付
let body = 'XXX的充值支付' // 用于显示在支付界面的提示词
```

然后开始写签名函数：

```js
const signString = (fee, ip, nonce) => {
  let tempString = `appid=${appid}&body=${body}&mch_id=${mch_id}&nonce_str=${nonce}&notify_url=${notify_url}&out_trade_no=${nonce}&spbill_create_ip=${ip}&total_fee=${fee}&trade_type=${trade_type}&key=${mch_api_key}`
  return md5(tempString).toUpperCase()
}
```

其中`fee`是要充值的费用，以分为单位。比如要充值1块钱，`fee`就是100。ip是个比较随意的选项，只要符合规则的ip经过测试都是可以的，下文里我用的是`server_ip`。`nonce`就是微信要求的不重复的32位以内的字符串，通常可以使用订单号等唯一标识的字符串。

由于跟微信的服务器交流都是用xml来交流，所以现在我们要手动组装一下post请求的`xml`:

```js
const xmlBody = (fee, nonce_str) => {
  const xml = `
    <xml>
    <appid>${appid}</appid>
    <body>${body}</body>
    <mch_id>${mch_id}</mch_id>
    <nonce_str>${nonce_str}</nonce_str>
    <notify_url>${notify_url}</notify_url>
    <out_trade_no>${nonce_str}</out_trade_no>
    <total_fee>${fee}</total_fee>
    <spbill_create_ip>${server_ip}</spbill_create_ip>
    <trade_type>NATIVE</trade_type>
    <sign>${signString(fee, server_ip, nonce_str)}</sign>
    </xml>
  `
  return {
    xml,
    out_trade_no: nonce_str
  }
}
```

> 如果你怕自己的签名的`xml`串有问题，可以提前在微信提供的[签名校验工具](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=20_1)里先校验一遍，看看是否能通过。

## 发送请求

因为需要跟微信服务端发请求，所以我选择了`axios`这个在浏览器端和node端都能发起ajax请求的库。

安装过程不再赘述。继续在`wechatpay.js`写发请求的逻辑。

由于微信给我们返回的也将是一个xml格式的字符串。所以我们需要预先写好解析函数，将xml解析成js对象。为此你可以安装一个[xml2js](https://github.com/Leonidas-from-XIV/node-xml2js)。安装过程跟上面的类似，不再赘述。

微信会给我们返回一个诸如下面格式的`xml`字符串：

```xml
<xml><return_code><![CDATA[SUCCESS]]></return_code>
<return_msg><![CDATA[OK]]></return_msg>
<appid><![CDATA[wx742xxxxxxxxxxxxx]]></appid>
<mch_id><![CDATA[14899xxxxx]]></mch_id>
<nonce_str><![CDATA[R69QXXXXXXXX6O]]></nonce_str>
<sign><![CDATA[79F0891XXXXXX189507A184XXXXXXXXX]]></sign>
<result_code><![CDATA[SUCCESS]]></result_code>
<prepay_id><![CDATA[wx152316xxxxxxxxxxxxxxxxxxxxxxxxxxx]]></prepay_id>
<trade_type><![CDATA[NATIVE]]></trade_type>
<code_url><![CDATA[weixin://wxpay/xxxurl?pr=dQNakHH]]></code_url>
</xml>
```

我们的目标是转为如下的js对象，好让我们用js来操作数据：

```js
{
  return_code: 'SUCCESS', // SUCCESS 或者 FAIL
  return_msg: 'OK',
  appid: 'wx742xxxxxxxxxxxxx',
  mch_id: '14899xxxxx',
  nonce_str: 'R69QXXXXXXXX6O',
  sign: '79F0891XXXXXX189507A184XXXXXXXXX',
  result_code: 'SUCCESS',
  prepay_id: 'wx152316xxxxxxxxxxxxxxxxxxxxxxxxxxx',
  trade_type: 'NATIVE',
  code_url: 'weixin://wxpay/xxxurl?pr=dQNakHH' // 用于生成支付二维码的链接
}
```

于是我们写一个函数，调用`xml2js`来解析xml：

```js
// 将XML转为JS对象
const parseXML = (xml) => {
  return new Promise((res, rej) => {
    xml2js.parseString(xml, {trim: true, explicitArray: false}, (err, json) => {
      if (err) {
        rej(err)
      } else {
        res(json.xml)
      }
    })
  })
}
```

上面的代码返回了一个`Promise`对象，因为`xml2js`的操作是在回调函数里返回的结果，所以为了配合Koa2的`async`、`await`，我们可以将其封装成一个`Promise`对象，将解析完的结果通过`resolve`返回回去。这样就能用`await`来取数据了：

```js
const axios = require('axios')
const url = 'https://api.mch.weixin.qq.com/pay/unifiedorder' // 微信服务端地址
const pay = async (ctx) => {
  const form = ctx.request.body // 通过前端传来的数据

  const orderNo = 'XXXXXXXXXXXXXXXX' // 不重复的订单号
  const fee = form.fee // 通过前端传来的费用值

  const data = xmlBody(fee, orderNo) // fee是费用，orderNo是订单号（唯一）
  const res = await axios.post(url, {
    data: data.xml
  }).then(async res => {
    const resJson = await parseXML(res.data)
    return resJson // 拿到返回的数据
  }).catch(err => {
    console.log(err)
  })
  if (res.return_code === 'SUCCESS') { // 如果返回的
    return ctx.body = {
      success: true,
      message: '请求成功',
      code_url: res.code_url, // code_url就是用于生成支付二维码的链接
      order_no: orderNo // 订单号
    }
  }
  ctx.body = {
    success: false,
    message: '请求失败'
  }
}

router.post('/api/pay', pay)

module.exports = router
```

然后我们要将这个router挂载到根目录的`app.js`里去。

找到之前默认的两个路由，一个`index`，一个`user`：

```js
const index = require('./routes/index')
const users = require('./routes/users')
const wechatpay = require('./routes/wechatpay') // 加在这里
```

然后到页面底下挂载这个路由：

```js
// routes
app.use(index.routes(), index.allowedMethods())
app.use(users.routes(), users.allowedMethods())
app.use(wechatpay.routes(), users.allowedMethods()) // 加在这里
```

于是你就可以通过发送`/api/pay`来请求二维码数据啦。（如果有跨域需要自己考虑解决跨域方案，可以跟Koa放在同域里，也可以开一层proxy来转发，也可以开CORS头等等）

**注意**， 本例里是用前端来生成二维码，其实也可以通过后端生成二维码，然后再返回给前端。不过为了简易演示，本例采用前端通过获取`code_url`后，在前端生成二维码。

## 展示支付二维码

前端我用的是`Vue`，当然你可以选择你喜欢的前端框架。这里关注点在于通过拿到刚才后端传过来的`code_url`来生成二维码。

在前端，我使用的是[@xkeshi/vue-qrcode](https://github.com/xkeshi/vue-qrcode)这个库来生成二维码。它调用特别简单：

```js
import VueQrcode from '@xkeshi/vue-qrcode'
export default {
  components: {
    VueQrcode
  },
  // ...其他代码
}
```
然后就可以在前端里用`<vue-qrcode>`的组件来生成二维码了：

```html
<vue-qrcode :value="codeUrl" :options="{ size: 200 }">
```

放到Dialog里就是这样的效果：

> 文本是我自己添加的

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/wechat-pay.png)

## 付款成功自动刷新页面

有两种将支付成功写入数据库的办法。

一种是在打开了扫码对话框后，不停向微信服务端轮询支付结果，如果支付成功，那么就向后端发起请求，告诉后端支付成功，让后端写入数据库。

一种是后端一直开着接口，等微信主动给后端的`notify_url`发起post请求，告诉后端支付结果，让后端写入数据库。然后此时前端向后端轮询的时候应该是去数据库取轮询该订单的支付结果，如果支付成功就关闭Dialog。

第一种比较简单但是不安全：试想万一用户支付成功的同时关闭了页面，或者用户支付成功了，但是网络有问题导致前端没法往后端发支付成功的结果，那么后端就一直没办法写入支付成功的数据。

第二种虽然麻烦，但是保证了安全。所有的支付结果都必须等微信主动向后端通知，后端存完数据库后再返回给前端消息。这样哪怕用户支付成功的同时关闭了页面，下次再打开的时候，由于数据库已经写入了，所以拿到的也是支付成功的结果。

所以`付款成功自动刷新页面`这个部分我们分为两个部分来说：

### 前端部分

Vue的data部分
```js
data: {
  payStatus: false, // 未支付成功
  retryCount: 0, // 轮询次数，从0-200
  orderNo: 'xxx', // 从后端传来的order_no
  codeUrl: 'xxx' // 从后端传来的code_url
}
```
在methods里写一个查询订单信息的方法：
```js

// ...

handleCheckBill () {
  return setTimeout(() => {
    if (!this.payStatus && this.retryCount < 120) {
      this.retryCount += 1
      axios.post('/api/check-bill', { // 向后端请求订单支付信息
        orderNo: this.orderNo
      })
        .then(res => {
          if (res.data.success) {
            this.payStatus = true
            location.reload() // 偷懒就用reload重新刷新页面
          } else {
            this.handleCheckBill()
          }
        }).catch(err => {
          console.log(err)
        })
    } else {
      location.reload()
    }
  }, 1000)
}
```

在打开二维码Dialog的时候，这个方法就启用了。然后就开始轮询。我订了一个时间，200s后如果还是没有付款信息也自动刷新页面。实际上你可以自己根据项目的需要来定义这个时间。

### 后端部分

前端到后端只有一个接口，但是后端有两个接口。一个是用来接收微信的推送，一个是用来接收前端的查询请求。

先来写最关键的微信的推送请求处理。由于我们接收微信的请求是在Koa的路由里，并且是以流的形式传输的。需要让Koa支持解析xml格式的body，所以需要安装一个[rawbody](https://github.com/stream-utils/raw-body)来获取xml格式的body。


```js
// 处理微信支付回传notify
// 如果收到消息要跟微信回传是否接收到
const handleNotify = async (ctx) => {
  const xml = await rawbody(ctx.req, {
    length: ctx.request.length,
    limit: '1mb',
    encoding: ctx.request.charset || 'utf-8'
  })

  const res = await parseXML(xml) // 解析xml

  if (res.return_code === 'SUCCESS') {
    if (res.result_code === 'SUCCESS') { // 如果都为SUCCESS代表支付成功
      // ... 这里是写入数据库的相关操作

      // 开始回传微信
      ctx.type = 'application/xml' // 指定发送的请求类型是xml
      // 回传微信，告诉已经收到
      return ctx.body = `<xml>
        <return_code><![CDATA[SUCCESS]]></return_code>
        <return_msg><![CDATA[OK]]></return_msg>
      </xml>
      `
    }
  }

  // 如果支付失败，也回传微信
  ctx.status = 400
  ctx.type = 'application/xml'
  ctx.body = `<xml>
    <return_code><![CDATA[FAIL]]></return_code>
    <return_msg><![CDATA[OK]]></return_msg>
  </xml>
  `
}

router.post('/api/notify', handleNotify)
```

这里的坑就是Koa处理微信回传的xml。如果不知道是以`raw-body`的形式回传的，会调试半天。。

接下来这个就是比较简单的给前端回传的了。

```js
const checkBill = async (ctx) => {
  const form = ctx.request.body
  const orderNo = form.orderNo
  const result = await 数据库操作

  if (result) { // 如果订单支付成功
    return ctx.body = {
      success: true
    }
  }

  ctx.status = 400
  ctx.body = {
    success: false
  }
}

router.post('/api/check-bill', checkBill)
```

## 总结

至此，一整个基于Koa2的微信二维码支付流程就简单演示完了，由于不是公开的项目，所以没有实际的GitHub仓库。不过基本上关键的代码我都已经注释出来啦。我参考了不少人的实现，曾考虑过用一些比如`wechatpay`的npm库，不过最终还是自己解决了。这里面感谢很多前人的分享，也希望我这篇文章能给你一些帮助。

## 参考文章

微信支付文章

https://www.itbaby.me/blog/59e21af45d21b31fcd4e02c6

https://juejin.im/post/5a8e84faf265da4e7e10c92f

返回接口

http://webcache.googleusercontent.com/search?q=cache:iFC0HZuFB1gJ:jeffdeng.me/wx/2017/03/13/wx-platform-conect.html+&cd=4&hl=zh-CN&ct=clnk&gl=us

XML流处理

https://blog.csdn.net/yxz1025/article/details/52313221

https://juejin.im/post/5a6c558ef265da3e4b77030f
