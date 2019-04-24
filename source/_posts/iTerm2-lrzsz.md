---
title: OSX下iTerm2实现rz/sz与服务器进行文件上传/下载 
categories: 
  - 其他
  - 极客
date: 2016-07-27 22:15:00
tags: Mac
---

在windows下用xshell、secureCRT等工具只要在服务端装好lrzsz工具包就可以实现简单方便的文件上传下载。昨天在Mac上用iTerm的时候发现iTerm原生不支持rz/sz命令，也就是不支持Zmodem来进行文件传输。遂查了一下，发现解决办法还算简单，不过有点小坑。

<!--more-->

## 通过brew安装lrzsz

首先先假定你安装了[Homebrew](http://brew.sh/)，然后我们通过它，先给Mac安装lrzsz。  
在终端下输入`brew install lrzsz`，静等一会即可安装完毕。

## 下载配置iTerm2的相关脚本

这一步，在给iTerm2加上相应配置前需要下载两个前人已经写好的脚本文件。

这里是[下载地址](https://github.com/mmastrac/iterm2-zmodem)，将iterm2-recv-zmodem.sh和iterm2-send-zmodem.sh下载到本机，然后将它们放到`/usr/local/bin`目录下。

如果你安装过`wget`，也可以在`/usr/local/bin`目录下直接执行：

`wget https://raw.githubusercontent.com/mmastrac/iterm2-zmodem/master/iterm2-send-zmodem.sh https://raw.githubusercontent.com/mmastrac/iterm2-zmodem/master/iterm2-recv-zmodem.sh`

然后我们将这两个文件赋予可执行权限：

`chmod +x /usr/local/bin/iterm2-send-zmodem.sh /usr/local/bin/iterm2-recv-zmodem.sh`

## 配置iTerm2

找到iTerm2的配置项：iTerm2的Preferences-> Profiles -> Default -> Advanced -> Triggers的Edit按钮。

然后配置项如下：

|Regular Expression|Action|Parameters|Instant|
|---|---|---|---|
|rz waiting to receive.\\\*\\\*B0100|Run Silent Coprocess|/usr/local/bin/iterm2-send-zmodem.sh|checked|
|\\\*\\\*B00000000000000|Run Silent Coprocess|/usr/local/bin/iterm2-recv-zmodem.sh|checked|

**尤其注意最后一项需要你将`Instant`选项勾上，否则将不生效**

注意看图：

![iterm2-lrzsz](https://img.piegg.cn/iterm2-lrzsz.png)

至此结束。只要服务端也已经装好了`lrzsz`工具包便可以方便地通过rz\sz来进行文件上传下载了。