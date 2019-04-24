---
title: Nodejs学习日志（三）——Express+jade+stylus+mongodb搭建简单应用（下）
tags: 
  - 前端
  - Nodejs
categories:
  - Web
  - 开发
  - Nodejs
date: 2016-04-10 15:52:00
---
前段时间，把之前用Nodejs做的爬虫程序（参看之前的[cheerio爬虫](http://molunerfinn.com/nodejs-1/))，整合了一下。采用Nodejs市面上最普遍最常见的Express框架，简易地做了一个北邮人论坛每日十大的记录，暂且叫为[论坛日报](http://topten.piegg.cn)。在应用框架的过程中，我发现绝大部分的教程，还仅是在教你用Express写个简单的`Hello World`，然后就没有然后了。或者就是写多人博客，但是在一些关键的地方如数据的录入，读取等地方却没有详尽的描述。导致我在这个项目一开始真的是寸步难行。有数据但是不知道怎么写进数据库，怎么读出来->渲染到前端。  

这篇文章算是我自己做项目的切身体会，能够了解到Express基本使用、路由基本使用、数据写入读出、模板引擎渲染等等。希望能够帮上想要用Nodejs、Express做些简单应用的小伙伴们吧。  

上一篇[文章](http://molunerfinn.com/nodejs-2/)已经讲到了将mongodb开启以及我们输出了第一个页面了，从后台向前台render了数据并渲染。这次我们讲讲如何从mongodb里取数据、写数据并输出前端。

<!--more-->

### mongodb的简单使用

在之前安装的mongodb的bin的目录下，打开命令行。如果是Windows输入`mongo`回车，如果是linux输入`./mongo`，就能进入mongodb的命令行界面。（前提是你开启了mongodb的服务）。

一般来说会有提示`connecting to: test`。`mongodb`默认是没有设置用户名和密码，这需要你手动设置。本文就不对这方面进行赘述。  
我们新建一个库。输入`use test`，就会切换到叫做`test`的数据库中。这个数据库在我们没有写入数据的时候，仅仅是被创建但是并未真正生成，如果没有写入数据，我们退出mongodb的话这个数据库就会自动销毁。接着我们写入第一个数据，让它真正创建。尤其开心的是，mongodb的数据存储采用的是`json`的格式，也就是很方便的能够进行构建和解析。

输入`db.nodetest.insert({"name":"Jack","age":"16","sex":"man"})`。这句话的意思是，在`test`这个数据库中，创建一个叫做`nodetest`的数据库表或者叫集合（`collection`)并插入一个数据。  

要查看我们插入的这个数据，只需要输入`db.nodetest.find()`就能够输出结果了。

这就是最简单的mongodb的操作。

### express与mongodb的对接

要将express与mongodb对接，需要在`app.js`这个文件里加入依赖。依赖包我们在上一篇文章里已经安装了。这次是写到`app.js`里。  
你可以看到开头有这么几句代码：

```js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
```
我们在这段代码后面加上几句，使它变成这样：
```js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

var mongo = require('mongodb');
var monk = require('monk');
var db = monk('localhost:27017/test');
```

这样就能够引入依赖了。但是我们还需要将`db`引入我们的路由中。

你能够看到两句代码：

```js
app.use('/',routes);
app.use('/users', users);
```
我们在这两句话的上方加上一段代码：

```js
app.use(function(req,res,next){
  req.db = db;
  next();
});
```

**注意：如果没有把这句话放到`app.use('/',routes);`上方，那么数据库将无法工作！**

到此，整个`app.js`应该长这样：

```js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var mongo = require('mongodb');
var monk = require('monk');
var db = monk('localhost:27017/test');

var routes = require('./routes/index');
var users = require('./routes/users');

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use(function(req,res,next){
  req.db = db;
  next();
});

app.use('/', routes);
app.use('/users', users);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handlers

// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
  app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
      message: err.message,
      error: err
    });
  });
}

// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: {}
  });
});


module.exports = app;
```

### 从mongodb中读取数据并渲染到前端

在`routes`文件夹下找到`index.js`

`module.exports = router`这句话的上方写入如下信息：

```js
router.get('/users', function(req, res) {
  var db = req.db;
  var user = db.get('nodetest');
  user.find({},{},function(e,docs){
    res.render('users', {
      "users" : docs
    });
  });
});
```
解释一下，我们之前创建了一个叫做`nodetest`的数据表，我们通过`db.get()`方式将其取出。但是这样的话不能直接用，必须通过调用`.find()`方法中的回调函数里的第二个参数`docs`——这个参数才是真正取出来的`json`数据，才是我们要用到输出到前端的数据。

然后我们到`view`文件夹下创建一个叫做`users.jade`的文件。注意，这里的`users`就是对应着路由文件里的`.get('/users', *** )`。

然后我们往jade文件里输入东西：

```html
extends layout

block content
  h1.
    User List
  ul
    each user, i in users
      li
        span= user.name
        span= user.age
        span= user.sex
```
这里我们用到了一个很关键的东西，注意这句话：
`each user, i in users`，这里的`user`是自己定义的数组元素名，i是这个元素在`users`这个表中的位置。而`user`可以随便自己改，但是`users`却是对应着我们之前在路由文件中render的数据名：`"users" : docs`，这个是不能随便改的，要改的话需要路由文件也更改，jade里面也更改才可以。

而`user.name`很明显就是取出了我们之前存在里面的数据。在这里应该是取出为`Jack`。

至此我们已经从`mongodb`中读出了数据。下一节是讲述如何从express往mongodb中写入数据。

### 将数据写入mongodb中并渲染到前端

这里我们创建一个叫做`adduser.jade`的前端页面来输入数据并提交到后台。

```js
extends layout

block content
  h1= title
  form(name="adduser",method="post",action="/adduser")
    input(type="text", placeholder="username", name="username")
    input(type="text", placeholder="userage", name="userage")
    input(type="text", placeholder="usersex", name="usersex")
    button(type="submit") submit
```

仅有这个jade是不够的，因为express不知道它的提交会提交到哪。所以需要在router里写指定的方法并返回数据。

继续在`index.js`里写入如下的东西：

```js
router.post('/adduser', function(req, res) {
  var db = req.db;
  var userName = req.body.username;
  var userAge = req.body.userage;
  var userSex = req.body.usersex;
  var user = db.get('nodetest');
  user.insert({
    "name" : userName,
    "age" : userAge,
    "sex" : userSex
  }, function (err, doc) {
      if (err) {
        res.send("Error");
      }
      else {
        res.redirect("users"); // 重定向到 /users
      }
  });
});
```
通过这个post的请求，我们将获取从adduser这个页面发送的提交信息。然后再将它组装后`insert`到mongodb中。成功后再重定向（跳转）到`users`这个页面，然后你就能看到刚刚输入的信息出现在`users`这个页面了。

整个前后都由js由自己掌控，是不是非常酷？

### 结语

作为一个前端开发人员，在nodejs诞生前，如果只会js你是无法在后端进行操作的。要做后台的处理，就必须依赖后端的语言例如php，python等等。如今，nodejs已经大面积渗透到前端的世界中，哪怕你只会用js也能前后都靠前端工程师来构建。本文就是采用了最简单的方式阐述了express最简单的前后交互。希望能给想用nodejs做开发的前端er一些启发~这毕竟也是作为我的nodejs学习日志，如果不出意外，每周都将更新新的干货。
