---
title: 把你的github操作从ssh转成https
tags: 
  - git
categories:
  - Web
  - 开发
date: 2017-10-26 21:44:00
---

从10月24日开始，由于总所周知的原因，某些地区一些运营商的网络环境下已经无法通过ssh的方式对一些国外服务器进行操作。很不幸github也因此被误杀。这对于广大程序猿来说，简直是一大噩耗。不过我发现通过https的方式还是可以对github进行操作的。毕竟技术是无罪的，不管怎么样，github总是要用的。所以可以将现有的ssh方式改成https。

<!-- more -->

## 从ssh到https

把原有项目从ssh的方式转成https其实很简单：

```bash
$ git remote set-url origin https://xxxxx #your https repo url
```

通过如下命令查看是否更改成功：

```bash
$ git remote -v
origin  https://github.com/xxx/xxx.github.io.git (fetch)
origin  https://github.com/xxx/xxx.github.io.git (push)
```

不过转成https之后会带来一个问题，那就是每次提交的时候都需要输入github的用户名密码。其实当初用ssh的方式除了安全之外很重要的一个原因就是不用每次都手动输入账号密码。不过其实https的方式也是可以实现的，只是需要一些额外配置。

## 配置https免输入密码

git官方手册里有对于[git-credential-store](https://git-scm.com/docs/git-credential-store)的描述。简单来说，就是将用户名和密码缓存在本地，每次提交的时候自动帮你填入用户名密码。

> 注意git版本需要在1.7以上

开启也很简单：

```bash
$ git config --global credential.helper store
```

然后你对某个仓库第一次执行push操作的时候，会要求输入用户密码，之后就再也不用了：

> 官方示例

```bash
$ git config credential.helper store
$ git push http://example.com/repo.git
Username: <type your username>
Password: <type your password>

[several days later]
$ git push http://example.com/repo.git
[your credentials are used automatically]
```

通过store的方式会将你的账号密码以明文的形式存在`~/.git-credentials`文件里，大致长这样：

```bash
$ cat ~/.git-credentials
https://username:password@github.com
```

如果你觉得这样不太安全，见下一章：

## 提高安全性

### Mac

对于mac而言，可以将git默认的`credential.helper`指定成`osxkeychain`

```bash
$ git config --global credential.helper osxkeychain
```

然后可以通过

```bash
$ git config credential.helper
osxkeychain
```

来查看是否指定成功了。之后用户名密码将会保存在系统自带的`keychain access`里。

### Windows

使用微软开发的[Git-Credential-Manager-for-Windows](https://github.com/Microsoft/Git-Credential-Manager-for-Windows)

最新版下载地址：https://github.com/Microsoft/Git-Credential-Manager-for-Windows/releases/latest

安装之后，在控制台里输入：

```cmd
git config --global credential.helper manager
```

之后还是一样的，某个项目里输入用户名密码之后，以后就再也不用输入了。你可以在`管理网络密码`里找到你的用户密码——其实不是密码，而是token了，因为`Git-Credential-Manager-for-Windows`它会自动调用github的api生成`Personal access tokens`，你可以在你的github的`Personal settings`里找到它。所以安全性还是有保障的！


## 总结

如果你已经无法用ssh的方式连接github的话，不妨试试https的方式。至少目前来说还是有效的，而且配置也不难~Happy coding again！
