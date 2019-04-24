---
title: Nodejs学习日志（一）——cheerio爬虫
tags: 
  - 前端
  - Nodejs
categories:
  - Web
  - 开发
  - Nodejs
date: 2016-02-23 22:02:00
---

等了好久的寒假等了好久的空闲时间。大三的课还是忙到想学东西都没有足够的时间。学习新事物的速度跟大二的时候一比简直不忍直视。不过也有一部分原因是在于入门前端后，现在所处的阶段正好是瓶颈期。发发牢骚，有时间了，该学还是要学的。

<!--more-->

## Nodejs的基本认知

第一次接触Nodejs这个名词的具体时间已经忘了。但是自我15年5月开始学习前端后，这个名词就一直没从我的视线里消失过。官方解释，不管是英文的还是中文的版本，大部分人都已经看过了。我只说说我自己的理解：**Nodejs是个可以运行在服务器端的一个js运行环境；得益于谷歌的V8引擎，对于js解析的速度非常快；单线程，但是由于事件驱动与非阻塞的I/O让它轻量而高效。**从此前端（js）开发人员手中有了一个利器——前后都能由前端人员用js来写了。  
因为Nodejs的出现，一大堆因此衍生的东西从此不可阻挡地诞生在前端世界中。

## Nodejs的基本安装

我手头的机器的操作系统目前只有Windows和Linux。Mac版的我想应该也跟Windows一样很容易安装吧。 

### Windows版本Nodejs的安装

