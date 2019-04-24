---
title: Nodejs学习日志（二）——Express+jade+stylus+mongodb搭建简单应用（上）
tags: 
  - 前端
  - Nodejs
categories:
  - Web
  - 开发
  - Nodejs
date: 2016-04-04 17:06:00
---
前段时间，把之前用Nodejs做的爬虫程序（参看之前的[cheerio爬虫](http://molunerfinn.com/nodejs-1/))，整合了一下。采用Nodejs市面上最普遍最常见的Express框架，简易地做了一个北邮人论坛每日十大的记录，暂且叫为[论坛日报](http://topten.piegg.cn)。在应用框架的过程中，我发现绝大部分的教程，还仅是在教你用Express写个简单的`Hello World`，然后就没有然后了。或者就是写多人博客，但是在一些关键的地方如数据的录入，读取等地方却没有详尽的描述。导致我在这个项目一开始真的是寸步难行。有数据但是不知道怎么写进数据库，怎么读出来->渲染到前端。  

这篇文章算是我自己做项目的切身体会，能够了解到Express基本使用、路由基本使用、数据写入读出、模板引擎渲染等等。希望能够帮上想要用Nodejs、Express做些简单应用的小伙伴们吧。  
<!--more-->

## Express的安装和使用

要做一个Express应用，首先要先下载、安装Express。在命令行界面下（WIN用CMD，MAC，LINUX就用终端就行），输入`npm install -g express-generator`或者如果你装了cnpm也可以用`cnpm install -g express-generator`（下同）。之所以不直接安装`express`，是因为用`express-generator`能够直接生成一个`express`应用的骨架。  

接着，在想要创建项目的地方，输入：`express test1`（注：test1是随便取的项目名字），于是命令行就会出现大致是如下的结果：

```js
create : test1
create : test1/package.json
create : test1/app.js
create : test1/public
create : test1/public/javascripts
create : test1/public/images
create : test1/public/stylesheets
create : test1/public/stylesheets/style.css
create : test1/routes
create : test1/routes/index.js
create : test1/routes/users.js
create : test1/views
create : test1/views/index.jade
create : test1/views/layout.jade
create : test1/views/error.jade
create : test1/bin
create : test1/bin/www
```

看，`express-generator`就帮我们创建了一个基本的express应用的骨架。

到项目根目录，也就是`package.json`所在的目录，然后在命令行内输入`npm install`或者`cnpm install`安装依赖。

然后在项目根目录下，输入`npm start`，或者`node ./bin/www`，就能启动express。在你的浏览器地址栏里输入`127.0.0.1:3000`是不是就能看见效果了？

## 来个Express版的HelloWorld

作为一个程序、项目的起始，一个Helloworld，总是不可避免。那么我们也来写一个。很简单。首先找到routes目录下的`index.js`，你能看见：

```js
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});
```

我们紧接这段话后面加上：

```js
router.get('/helloworld', function(req, res) {
  res.render('helloworld', { title: 'Hello, World!' });
});
```

router.get是一个方法，用get方法收到来自`/helloworld`的访问请求，然后在回调的函数里，用res.render()方法渲染所要的东西。render的第一个参数`helloworld`对应的就是`/helloworld`，意思是向`/helloworld`这个地址渲染东西。而后面的`{ title: 'Hello, World!' }`是指渲染的数据。这在接下去的jade里面会用到。

然后在根目录下找到views文件夹，进入，创建一个叫做`helloworld.jade`的文件，打开，输入：

```jade
extends layout

block content
    h1= title
    p Welcome to #{title}
```
然后回到根目录，输入`npm start`，或者`node ./bin/www`，启动express。在你的浏览器地址栏里输入`127.0.0.1:3000/helloworld`是不是就能看见**`Hello，World！`**了？

可以看出，我们从render里创建的`{ title: 'Hello, World!'}`传到了jade里的`title`了，是不是很酷？当然前提是，关于jade的语法你可能需要有所了解。


## Mongodb数据库的简易搭建

由于要做的是个应用，这个应用需要跟后台的数据进行交互。如果用不上数据库的话，那么我们也就没必要用上express这个框架了，写个静态的页面就行了。在Nodejs出来之前，如果要遇到跟后台交互的部分，通常情况下用的是PHP,Python,Java等语言来实现后端的逻辑。然而Nodejs诞生之后，js也能在后端发挥它的作用后，前后都能用js来写了。扯远了。这里我们没用通常情况下耳熟能详的Mysql这个数据库，而是用Mongodb这个Nosql数据库。它能够采用json的格式进行数据存储，前期的入门来说更简单，而且速度还很快，性能还很好。更重要的是还是开源免费的项目。总之优点很多。简易说下步骤：

### 下载mongodb

首先去官网下载对应操作系统、版本的mongodb：`https://www.mongodb.org/downloads#production`

### windows版的安装开启

对于windows用户来说，会下载到一个msi文件，点击安装。它能自动安装mongodb。然后装完之后，找到安装的目录，找到所在目录的`bin`文件夹。（你可以将其复制到任何一个你想要的地方，然后下面的步骤的是一样的。）进入，然后打开命令行工具，输入：`mongod --logpath "D:/Web/Mongodb/mongodb.log" --logappend --dbpath "D:/Web/Mongodb/data"`注意，我这里的路径是我自己定义的。你需要自己依照自己的情况创建相应的目录。  

简要说明一下，`mongod`是bin下的一个文件，它能执行开启mongodb服务的功能。所以如果想要直接用`mongod`这个命令在系统的任意地方使用，你需要将其的路径添加到系统的path。
`--logpath`是mongodb的日志文件路径，`--dbpath`是mongodb的数据文件路径。`--logappend`是一个追加日志的选项，可以不指定。这些路径的指定在哪里其实都并没有太多问题，因为mongodb服务一旦开启，你在哪里都能使用。

如果没有报错的话，在浏览器地址栏输入`127.0.0.1:27017`,只要能打开看到`It looks like you are trying to access MongoDB over HTTP on the native driver port.`这句话，说明你的mongodb的服务已经成功开启。

### Linux版的安装开启

对于Linux用户来说，安装mongodb实际上只是个解压的过程。我习惯是将mongodb的压缩包解压到/usr/local/mongodb/下。然后创建两个文件夹，一个是data，一个是log，分别用于存放数据文件和日志文件。然后进入bin下，执行`./mongod --dbpath=/usr/local/mongodb/data –logpath=/usr/local/mongodb/log/mongo.log --logappend`。  

跟windows版一样，如果没有报错的话，在浏览器地址栏输入`*.*.*.*:27017`，（注：`*.*.*.*`是你的linux服务器的地址，如果本机是linux，那么就是`127.0.0.1`）只要能打开看到`It looks like you are trying to access MongoDB over HTTP on the native driver port.`这句话，说明你的mongodb的服务已经成功开启。

### Mac版的安装开启

因为手头没有Mac的机器，可以参见Linux版的安装开启。

### 注意事项

服务一旦开启，就能在系统的各个角落连上它。不过它会占用一个窗口。所以如果用linux的话推荐新开一个screen或者就是多开一个终端。用windows或者MAC的话就是多开一个终端让它的服务跑起来就是了。

## 安装Nodejs的mongodb依赖

这个地方走了不少的坑，由于我用于连接mongodb的工具不是常见的`mongoose`而是`monk`，而`monk`需要的Nodejs的`mongodb`驱动的版本又是比较特别的，太高还不兼容。所以
这里就需要进行一些改动。  

在test1的文件夹下，可以发现一个叫做`package.json`的文件，在这个文件的`"dependence"`下加入两个依赖：

```js
"mongodb": "^1.4.4",
"monk": "^1.0.1"
```

至于为什么我没用`mongoose`而是`monk`，是因为`monk`更容易上手。如果你犹豫不决的话可以参考这个：  
> http://stackoverflow.com/questions/23615377/monk-vs-mongoose-for-mongodb

然后回到项目根目录，也就是`package.json`所在的目录，然后在命令行内输入`npm install`或者`cnpm install`，就能将刚才那两个mongodb的依赖加入了。

注：`mongodb`这个依赖不是安装了`mongodb`这个数据库，而是安装了Nodejs下对于`mongodb`数据库的驱动。而`monk`这个库才是对于数据库进行操作的依赖。

## 未完待续

下一篇文章将会讲述如何进行数据的写入和读出，让你的页面真正不再是静态页面，而是能“动起来”。
