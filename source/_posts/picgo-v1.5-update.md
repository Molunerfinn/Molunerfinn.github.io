title: 基于Electron-vue的图床上传工具PicGo v1.5更新说明
tags: 
  - 前端
  - Vue
  - Electron
  - Electron-vue
categories:
  - Web
  - 开发
date: 2018-05-08 21:25:00
---
经过一个多月的努（lan）力（duo）开发，基于electron的图床上传工具[PicGo](https://github.com/Molunerfinn/PicGo)终于迎来了一个minor版本的更新。如果你对此感兴趣，不妨看看都更新了哪些有趣而实用的功能吧。

<!-- more -->

### 支持GitHub图床

早先PicGo所支持的图床基本上都是属于国内的服务商提供的图床（如七牛、腾讯云COS等），这次更新加入了GitHub图床的支持。用GitHub做图床其实是不少写博客的朋友的做法。免费、原生支持HTTPS、GitHub仓库易于管理、和issue等功能无缝衔接都是它的优点。如果能接受GitHub在国内的访问速度不是特别快的缺点的话，用它来做你的图床是个不错的选择。来看看在PicGo里如何配置它：

**1. **首先你得有一个GitHub账号。注册GitHub就不用我多言。

**2. **新建一个仓库

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/create_new_repo.png)

记下你取的仓库名。

**3. **生成一个token用于PicGo操作你的仓库：

访问：https://github.com/settings/tokens

然后点击`Generate new token`。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/generate_new_token.png)

把repo的勾打上即可。然后翻到页面最底部，点击`Generate token`的绿色按钮生成token。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/20180508210435.png)

**注意：**这个token生成后只会显示一次！你要把这个token复制一下存到其他地方以备以后要用。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/copy_token.png)

**4. **配置PicGo

**注意：**仓库名的格式是`用户名/仓库`，比如我创建了一个叫做`test`的仓库，在PicGo里我要设定的仓库名就是`Molunerfinn/test`。一般我们选择`master`分支即可。然后记得点击确定以生效，然后可以点击`设为默认图床`来确保上传的图床是GitHub。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/setup_github.png)

至此配置完毕，已经可以使用了。当你上传的时候，你会发现你的仓库里也会增加新的图片了：

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/success.png)

### 支持腾讯云COS v5版本

> 在支持腾讯云COS的路上，我可谓是费了一番心血。首先是官方提供的node-sdk对我来说基本属于瘫痪状态，只能上传具体文件而不能上传base64编码后的文件。而且居然还有v4和v5两个版本的COS，甚至两个版本的认证签名、上传url等等都**完！全！不！同！**。由于之前我只有v4版本的COS权限，只能开发和测试出v4版本的上传。而近来发现很多朋友用的都已经是v5版本的了，所以我提交了一个工单向腾讯云申请了v5版本的权限，没想到很快就给我派发权限了。于是就有了v5版本的面世。目前市面上能同时支持v4、v5版本COS的估计也只有PicGo了！

如果你是v5用户，但是之前下载了PicGo却不能用的话，别担心，v1.5版本的配置跟之前的配置几乎一致，而且可以一键切换v4\v5版本。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/v5_setup.png)

**1. **获取你的APPID、SecretId和SecretKey

访问：https://console.cloud.tencent.com/cam/capi

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/get_key_id_secret.png)

**2. **获取bucket名以及存储区域代号

访问：https://console.cloud.tencent.com/cos5/bucket

创建一个存储桶。然后找到你的存储桶名和存储区域代号：

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/get_bucket_area.png)

v5版本的存储桶名称格式是`bucket-appId`，类似于`xxxx-12312313`。存储区域代码和v4版本的也有所区别，v5版本的如我的是`ap-beijing`，别复制错了。

**3. **选择v5版本并点击确定

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/choose_v5.png)

然后记得点击`设为默认图床`，这样上传才会默认走的是腾讯云COS。

### 支持编辑相册的图片信息

有些时候可能上传的图片的url事后需要更改，比如修改http到https，比如加上一些操作后缀（例：七牛图床支持的`?imgslim`）等等。PicGo本次的更新也让你能够更方便地管理你的图片库。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/picgo_edit_info.gif)

### 支持上传图片前重命名文件名

PicGo总共有三种上传模式：

1. menubar图标拖拽上传（仅支持macOS）
2. 主窗口拖拽或者选择图片上传
3. 剪贴板图片（最常见的是截图）上传（支持自定义快捷键）

其中前两种都是可以明确获得文件名，而第三种无法获取文件名（因为剪贴板里有些图片比如截图根本就不存在文件名），所以PicGo此前采取的规则是使用时间戳来命名剪贴板里的图片。这也导致了无法自定义文件名的问题。本次更新你可以选择开启「上传前重命名」这个选项：

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/rename_before_upload.png)

之后你在上传的时候就会弹出一个小窗口让你重命名文件。如果你不想重命名，点击确定、取消或者直接关闭这个窗口都是可以的。如果你想要重命名就在输入框里输入想要更改的名字，然后点击确定即可。另外这个特性也支持批量上传，如下：

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/picgo_rename.gif)

### 支持查看当前上传的图床

在主窗口的上传区，你可以直观地看到当前默认上传的图床，再也不用到处找当前的默认图床是哪个啦。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/current_picbed.png)

### 支持显示或隐藏相应的图床

很多时候你并不会使用上PicGo给你提供的全部的图床。所以为了精简显示你可以只选择你想要的图床来显示，这样侧边栏也就不会出现滚动条了。不过需要注意的是，这个仅仅是显示/隐藏而并不是剔除相应的功能。假如你隐藏了七牛云，你依然是可以通过七牛云来上传图片的。

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/picbed-choose.gif)

### 支持开机自启动

如果你觉得每次开机要主动开启PicGo是一件麻烦事，不妨试试让它开机自启吧~

![](https://raw.githubusercontent.com/Molunerfinn/test/master/picgo/autoStart.png)

### 修复若干bugs

v1.5不光更新了上述功能，也修复了不少问题。其中一个尤为重要的是从v1.4.1开始的一个bug——macOS的menubar无法拖拽上传。该bug也在这个版本被修复。

## 结语

PicGo第一个稳定版本是在少数派上发布的，详见[PicGo：基于 Electron 的图片上传工具](https://sspai.com/post/42310)。支持macOS和windows双平台，开源免费，界面美观，也得到了很多朋友的认可。本次更新也是充分聆听了大家的[意见](https://github.com/Molunerfinn/PicGo/issues/29)。如果你对它有什么意见或者建议，也欢迎在[issues](https://github.com/Molunerfinn/PicGo/issues)里指出。如果你喜欢它，不妨给它点个star或者请我喝杯咖啡（PicGo的GitHub[首页](https://github.com/Molunerfinn/PicGo)有赞助的二维码）？

> 下载地址：https://github.com/Molunerfinn/PicGo/releases

> Windows用户请下载`.exe`文件，macOS用户请下载`.dmg`文件。

Happy uploading！






