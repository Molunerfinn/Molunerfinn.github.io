---
title: PicGo 的签名与公证
tags: 
  - Electron
  - PicGo
categories:
  - Web
  - 开发
date: 2025-12-25 00:00
---

一直以来，PicGo 并没有做 macOS 的签名和公证，因为一直没注册苹果开发者账号（其实学生时代需要每年出 $99 确实有点贵），所以就会导致用户下载了 PicGo 之后，会遇到这个问题：

![App 损坏提示](https://pics.molunerfinn.com/blog/20251225104315062.png)

这个其实也有办法绕过去。不过近几年的 macOS 更新之后，对于应用签名是越来越严格，以前的一行命令还不够，还需要到系统设置里放行比如所有来源的应用之类的。总之门槛越来越高。同时，Homebrew 将在 2026 年 9 月 1 日开始，对于没有通过签名校验的应用，将无法再通过 brew cask 下载安装。趁着这次打算给 PicGo 做点商业化的机会，我也决定注册苹果开发者账号，然后给 PicGo 上签名了。本篇文章就记录一下 Electron 应用在 macOS 上做签名和公证的过程。

<!-- more -->

## 1. 获取 Team ID

首先你得注册一个[苹果开发者账号](https://developer.apple.com/cn/programs/enroll/)，而开发者账号的注册是另外一个话题了（我简单发了篇[小红书笔记](https://www.xiaohongshu.com/discovery/item/694c9864000000000d03c43c?source=webshare&xhsshare=pc_web&xsec_token=ABAOC_W3g3S6rpYBXGuLtqEMJZBJkjhhxb89_iEt1hRww=&xsec_source=pc_share)可以参考），我是用我一直以来的苹果账号注册的，没啥问题，12 月 24 日下午申请注册，24 日晚上就收到通过邮件了。

通过之后，可以在[开发者账号页面](https://developer.apple.com/account) 找到 Team ID，记录下来，后面有用。
![](https://pics.molunerfinn.com/blog/20251225135339137.png)


## 2. 获取证书

我们最终的目标，是导出一个被认证的 p12 文件。

1. 创建一个 csr 文件
2. 上传 csr 文件，获得一个 cer 证书
3. 将证书导入钥匙串，导出 p12 文件。


![](https://pics.molunerfinn.com/blog/20251225110811051.png)

然后邮件地址我都填的我苹果开发者账号注册的邮箱地址，request 选择存到本地，注意勾上最下面那个指定密钥对！
![](https://pics.molunerfinn.com/blog/20251225110949259.png)

然后登录 [苹果开发者官网](https://developer.apple.com/account/resources/certificates/list) 申请证书，我暂时还没打算上架 Mac App Store，先选择 Developer ID Application。
![](https://pics.molunerfinn.com/blog/20251225112630318.png)


然后导入刚刚的 csr 文件，就能下载这个 cer 证书，证书是有有效期的，意味着到期之后要重新生成。
![](https://pics.molunerfinn.com/blog/20251225112822355.png)

如果是申请的 Mac Store App 分发的证书，这里将会只有一年的有效期。
![](https://pics.molunerfinn.com/blog/20251225111548504.png)

然后就可以双击这个文件导入你的证书到 keychain（钥匙链），注意导入到 login（登录） 中。如果发现导入后遇到证书不信任的问题，说明还有一些额外的证书需要先导入。你可以到刚刚在苹果开发者网站申请证书的底部找到这些证书，下载，双击安装到 login 中。

![](https://pics.molunerfinn.com/blog/20251225122315244.png)
然后把你刚刚那个提示证书不信任的证书，删掉，然后重新导入。重新导入后，你在 Certifacates 或者 My Certificates 里就能看到已经确认的证书信息（注意有个小三角，点开，下面展示是你的私钥，这个很重要，我们需要用它导出 P12 文件）
![](https://pics.molunerfinn.com/blog/20251225122949764.png)
然后右键证书，导出 P12 文件，这个导出的时候会要求你输入一个密码，这个密码你可以自己定，是用来保护这个 P12 文件的。

导出 P12 文件之后，将其 Base64 字符串导出到剪贴板里，可以复制到某个地方，后文会用到。

```bash
base64 -i certificate.p12 | pbcopy
```

## 3. 创建标识符 && App 专有密码

前往[苹果开发者账号](https://developer.apple.com/account/resources/identifiers/list/bundleId)页面，创建标识符。
![](https://pics.molunerfinn.com/blog/20251225133331752.png)

目前来说暂时不用申请什么权限，如果之后需要的话可以再加入就行。输入 Description 和 Bundle ID 即可继续注册。Bundle ID 自己取，独一无二即可，通常的格式页面里也告诉你了。
![](https://pics.molunerfinn.com/blog/20251225134135591.png)

然后去苹果官方的[账号管理页面](https://account.apple.com/account/manage)，申请一个 App Specific Passwords：
![](https://pics.molunerfinn.com/blog/20251225134359709.png)

申请完成后，需要自己找个地方好好存起来，只会展示一次：
![](https://pics.molunerfinn.com/blog/20251225134638662.png)

## 4. 签名和公证

签名和公证在很多时候会被混为一谈，实际上它们是有区别的：

- **签名 (Code Signing)**：证明“这是我写的，且没被篡改过”。
- **公证 (Notarization)**：证明“苹果查过了，这软件没毒”。

如果没有公证的话，就会遇到经典的 `PicGo.app 已损坏` 的弹窗。同时，公证的前提是需要有合法的签名。因此注册一个苹果开发者账号，交 `$99` 年费之后看来是每年必备了。

接下来就是整理和收集上面的各种信息，把它们放到环境变量里，并进行签名和公证了。我自己是在 PicGo 项目的本地创建了一个 `.env` 文件，内容如下

```bash
# 第 1 步获取的 TEAM ID
APPLE_TEAM_ID=xxx
# 这个是你注册苹果开发者账号的邮箱
APPLE_ID=xxx
# 这个是第 3 步获取的 APP 特定密码
APPLE_APP_SPECIFIC_PASSWORD=xxx
# 这个是第 2 步在导出 P12 文件的时候要求输入的密码
CSC_KEY_PASSWORD=xxx
# 这个是第 2 步导出的 P12 文件的 Base64 格式的字符串
CSC_LINK=xxx
```

然后因为我用的是 electron-builder，所以只要环境变量里有 `CSC_KEY_PASSWORD` 和 `CSC_LINK` 的话，构建的时候就会自动签名。


而公证需要在签名之后，把签名后的 app 文件上传到苹果服务器验证一圈，通过之后你的软件才不会被 gatekeeper 给拦截下来。所以公证我们需要利用 electron-builder config 里的一个 afterSign 的属性，执行一个公证的脚本（利用 [electron/notarize](https://github.com/electron/notarize) 这个包提供的能力），同时根据 [electron/notarize](https://github.com/electron/notarize) 的文档，还需要开启 hardenedRuntime 和一些权限，如下：

```js
const config = {
  // ...
  afterSign: 'scripts/notarize.js' // 具体公证代码的脚本需要根据各自项目目录决定
  mac: {
    // ...
    hardenedRuntime: true, // 苹果要求，必须开启强化运行时
    entitlements: "build/entitlements.mac.plist", // 必须配合 entitlements 
    entitlementsInherit: "build/entitlements.mac.plist" // 必须配合 entitlements
  }
}
export default config
```

然后公证的脚本如下，核心就是调用 @electron/notarize 提供的能力，然后这里需要用到我们刚刚放到 `.env` 文件里的环境变量：

```js
// scripts/notarize.js
require('dotenv').config()

const { notarize } = require('@electron/notarize')
const { APPLE_ID, APPLE_TEAM_ID, APPLE_APP_SPECIFIC_PASSWORD } = process.env
const APP_BUNDLE_ID = 'com.molunerfinn.picgo'

async function main(context) {
  const { electronPlatformName, appOutDir, packager } = context

  if (
    electronPlatformName !== 'darwin' ||
    !APPLE_ID ||
    !APPLE_APP_SPECIFIC_PASSWORD ||
    !APPLE_TEAM_ID
  ) {
    console.log('Skip notarization.')
    return
  }

  const appName = packager.appInfo.productFilename
  const appPath = `${appOutDir}/${appName}.app`

  const now = Date.now()

  console.log('Starting Apple notarization for', appPath)

  await notarize({
    appPath,
    appBundleId: APP_BUNDLE_ID,
    appleId: APPLE_ID,
    appleIdPassword: APPLE_APP_SPECIFIC_PASSWORD,
    teamId: APPLE_TEAM_ID
  })

  console.log('Finished Apple notarization for', appPath, `in ${(Date.now() - now) / 1000}s`)
}


module.exports = main
```

同时还需要准备一个权限文件 `entitlements.mac.plist`，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <!-- 允许 JIT (Electron 必须) -->
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    
    <!-- 允许加载未签名的动态库 (插件、原生模块必须) -->
    <key>com.apple.security.cs.disable-library-validation</key>
    <true/>
    
    <!-- 允许执行内存中可写的页 (部分 Electron 版本需要) -->
    <key>com.apple.security.cs.allow-unsigned-executable-memory</key>
    <true/>
  </dict>
</plist>
```

然后就可以正常构建了，构建完成之后，electron-builder 会自动帮我们进行签名和公证。整个过程大概需要 10 分钟左右，具体时间取决于苹果服务器的响应速度。

最后，本地跑通后，将它们集成到 GitHub Actions 里即可，GitHub Actions 里需要把上面的环境变量都配置好即可。这样做我们就是完成了签名和公证，这样用户从网络上下载 PicGo 就不会再被拦下了：
![5359b6e102cc84a6af154e3f320bdcda.png](https://pics.molunerfinn.com/blog/5359b6e102cc84a6af154e3f320bdcda.png)

后续应该会考虑一下如何上架到 Mac App Store，到时候再继续补充吧。