进nodejs的[英文官网](https://nodejs.org/en/ "https://nodejs.org/en/"),首先映入眼帘的就是两个大大的For windows用户的两个版本：LTS版本（长期支持版本）和STABLE稳定版本（拥有最新特性）。不管你选择哪个版本都是可以的。当然作为喜欢尝鲜的人自然会选择后者。这两个都是msi格式的文件，下载后可以傻瓜式地安装，并且会自动地将系统的环境变量PATH写好，以及装上相应版本的NPM（nodejs的插件包管理工具）。

### Linux版本Nodejs的安装

我不是很推荐用各个发行版自带的包管理工具例如`yum`,`apt-get`来直接install nodejs。因为版本跟不上。很多都还是node在0.10.*的版本。我自己的经验走来，可以推荐以下的步骤（安装**已编译**的nodejs）：

- 首先可以去`http://nodejs.org/download/`这里找到所需要的版本
- 然后在命令行下，选好下载的路径，`wget https://nodejs.org/download/release/v5.7.0/node-v5.7.0-linux-x64.tar.gz`(我这里选的是5.7的以编译好的版本)
- 接着`sudo tar --strip-components 1 -xzvf node-v* -C /usr/local`
- 安装好后可以用`node --version`或者`node -v`检查一下是否安装成功

而且这样做的话，更新还方便。

### MacOS版本Nodejs的安装

可以直接去官网下载dmg镜像文件安装，当然也可以用过brew直接`brew install nodejs`的方式来安装。装好nodejs之后可以通过`npm install -g n`安装一个管理nodejs版本的工具n，可以很方便的进行nodejs的升级。

### cnpm安装

由于自己吃了亏，所以不想让读者也吃亏。在国内的朋友，最好还是将官方的npm工具换成[cnpm](https://github.com/cnpm/cnpm)，这是因为npm的源在国外，会导致经常性的模块、插件下载失败。而cnpm这个工具，官方解释：`npm client for China mirror of npm`，npm的中国镜像的客户端。这个镜像源是淘宝做的：`https://npm.taobao.org/`。在这里衷心感谢淘宝的前端团队为国内前端开发人员做出的巨大贡献。这个镜像源跟官方同步是每10分钟一次，所以不必担心模块和插件包的版本跟不上的问题。而且下载速度是真心快。

安装方法官方给出两个：

- `$ npm install -g cnpm --registry=https://registry.npm.taobao.org`

- 
```
`alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"

# Or alias it in .bashrc or .zshrc
$ echo '\n#alias for cnpm\nalias cnpm="npm --registry=https://registry.npm.taobao.org \
  --cache=$HOME/.npm/.cache/cnpm \
  --disturl=https://npm.taobao.org/dist \
  --userconfig=$HOME/.cnpmrc"' >> ~/.zshrc && source ~/.zshr`
```

安装结束后，就可以用`cnpm`这个命令代替`npm`这个命令了。

------

## Nodejs-爬虫初探

曾经自己用2小时看完教程模仿着写出过python的小爬虫，很方便也很强大。这次正好有爬取数据的需求。我就想着能否用nodejs来完成爬虫的任务——因为我觉得这对于nodejs来说应该是很容易的吧。但是事实上并没有想象中那么简单。至少从搜索结果上来说，用nodejs来做爬虫还并没有用python来做爬虫受欢迎。毕竟相对来说，nodejs是新鲜事物以及准入门槛也相对高了点。不过还是让我发现了叫做[cheerio](https://github.com/cheeriojs/cheerio)的这个nodejs里用来处理网页文档的利器。  

> 注：以下的内容需要稍微有些前端的知识。  

我们来分析一下，一个网络爬虫要做的事情：

- 获取网页内容（http\request\superagent等）
- 筛选网页信息（cheerio）
- 输出或存储信息（console\fs\mongodb等）

这篇文章标题之所以没有涉及到其他模块仅仅提到了cheerio，是因为你可以看到，无论是第一步还是第三步都有多种可选项。而关键的筛选网页信息的部分只有cheerio是目前最优的选择。

### cheerio-最优选之一

简单来说，cheerio就是服务器端的jQuery，去掉了jQuery的一些效果类和请求类等等功能后，仅保留核心对dom操作的部分，因此能够对dom进行和jQuery一样方便的操作。它是我们筛选数据的利器——把多余的html标签去掉，只留下我们想要的内容的重要工具。  

而获取网页内容，则有很多种方案了。综合一下我能给出几种：

- Nodejs自带-http模块（异步）
- 第三方-request模块（异步）
- 第三方-superagent模块（异步）
- 第三方-sync-request模块（同步）

其实作为一个很简单的爬虫——没有登录注册、没有翻页的话，这几个任何一个都是可以的。如果有涉及到相对复杂的操作（请求，cookie，验证等等）的话，推荐采用第二个模块或者第三个模块。本文由于我自己实践所限，仅会提到`superagent`以及`sync-request`这两个第三方模块。  

### 引入模块

好了我们就开始写第一个小爬虫吧。第一个爬虫的目标是爬取[豆瓣电影](http://movie.douban.com)指定电影页的相应信息（例如，导演名，电影名，主演名，时长，类型，评分，人数等等）。

这个爬虫会用到三个模块：

- cheerio
- sync-request
- fs（nodejs文件系统模块）

在项目文件夹下可以输入如下命令：

`cnpm install cheerio sync-request --save`，这样就能将模块下载好，让我们在文件中引入。新建一个`index.js`的文件。  
在头部引入模块：
```js
var cheerio = require('cheerio');  
var request = require('sync-request');  
var fs = require('fs');
```

### 程序正文

然后是获取网页内容：  

```js
url = 'http://movie.douban.com/subject/25724855/'; //这里是举个例子而已，豆瓣的具体的电影网址可以自己替换
var html = '';
html = request('GET', url).getBody().toString();
```
在这里，同步获取的方式的优点就体现出来了，能够直接将获取的内容输出给变量，简单方便。但是缺点就是万一内容太多，速度将会很慢，并且没有错误判断机制。而这些就是异步获取的优势了。

根据我们要获取的内容，可以在网页相应部分右键-审查元素，查找相应的内容被什么标签包含着，然后用cheerio的dom操作将所需内容给剥离出来：

```js
function handleDB(html){
  var $ = cheerio.load(html); //引入cheerio的方法。这样的引入方法可以很好的结合jQuery的用法。
  var info = $('#info');
  // 获取电影名
  var movieName = $('#content>h1>span').filter(function(i,el){
    return $(this).attr('property') === 'v:itemreviewed';
  }).text();
  // 获取影片导演名
  var directories = '- 导演：' + $('#info span a').filter(function(i,el){
    return $(this).attr('rel') === 'v:directedBy';
  }).text();
  // 获取影片演员
  var starsName = '- 主演：';
  $('.actor .attrs a').each(function(i,elem){
      starsName += $(this).text() + '/';
  });
  // 获取片长
  var runTime = '- 片长：' + $('#info span').filter(function(i,el){
    return $(this).attr('property') === 'v:runtime';
  }).text();
  // 获取影片类型  
  var kind = $('#info span').filter(function(i,el){
    return $(this).attr('property') === 'v:genre'
  }).text();
    // 处理影片类型数据
  var kLength = kind.length;
  var kinds = '- 影  片类型：';
  for (i = 0; i < kLength; i += 2){
    kinds += kind.slice(i,i+2) + '/';
  }
  // 获取电影评分和电影评分人数
    // 豆瓣
  var DBScore = $('.ll.rating_num').text();
  var DBVotes = $('a.rating_people>span').text().replace(/\B(?=(\d{3})+$)/g,',');
  var DB = '- 豆  瓣评分：' + DBScore + '/10' + '(' + 'from' + DBVotes + 'users' + ')';
    // IMDBLink
  IMDBLink = $('#info').children().last().prev().attr('href');
    
  var data = movieName + '\r\n' + directories + '\r\n' + starsName + '\r\n' + runTime + '\r\n' + kinds + '\r\n'+ DB +'\r\n';
  // 输出文件
  fs.appendFile('dbmovie.txt', data, 'utf-8', function(err){
    if (err) throw err;
    else console.log('大体信息写入成功'+'\r\n' + data)
  });
}
```
从上面可以看出，对于dom的操作，cheerio和jQuery几乎是一致的。这些方法，只要你会用jQuery都不会陌生。最关键的部分基本上都是调用*.text()的方法获取文本内容。当然，获取一些链接之类的属性名自然会用相应的方法，这也是无可厚非。

这个爬虫有个比较关键的地方在于，需要爬取相应电影的IMDB评分。因为在豆瓣的电影页里，会有专门的一行放置IMDB评分链接的地方。我们通过上面叫做`handleDB`的方法能够获取到IMDB的链接，然后再用类似的办法在IMDB页中获取相应的评分和人数。

`Link = request('GET', IMDBLink).getBody().toString();`

然后再用获取豆瓣电影相应信息的方法类似的再获取一遍：

```js
function handleIMDB(Link){
  var $ = cheerio.load(Link);
  // 获取IMDB评分
  var IMDBScore = $('.ratingValue span').filter(function(i,el){
    return $(this).attr('itemprop') === 'ratingValue';
  }).text();
  // 获取IMDB评分人数
  var IMDBVotes = $('.small').filter(function(i,el){
    return $(this).attr('itemprop') === 'ratingCount';
  }).text();
  // 字符串拼接
  var IMDB = '- IMDB评分：' + IMDBScore + '/10' + '(' + 'from' + IMDBVotes + 'users' + ')' + '\r\n';
  // 输出文件
  fs.appendFile('dbmovie.txt', IMDB, 'utf-8', function(err){
    if (err) throw err;
    else console.log('IMDB信息写入成功' + '\r\n' + IMDB)
  });
}
```
由于cheerio对于dom操作的部分，跟jQuery没区别所以我就不再赘述，如果你有疑惑，可以参考cheerio的[官方说明](https://github.com/cheeriojs/cheerio#api)。重点说一下fs模块。这个模块在我们的爬虫里是作为输出爬取数据的最终步骤。简单的说说我们所用的api。（fs全部api官方文档请看[这里](https://nodejs.org/dist/latest-v5.x/docs/api/fs.html))。  

`fs.appendFile(file, data[, options], callback)`

`fs.appendFile()`是个异步输出文件的方法。它是的作用是追加文件内容。如果这个文件不存在那么就会新建一个，并写入追加内容。其中：

- file->文件名
- data->要写入的内容
- options->选项，可以指定输出的文件编码格式
- callback->回调函数，可以用于输出错误或者输出相应读写成功信息

其中callback中会传递一个error参数用于反馈错误所用。我们在这里主要用于输出成功信息。当然如果发生错误将会报错。让我们来看看输出结果：  
`dbmovie.txt`:  

    房间 Room  
    - 导演：伦尼·阿伯拉罕森  
    - 主演：布丽·拉尔森/雅各布·特伦布莱/琼·艾伦/肖恩·布里吉格斯/威廉姆·H·梅西/梅根·帕克/阿曼达·布鲁盖尔/卡斯·安瓦尔/乔·平格/温迪·古逊/兰道尔·爱德华/杰克·富尔顿/汤姆·麦卡穆斯/  
    - 片长：118分钟  
    - 影  片类型：剧情/家庭/  
    - 豆  瓣评分：8.7/10(from21,211users)  
    - IMDB评分：8.3/10(from56,920users)  


是不是还不错呢？不过这个方法有个缺陷，因为node只能识别utf-8编码格式，所以一旦我们爬取的网页是非utf-8，例如GBK、GB2312等，那么该如何应对？下面将会讲述解决办法

### 处理GBK\GB2312等编码网页的爬虫

我们在爬取网页的时候尤其要注意编码格式。一旦爬取的编码格式和输出的编码格式不一致将会产生乱码。所以遇到较为早期的网站用的是非utf-8编码的将会比较头疼。不过也并不是没有办法。网上比较常见的是用[iconv-lite](https://github.com/ashtuchkin/iconv-lite)这个第三方库进行转码。不过有个更方便的第三方库[superagent-charset](https://github.com/magicdawn/superagent-charset)，它能够在采集网页信息后调用一个`.charset()`的方法就能实现同样的功能。因此在这篇文章里我将会简要说明一下这个方法。  

> 注：`superagent-charset`模块依赖于`superagent`模块

举个具体实例。这个例子我们将要爬取[北邮人论坛](http://bbs.byr.cn)的每日十大的信息，将它们封装成json数组存起来。为此我们将采取从rss作为信息源来获取我们想要的信息。    

为此我们同样先安装好模块。然后新建一个`index.js`文件。

### 引入模块

```js
var cheerio = require('cheerio');  
var superagent = require('superagent-charset');  
var fs = require('fs'); 
```

### 程序正文

```js
var url = 'http://bbs.byr.cn/rss/topten';
var toptens = []; // 初始化json数组

superagent.get(url) // 获取网页内容
  .charset('gb2312') // 转码-将gb2312格式转成utf-8
  .end(function (err, res) {
    // 常规的错误处理
    if (err) {
      return next(err);
    }
    var $ = cheerio.load(res.text,{
      xmlMode: true // 由于从rss里读取xml，所以这一步一定要有，切记
    });
    
    var d = new Date();
    var date = d.getFullYear()+"-"+(d.getMonth()+1)+"-"+(d.getDate()); // 取得爬取的日期

    var topten = { // 设定爬取的json数组
      date: date,
      info: []
    };

    // 具体爬取内容，主要都是cheerio操作了
    $('item').each(function(i,el){
      i += 1;
      topten.info.push({
        topno: i,
        title: $(this).find('title').text(),
        author: $(this).find('author').text(),
        pubDate: $(this).find('pubDate').text(),
        boardName: $(this).find('guid').text().replace(/http:\/\/bbs.byr.cn\/article\//,'').replace(/\/\d+/,'').trim(),
        link: $(this).find('link').text(),
        content: $(this).find('description').text()
      })
    })
    toptens.unshift({topten: topten}); // 从原有的json数据之前追加json数据

    var json = JSON.stringify(toptens); // json格式解析，这步也是一定要有

    fs.writeFile('toptens.json', json, 'utf-8', function(err){
      if (err) throw err;
      else console.log('JSON写入成功'+'\r\n' + json)
    });
  });
```

从上面我们可以看出，`superagent-charset`模块就是在`.get()`方法之后追加了一个`.charset()`的方法用于转换网页编码格式，这个就很方便。同时可以注意到，主程序实际上是放在一个回调函数里的——`.end(function(err,res){...})`，在获取网页内容结束（end）后，进入function这个回调函数，进行内容的筛选。这就是`superagent`与`sync-request`模块的区别之处了。异步的方式对于初学者来说相对会有些难尤其是在一些输出内容上会有与想法出入的地方——这是因为node本身的原因决定了异步回调才是node的精髓与难点。

## 总结

这次的爬虫练习跟之前用python写的爬虫感觉还是有很大的不同——python的爬虫入门更依赖于正则表达式对于网页内容的切割。然而用了node之后，用了cheerio进行的内容筛选更符合前端的思维。对于dom的操作毕竟还是比用正则表达式来的直观。并且二者实际上实现简单功能来说，程序的复杂度也都并没有很高。既然如此，对于纯前端的人员来说，爬虫从nodejs学起，其实也不是一件很难的事，而是一件很棒的事。

附上cheerio的一些教程的地址以及我自己做的DEMO在github上的地址：

- cheerio的学习：`http://www.hubwiz.com/course/5636b7a11bc20c980538e998/`，讲解的很清晰。
- 通读cheerio API： `https://cnodejs.org/topic/5203a71844e76d216a727d2e`，没学过jQuery的人可以看看
- dbmovie: `https://github.com/Molunerfinn/dbmovie-spider`
- Nodejs-ByrTopTen: `https://github.com/Molunerfinn/Nodejs-ByrTopTen`

欢迎评论！有的地方写得不好请指出！作者原创，需要转载请联系我。
