---
title: 极空间虚拟机安装 Ubuntu 24.04 踩坑记录
description: 在极空间里装 Ubuntu 24.04 虚拟机时踩过的坑：离线安装、驱动避坑、装完再补网卡和换国内源。
tags:
  - 极空间
  - Ubuntu
  - 虚拟机
  - NAS
categories:
  - 折腾记录
date: 2026-02-16 00:00:00
---

最近想在极空间里装一个 Ubuntu 24.04 虚拟机，主要用来跑脚本和一些小工具。原本以为很快就能搞定，结果重装了几次才跑通。这里把关键步骤和坑点整理一下，尽量让你一次安装成功。

先说结论：安装阶段先不要加网卡，走离线安装；驱动安装那一步不要勾选额外驱动；系统装完后再把网卡加回去。

<!-- more -->

## 1. 先准备镜像和虚拟机参数

镜像可以在 [Ubuntu 官网](https://cn.ubuntu.com/download) 下载，也可以用清华等镜像源。

创建虚拟机之前，先把极空间网络设置改成网桥模式：

![bridge](https://pics.molunerfinn.com/blog/20260214234530046.png)

新建虚拟机时，我这边用的是以下配置：

- CPU / 内存：至少 2 核 4GB（资源够的话可以再加）
- 固件：UEFI
- 虚拟机框架：q35（默认即可）

![create new vm](https://pics.molunerfinn.com/blog/20260214232719808.png)

硬盘按用途分配就行，20GB 到 100GB 都常见。

![hard disk](https://pics.molunerfinn.com/blog/20260214233234183.png)

## 2. 第一个关键坑：安装前先删掉网卡

网卡这里建议先删掉，等系统装完再加回来。

![network settings](https://pics.molunerfinn.com/blog/20260214233353893.png)

创建完成后，点击虚拟机里的「访问」，通过网页 VNC 进入 Ubuntu 安装界面：

![visit](https://pics.molunerfinn.com/blog/20260214233550358.png)

## 3. 安装流程里要注意的点

安装界面我没留太多截图，这里只记最容易出问题的步骤：

1. 语言中英文都可以。
2. 因为前面删了网卡，所以会走离线安装，这样更稳也更快。
3. 遇到提示版本较低、是否更新时，先选跳过。
4. 安装方式选手动安装即可。
5. 询问是否安装额外驱动（比如显卡、音视频）时，不要勾选。
6. 其他步骤基本默认下一步即可。

我之前失败就是因为联网安装，还勾了额外驱动。最后网络不稳定，安装收尾阶段报错，连引导都没写进去，重启后直接进不去系统。

## 4. 安装完成后先做两件事

系统安装结束后先别急着用，先关机整理配置。

先点击强制关机：

![force shutdown](https://pics.molunerfinn.com/blog/20260214234132595.png)

然后在编辑页面里做两件事：

1. 卸载安装镜像（否则可能再次进入安装引导）
2. 把网卡加回来

![edit vm](https://pics.molunerfinn.com/blog/20260214234236020.png)

![unmount iso](https://pics.molunerfinn.com/blog/20260214234312189.png)

![add nic](https://pics.molunerfinn.com/blog/20260214234356749.png)

完成后再开机。

## 5. 可选优化：换国内软件源

系统进桌面后，可以把 Ubuntu 的软件源切到阿里云，更新会快很多。

![software source](https://pics.molunerfinn.com/blog/20260214234817064.png)

下载源选择「其他」，然后选 `aliyun.com` 的镜像地址：

![aliyun mirror](https://pics.molunerfinn.com/blog/20260214234913985.png)

关闭后会弹窗询问是否更新，这时候直接更新即可：

![update prompt](https://pics.molunerfinn.com/blog/20260214235000165.png)

也可以在终端手动更新：

```bash
sudo apt update && sudo apt upgrade -y
```

到这里，系统就可以正常用了。

## 小结

这次最关键的经验就三条：

1. 安装时先去掉网卡，离线装最稳。
2. 额外驱动先别装，避免安装尾声翻车。
3. 装完先卸载镜像、加回网卡，再开机进系统。

如果你也在极空间里装 Ubuntu 24.04，希望这篇能帮你少走几步弯路。
