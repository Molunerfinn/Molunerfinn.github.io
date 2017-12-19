title: Hexo主题开发经验杂谈 
tags: 
  - hexo
categories:
  - hexo
date: 2017-09-14 22:46:00
---

## 前言

之前学前端的初衷就是为了让自己的个人博客好看点。Hexo主题如今很大概率你能够看到[Next](https://github.com/iissnan/hexo-theme-next)主题以及它的一些个人修改、衍生版本。我记得去年在看一篇[Hexo主题开发指南](http://chensd.com/2016-06/hexo-theme-guide.html)的时候，有句话对我感触很深：

> 当你看到你用的主题出现在两个以上的博客的时候，那你就要考虑自己写一个了。

懒癌晚期的自己以及毕业设计等等事情的拖延，终于在最近完成了自己的Hexo主题——[Melody](https://github.com/Molunerfinn/hexo-theme-melody)。

本文将讲述如何制作一个`Hexo`主题，以及在制作过程中的一些坑和一些经验。

**在我主题制作过程中，Next主题以及其他一些优秀的前端博客例如[Hux](https://huangxuan.me/)，对我帮助和启发很大，再次感谢**

<!-- more -->

## 预备知识

### 模板引擎

Hexo的博客的页面都是通过模板引擎渲染的静态页面。当然如果你不想用模板引擎，用html也是可以的。但是这样做的话会很不方便，因为其实很多代码是可以复用的，用模板引擎的话可以比较方便地帮我们实现代码的复用。

我是[Pug](https://pugjs.org/api/getting-started.html)模板引擎的忠实粉丝，所以`melody`主题我用的是`Pug`。所以你可能需要了解和学习pug模板引擎的语法和入门使用。

### CSS预处理

跟模板引擎一样，CSS预处理能够更方便地写CSS，复用代码、函数等功能是十分方便的。我是[Stylus](http://stylus-lang.com/)的忠实粉丝，所以`melody`主题我用的是`Stylus`。所以你可能需要学习stylus的语法和入门使用。

### Hexo的变量以及辅助函数

Hexo的官方文档是出了名的烂。不过有两个用得比较多的部分，一个是Hexo会植入模板引擎的[变量](https://hexo.io/docs/variables.html)，以及在很多地方都用得到的[辅助函数](https://hexo.io/docs/helpers.html)。这些通常是用来进行配置不同页面显示不同的内容。

## 搭建主题脚手架

### 页面

通常来说一个Hexo主题需要包括如下几个页面：

1. 首页 `Home`
2. 归档页 `Archive`
3. 标签页 `Tag`
4. 分类页 `Category`
5. 文章页 `Post`
6. 页面详情页 `Page`

这些页面文件都要放到`layout`目录里。构建的时候将会读取里面的内容进行编译。

### 资源

像`CSS`，`JS`，`IMG`这些都可以算作是资源文件，构建的时候作为引用的资源。这些都放到`source`目录里。

### 搭建书写主题的舒适环境

首先需要搭建一个Hexo的博客环境。去[Hexo](https://hexo.io)的官网，按提示安装`hexo-cli`，然后在本地创建一个Hexo博客的目录。它会预先置入一个默认主题`landspace`以及一篇默认的`hello world`文章。当然我们不是在`landspace`上修改，我们需要自己撰写。

主题在写的时候我们需要实时看到效果，而不是写完重新构建一遍才能看到效果，所以需要借助`hexo-server`和`hexo-browsersync`的帮助。前者能够开启一个小型服务器，自动构建来展示hexo博客的页面。而后者能够在你修改了主题文件的时候自动帮你刷新浏览器，帮你省去刷新的动作。所以我们需要安装一下。

另外我们的主题需要`pug`和`stylus`的渲染引擎，所以如下一并安装了：

`npm install hexo-server hexo-browsersync hexo-renderer-jade hexo-renderer-stylus --save-dev`

> 注意，新版的`hexo-renderer-jade`已经包括了处理`pug`的渲染引擎。

#### Bug以及解决

1\. 安装了`hexo-browsersync`之后也不能实现修改`pug`文件之后刷新出修改后的结果。只能实现自动刷新，但是刷新了之后还是修改前的页面。所以我找了一种办法使其能够达到预期的刷新并修改的效果，可以参考这个[issue](https://github.com/hexojs/hexo-renderer-jade/issues/6)里最下面我的回答：

在你的`node_modules`文件夹里找到`hexo-renderer-jade`的文件夹，然后将里面`lib/pug.js`（jade同理）的其中一行代码注释掉：

```js
// pugRender.compile = pugCompile
```

我初步看了一下应该是跟预编译有关系。

2\. `hexo-server`模式下，中文文章渲染不全。可以参考这个相关[issue](https://github.com/hexojs/hexo-server/issues/23)。解决办法是在站点的（而不是主题的）`_config.yml`里添加如下配置：

```yml
server:
  compress: true # 开启压缩
```

3\. `hexo`默认的`highlight`渲染在未指定代码类型的时候会很慢，为了规范我们的文章书写以及提高渲染速度，我们应该在站点的配置文件里加上：

```
highlight:
 auto_detect: false
```

并在写文章的时候，代码块的声明区域边上直接带上代码类型，比如：\`\`\`js，这样就正常了。

### 用Yeoman来生成主题结构

生成`Hexo`主题的话，用`Yeoman`是很方便的。如果系统里没有首先先安装一下`npm install yo -g`，然后再安装一下`npm install generator-hexo-theme -g`。（注意全局安装可能需要权限）于是我们就拥有了一个可以生成主题目录结构的脚手架工具。

进入之前创建好的Hexo的博客目录，找到`themes`文件夹，进入。然后`yo hexo-theme`，这样就会自动生成对应选项，根据选项我们选择`pug`和`stylus`，给这个主题命名为`temp`。然后就会生成一个还不错的项目目录结构。

```
.
├── _config.yml # 主题配置文件
├── layout # 布局文件夹
│   ├── archive.pug # 归档页
│   ├── category.pug # 分类页
│   ├── includes # 复用的公共页
│   │   ├── layout.pug # 页面布局
│   │   ├── pagination.pug # 翻页模板
│   │   └── recent-posts.pug # 文章列表模板
│   ├── index.pug # 主页
│   ├── page.pug # 页面详情页
│   ├── post.pug # 文章详情页
│   └── tag.pug # 标签页
└── source # 资源文件夹
    ├── css # CSS
    │   └── temp.styl
    ├── favicon.ico # 站点图标
    └── js # JS
        └── temp.js
```

如果完全依照它给的结构来写东西显示是不够的。我们需要一点点给它加东西进去。

先找到站点的`_config.yml`文件，找到`theme: landsacpe`的字样，把它改成`theme: temp`。（改成的主题名字得根据你在`themes`目录下创建的主题的文件夹名字，本教程为`temp`）

然后我们用`hexo s`启动服务器，此时应该就能够看到效果了——嗯，一个完全没有CSS样式的页面。确实，因为这个脚手架并没有帮我们写好任何JS和CSS。除了帮我们建好一些模板之外，剩余的完全就靠我们自己来写了。

> 对于`hexo s`而言，这个是`hexo server`的缩写。它的默认端口应该是3000，如果跟你本机的一些端口冲突的话我们可以考虑采用指定端口。比如`hexo server -p 3111`就能指定在3111端口开启服务。还可以加个`-o`的参数让它自动帮我们打开浏览器。当然每次写主题的时候都要这样输这么多字肯定不舒服，所以我们可以把命令写入`npm scripts`

```json
scripts: {
  "dev": "hexo s -p 3111 -o"
}
```

这样以后我们只需要`npm run dev`就行了。

## 主题配置文件

我们很经常会跟主题的配置文件打交道，因为有些功能的开启与关闭我们可以在主题的`_config.yml`实现。配置文件采用的是[Yaml](http://docs.ansible.com/ansible/latest/YAMLSyntax.html)的语法。它跟json其实很类似，但是写法上简单一些，少去了引号逗号花括号等等，取而代之的是缩进以及`-`前缀符号。

之所以把主题配置文件提到这么前面来讲，主要是我踩了一些坑。涉及到主题开发和之后发布上的问题。所以要在这里说一下。往常我们用hexo的主题的话，一些配置项通常是直接修改`_config.yml`文件，然后再编译一遍站点就可以看到效果。这样做的话优点是方便省心，一次配置了之后就不用管了。

然而这样做缺点也很明显：

1\. 对于使用者来说，**更新主题的话难免需要把之前的配置文件不管是复制还是剪切，都要找个新的地方先暂时存放，然后再和更新后的主题的配置文件合并**。对于一个不断更新的主题而言，如果你想要用上一些新的功能难免要不断升级。那么升级的话就会遇到配置文件频繁挪动的困扰。

2\. 对于开发者来说，如果单纯把配置项全部写在主题的`_config.yml`里，那么在发布的时候，因为涉及到一些隐私或者是一些功能默认不需要打开的话，就需要先把`_config.yml`里的东西先整理成一份干净的配置文件，不能有自己的诸如`disqus`的id这样的内容在里面。对于开发者而言这也是很痛苦的。每次要发布主题就需要把自己的`_config.yml`清空一份，然后发布。然后开发主题的时候又要恢复回来。

### data files

为了解决这个问题，我找了不少资料。后来发现了Hexo3.0之后自带支持的这个功能：[data files](https://hexo.io/docs/data-files.html)，能够实现我的需求。并且在参阅`Next`主题对于这个特性的一些实现之后，发现这个确实是可行的。

简单来说，就是你可以通过在站点的`source/_data`目录下（`_data`文件夹不存在的话手动创建一个）新建配置文件，比如`temp.yml`，然后在主题里可以通过`site.data.temp.xxx`去访问配置文件里的配置。

但是这样对于写主题而言还是不太方便。因为有的用户并不需要频繁更新主题，他只需要修改主题的`_config.yml`就好了。那么我们在模板引擎里引用主题配置文件的内容是用`theme.xxx`来访问的。如果两种状态都要考虑的话，我们可能需要在写任何一个配置的时候都要判断一下主题的`_config.yml`里或者_data里的`temp.yml`存不存在。这样很麻烦。

参考了`Next`主题对于这个的实现后，`melody`对于这个问题的实现如下：

参考hexo渲染的[事件](https://hexo.io/api/events.html)，可以找到`generateBefore`这个钩子，只要在这个钩子触发的时候，判断一下存不存在`data files`里的配置文件，存在的话就把这个配置文件替换或者合并主题本身的配置文件。`Next`主题采用的是覆盖，`melody`主题采用的是替换。各有各的好处，并不是绝对的。

写法是就是在我们的`temp`主题目录下的`scripts`文件夹里（没有就创建一个），写一个js文件，内容如下：

```js
/**
 * Note: configs in _data/temp.yml will replace configs in hexo.theme.config.
 */
hexo.on('generateBefore', function () {
  if (hexo.locals.get) {
    var data = hexo.locals.get('data') // 获取_data文件夹下的内容
    data && data.temp && (hexo.theme.config = data.temp) // 如果temp.yml 存在，就把内容替换掉主题的config
  }
})
```

这样，hexo在构建的时候，会去`scripts`文件夹里执行里面的代码并运用在渲染中。

### 平滑升级

有了上面的步骤之后，作为用户要更新主题，如果一开始是用`git clone`的方式克隆的话，只需要在目录里`git pull`就行了。而作为开发者，可以放心的在自己的`data files`里定义的配置文件里写下自己的一些配置项的参数，比如一些不适合暴露出去的id等，然后在主题里的`_config.yml`写上对应的空配置项即可。

> **注意，这个方法的缺陷是，每当修改了`data files`里的配置，需要重新运行`hexo s`或者`hexo g`才能看到效果**

## 页面书写

动手开始写主题的页面开始，需要注意到`layout.pug`这个文件，这个文件是整个网站布局的最核心的基础代码——连index页面都是通过`extends`它而来：

```pug
extends includes/layout.pug

block content
  include includes/recent-posts.pug
  include includes/pagination.pug
```

所以它决定了我们整个网站的布局。打开`layout.pug`可以看到，它暴露一个`block content`给我们去根据不同的页面来写不同的内容。这个是整个Hexo主题的主入口。根据不同的页面类型，渲染不同的页面内容。不同的页面，书写代码的思路其实说到底跟上面的方式差不多了：

```pug
extends includes/layout.pug // 首先继承layout模板

block content
  include includes/recent-posts.pug // block content 区域引入rencent-posts模板
  include includes/pagination.pug // block content 区域引入pagination模板
```

可以发现，这样做的好处是最大化复用了`layout`的模板，让所有页面都一致或者相似的地方能够少些不少代码。

### 数据获取

页面数据的获取，一般有三种途径：

1. Hexo预置（[变量](https://hexo.io/docs/variables.html)、[辅助函数](https://hexo.io/docs/helpers.html)）
2. 站点的配置文件（_config.yml）
3. 主题的配置文件（temp.yml或者_config.yml）

下面详细说一下如何使用。

#### Hexo预置

Hexo预置的变量，查看文档后你能发现主要有`site`、`page`、`post`和`theme`（会在主题的配置文件里说）。

1.`site`变量可以说是个对象，长得如下：

```js

site = {
  posts: [object object], // 包含文章对象的数组
  pages: [object object], // 包含页面对象的数组
  categories: [object object], // 包含分类对象的数组
  tags: [object object] // 包含标签对象的数组
}

```
通常，我们可以通过`site`变量获取站点的文章总数，标签总数和分类总数

2.`page`是一个很神奇的变量，也是最有用的变量。不同的页面，从page拿到的文章不一样。

比如首页，如果你在站点的配置文件里配置了一页显示多少文章的话（比如10篇文章），那么在首页，`page`里就只会有这10篇文章的相关信息，而不是你的博客里所有的文章的相关信息。如果在标签页，你通过`page`拿到的值将只会是某个标签对应的所有文章的相关信息（当然也受制于一页显示多少文章）

所以`page`将会是很常用的一个变量。

3.`post`变量实际上是某一篇文章的具体信息。它和`page`变量里的某一项差别就在于，`post`变量还多包含了这一篇文章的`tags`、`categories`和`published`变量。前两者很常用，它们通常就是用来显示某一篇文章所具有的标签以及所处的分类。

4.`theme`这个变量实际上就是主题的配置文件的对象。这个我们将放到后面再讲。

Hexo预置的辅助函数，常用的我觉得有如下几个：

1. `is\_home`、`is\_post`、`is\_tag`、`is\_category`、`is\_post`等几个判断当前文章类型的函数
2. `date`、`time`等时间字符串处理函数
3. `list\_categories`、`list\_tags`、`tagcloud`等生成分类列表、标签列表、标签云等函数
4. `toc` 生成文章目录
5. `paginator` 生成页面页码

#### 站点配置文件和主题配置文件

站点配置文件和主题配置文件在hexo页面里能够暴露出来的配置项分别是`config`和`theme`。而且相当简单就能获取。

比如我在主题的`temp.yml`配置文件里有配置如下：

```yaml
menu:
  Home: /
  Archives: /archives
  Tags: /tags
  Categories: /categories
```

那么我在模板引擎里就可以通过`menu.Tags`的方式获取`/tags`这个字符串

### 数据查看与填充

在pug里如果你想要查看一个变量的值（比如site的pages），你可以直接这样做：

```pug
- console.log(site.pages)
```

注意到，如果要在`pug`里书写变量声明，表达式，可以有如下写法：

**单行书写**

```pug
- console.log(1)
- var a = 2
```

通过短横线`-`后加个空格，然后书写单行表达式。如果要多行书写要用如下写法：

**多行书写**

```pug
-
  console.log(1)
  var a = 2
```

通过短横线`-`下方代码区缩进来书写多行表达式。**需要注意的是，此时`-`号后面不能跟空格，直接回车。**

那么你就可以在控制台看到具体的pages的信息。这并不会渲染到最终的html里。既然拿到了数据，那我们就可以开始渲染了。

比如对于`recent-posts`，实际上就是我们常见的首页能看到一堆文章列表的页面。要对这个页面进行渲染，可以采用如下写法：

```pug
each article in page.posts.data //- 注意首页也是一个page，它包含的posts只包括了首页会显示出来的文章。
  .recent-post-item
    - var link = article.link || article.path
      a.article-title(href=url_for(link))= article.title || 'no_title'
    .content!= article.excerpt //- excerpt是post的一个变量，只会显示文章的摘要部分（也就是文章里<!--more-->之前的内容）
    a.more(href=url_for(link) + '#more') 阅读更多
    hr
```

注意到上面的a标签里，我们用到了一个`url_for(link)`，`url_for()`是个辅助函数，负责将相对路径转换为页面的绝对地址，这样在哪里都能点开而不用考虑相对路径的问题了。

注意到pug里，如果要输出变量，有两种写法`=`和`!=`。这两者的区别就是前者的输出带转义，后者的输出不带转义。可以查看pug的官方文档对于这二者区别的[解释](https://pugjs.org/zh-cn/language/code.html#buffered-code)。简单来说后者的输出可以输出标准的html标签。而前者将会把html标签转义成字符串。

#### 页码的实现

我们能够发现通常博客底部是有页码的，告诉读者有多少页的文章，并且当前在第几页。这个功能在Hexo里实现起来特别容易。就是运用`paginator`这个辅助函数。

找到脚手架生成的`pagination.pug`文件，发现它的实现还蛮复杂。我们将它改写一下：

```pug
-
  var options = {
    prev_text: '<',
    next_text: '>'
  }
#pagination
  .pagination
    !=paginator(options)

```

那么它就会按照页面的不同生成不同的页码。不过有个例外，在文章页，由于没有分页，我们通常需要实现上一篇文章和下一篇文章的链接按钮供读者快速切换邻近的文章。所以改写如下：

```pug
-
  var options = {
    prev_text: '<',
    next_text: '>'
  }
#pagination
  if(!is_post())
    .pagination
      !=paginator(options)
  else //- 
    if(page.prev)
      .prev-post.pull-left
        a(href=url_for(page.prev.path))
          i.fa.fa-chevron-left   
          span=page.prev.title
    if(page.next)
      .next-post.pull-right
        a(href=url_for(page.next.path))
          span=page.next.title 
          i.fa.fa-chevron-right

```

#### 侧边栏目录的实现

侧边栏目录的实现其实很简单。通过`toc`这个辅助函数，其实很容易就能做到。如果不需要额外配置，最简单的话如下就可以实现：

```pug
if(is_post()) //- 如果是文章页面
  .sidebar-toc!= toc(page.content) //- 给toc传入当前页面的内容用于生成目录结构
```

它大致生成如下的HTML结构：

```html
<ol class="toc">
  <li class="toc-item toc-level-2">
    <a class="toc-link" href="#hexo">
      <span class="toc-number">1.</span>
      <span class="toc-text">hexo</span>
    </a>
  </li>
</ol>
```

然后你可以用js去控制它是否可以展开，是否可以跟随页面滚动而高亮等等，以及用css去修改它的默认样式。

如果还要定义目录的层级、是否显示前缀数字，可以根据官网的文档，在`toc`里传入额外的参数来实现。

#### 标签云页+分类归纳页

标签云页，也就是我们常见的一个能够展现所有标签集合的页面：

![](https://ws1.sinaimg.cn/large/8700af19ly1fjib2khkdoj21je0y20x0.jpg)

分类归纳页，也就是展示所有`categories`层级关系的页面：

![](https://ws1.sinaimg.cn/large/8700af19ly1fjib7uhzo0j21jc0rydie.jpg)

一开始我以为是在`tag.pug`页面书写这个这个标签云的。后来我发现错了。这个标签云的页面实际上是在`page.pug`页面，通过判断页面类型来进行不同输出的。而分类归纳页也是同理：

```pug
extends includes/layout.pug

block content
  if page.type === 'tags'
    .tag-cloud
      .tag-cloud__title= page.title  
        |  - 
        span.tag-cloud__amount= site.tags.length
      .tag-cloud-tags!= tagcloud({min_font: 12, max_font: 30, amount: 200, color: true, start_color: '#A4D8FA', end_color: '#0790E8'})
  else if page.type === 'categories'
    .category-lists
      .category__title= page.title  
        |  - 
        span.category__amount= site.categories.length
      div!= list_categories()
  else
    article#page
      h1= page.title
      != page.content
    include includes/pagination.pug
```

其中，标签云的生成依赖于辅助函数`tagcloud`，相关配置项也可以在官网找到说明。而分类归纳列表，则通过辅助函数`list_categories`生成。至于样式，当然是根据自己的主题风格自己书写CSS来适配了。

#### 详情页+文章归档页

例外的是，文章归档页（archives）不适用于上面的规则，而是直接在`archive.pug`里直接书写相应的页面代码就好了。

而实际上，你会发现，从标签云点击某个标签页，以及从分类归纳页点击某个分类后，进入的页面，跟文章归档页其实是很类似的：

1\.具体的标签页

![](https://ws1.sinaimg.cn/large/8700af19ly1fjibyqrm5gj21z211kjxf.jpg)

2\.具体的分类页

![](https://ws1.sinaimg.cn/large/8700af19ly1fjibyqq731j21z212244o.jpg)

3\.文章归档页

![](https://ws1.sinaimg.cn/large/8700af19ly1fjibyr0atdj21z211e447.jpg)

可以发现，基本上除了标题有所不同之外，基本都是文章列表的形式。如果没有特殊要求，实际上就跟做首页一样，统一用`each article in page.posts.data`，然后把文章标题和标题地址拿出来就好了。然而由于我对文章列表的形式有所要求和定制，所以我写了一个`mixin`，用于处理文章列表。

```pug
mixin articleSort(posts)
  .article-sort
    - var year
    - posts.each(function (article) {
      - var tempYear = date(article.date, 'YYYY')
      if tempYear !== year
        - year = tempYear 
        .article-sort-item.year= year //- 展示年份
      .article-sort-item
        time.article-sort-item__time= date(article.date)
        a.article-sort-item__title(href=url_for(article.path) target="_blank")= article.title || 'No Title'
    - })
```

然后比方说在`tag.pug`页面，就可以用如下的写法来实现：

```pug
extends includes/layout.pug

block content
  include ./includes/mixins/article-sort.pug //- 引入mixin
  #tag
    .article-sort-title= 'Tag - ' + page.tag
    +articleSort(page.posts) //- 调用mixin
  include includes/pagination.pug
```

### Stylus一些技巧

在`pug`里可以通过诸如`div=theme.xxx`的方式来获取主题配置文件里的`xxx`项的值。而`stylus`里要获取主题配置里的变量要怎么做呢？最普遍的需求就是，代码高亮配色方案的选择，比如`melody`主题提供了`material-theme`四种配色方案，要通过配置文件的配置项来配置采用何种配色方案。

比如：

```yaml
highlight: default
```

这个时候在stylus里可以通过比如：

```styl
$highlight-theme = hexo-config('highlight')
```
用`hexo-confg`这个方法，来获取当前在主题配置文件里的配置项。

然后运用`stylus`的条件语句就可以渲染不同的主题配色方案了。

另外，我们在使用stylus的时候，通常会书写一些变量文件stylus，这些变量文件可以运用到其他需要编译成css的stylus，而变量文件本身并不需要编译成具体的CSS文件。如何控制这些stylus文件不编译成具体的css呢？

例如`melody`主题的CSS文件夹结构如下：

```
.
├── _global
│   └── index.styl
├── _highlight
│   ├── diff.styl
│   ├── highlight.styl
│   └── theme.styl
├── _layout
│   ├── comments.styl
│   ├── footer.styl
│   ├── head.styl
│   ├── page.styl
│   ├── pagination.styl
│   ├── post.styl
│   └── sidebar.styl
├── _search
│   ├── algolia.styl
│   └── index.styl
├── _third-party
│   ├── jquery.fancybox.min.css
│   └── normalize.min.css
├── index.styl
└── var.styl
```

你会发现，里面的文件夹都是以下划线`_`作为起始的。我在`index.styl`里引用了它们：

```styl
@import "nib"
// third-party
@import "_third-party/jquery.fancybox.min.css"
@import "_third-party/normalize.min.css"
// project
@import "var"
@import "_global"
@import "_highlight/highlight"
@import "_layout/*"

// search
if hexo-config("algolia_search.enable")
  @import "_search/index"
  @import "_search/algolia"
```

实际上，以下划线作为文件夹命名起始的话，hexo在编译的时候就会略过他们不生成具体的css——也就是说最后只会生成`index.css`和`var.css`两个css文件，而不会生成诸如`_layout/footer.css`这样的文件了。

当然如果你需要做到生成`footer.css`这样的css文件，就把它所在文件夹的名字去掉前面的下划线就好啦。

### pug的一些使用技巧

通常我们主题或者站点的一些配置文件的配置项，根据需要可以由pug渲染成不同的html。但是难免我们需要比如js里获取来自主题配置或者站点配置的一些配置项。

一个比较鲜明的例子就是[algolia](https://www.algolia.com/)这个搜索框架需要我们在js里提供诸如`appId`等信息。这个时候我们需要在`pug`里写script，把来自主题或者站点配置的内容写到`script`标签里。

```pug
-
  var algolia;
  if (theme.algolia_search.enable) {
    algolia = JSON.stringify({
      appId: config.algolia.appId || config.algolia.applicationID,
      apiKey: config.algolia.apiKey,
      indexName: config.algolia.indexName,
    })
  } else {
    algolia = 'undefined'
  }
script.
  var GLOBAL = { 
    algolia: !{algolia},
  } 
```

上面的pug最后会输出成html如下：

```html
<script>
  var GLOBAL = {
    algolia: {
      appId: 'xxxxx',
      apiKey: 'xxxxx',
      indexName: 'xxxx'
    }
  }
</script>
```

注意到上面我们用到了`!{algolia}`的方式，把变量输入到了`pug`的script标签里。我们不能简单像在pug的html部分通过诸如`div= theme.algolia_search.enable`这样的方式把内容渲染到`script`标签里，因为script标签里的就是实打实最后要输出到html里的js内容。不过可以通过`!{}`的方式，pug在编译的时候会把这部分变量的内容填充到script标签里。

同时需要注意的两点是：

1. 要在`script`标签里写js代码的话，需要在`script`后面加个点：`script.`
2. 如果只是单纯填充某个变量，用`!{xxx}`就行了。但是如果该变量是个对象的话，则需要通过`JSON.stringify`先字符串化。具体可以看上面的代码。

## 主题发布

根据自己的奇思妙想，运用上面的各种方法去实现你自己的hexo的主题。而主题写完就可以发布了。首先先去fork Hexo的官方[站点](https://github.com/hexojs/site)，然后编辑`source/_data/theme.yml`文件，把你的主题信息加上去：

```yaml
- name: Melody
  description: A simple & beautiful & fast theme for Hexo # 简介
  link: https://github.com/Molunerfinn/hexo-theme-melody # 示范站点
  preview: https://molunerfinn.com # github 仓库
  tags: # 标签
    - simple
    - beautiful
    - fast
```

然后去`source/theme/screenshots/`文件夹里放置一个跟你主题名字一样的图片，大小是800*500。

弄完这些就把这些提交到自己fork的仓库里。然后，向官方站点发起一个`pull request`，静候Hexo官方维护人员把你的主题收录，就大功告成啦！

我的主题[hexo-theme-melody](https://github.com/Molunerfinn/hexo-theme-melody)就已经成功被官方收录啦：

![](https://ws1.sinaimg.cn/large/8700af19ly1fjjgshdhkwj20p20suwqd.jpg)

------

如果你喜欢我的主题，欢迎使用，star，pull request，也可以在issue里给我提意见或者建议。感谢阅读了我这篇啰嗦的文章的所有读者，希望对大家的Hexo主题开发能有所启迪~

