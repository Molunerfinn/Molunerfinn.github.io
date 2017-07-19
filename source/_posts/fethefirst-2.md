title: 前端-给自己的第一个博客添加样式
id: 310
categories:
  - Web
  - 开发
date: 2015-08-12 14:02:01
tags:
---

上一篇文章已经给出了一个两栏布局的博客的基本结构。我们实现了两栏布局的基本页面结构([DEMO](http://molunerfinn.com/display/MYBLOG/index-demo.html "http://molunerfinn.com/display/MYBLOG/index-demo.html"))，今天我们来给这个DEMO添加样式，让它变得更好看。

更新后的地址在这里 [NEW-DEMO](http://molunerfinn.com/display/MYBLOG/index.html "http://molunerfinn.com/display/MYBLOG/index.html")
<!--more-->
### 顶部栏背景色以及文字颜色设置

顶部栏的DIV我们给了一个class叫做`header`，于是在CSS样式中，我们对这个`header`进行样式设置。

```
    .header{
    margin:0px auto 10px; //设置header的外边距
    color: #fff;    //设置字体颜色
    background:rgba(55,130,231,0.8); //设置背景颜色
    border: 1px solid #3782E7; //设置边界颜色
    text-align: center; //使文本居中 
    }
```

### 左侧栏样式

左侧栏是以left为class的一个大DIV包容进来的。然后文章列表用的是这个大DIV下的子DIV，class叫做title-head。接下来来实现title-head的样式。

```
    .left{
        width: 450px;
        height: 500px;
        margin: 0 0 0 40px;
        float: left;
    }
    .title-head h3{
    padding-left: 15px; //内边距15px
    margin-left: 10px; //外边距10px
    display: block; //将h3这个行内元素转换为块级元素
    border-left: 3px solid #3782E7; //左侧蓝色的提示线
    }
```

### 右侧栏样式

右侧栏是以right为class的一个大DIV包容进来的。文章标题输入框与文章内容输入框分别用了article与textin两个class名。然后这个主要就是设定一下这两个框的长宽以及右下角提交按钮的位置设定。

```
    .right{
    height:500px; 
    width: 600px;
    margin: 0 40px; //上下外边距0，左右外边距40px；
    float: right; //向右侧父容器边界浮动
    }
    .article{
    width:500px;
    margin:20px 50px 10px 50px; //上、右、下、左边距
    }
    .textin{
    width:500px;
    height: 350px;
    margin:10px 50px;
    }
    .up{
    float:right;
    margin-right: 50px;
    }
```

这些主要的样式就实现了我们一个比较美观的两栏博客页面。主要要注意的地方就是margin、padding、float等这几个属性的用法，要明确什么是外边距，什么是内边距，什么是浮动，然后怎么样排版才会比较好看。当然我们做的毕竟只是前端，并没有数据存储的部分，那是后端要考虑的问题。所以这个页面也只是一个“好看”但“没用”的页面。不过从这个最基本的页面，我们能从中学到的东西，毕竟还是不少的。

### 总结

总结一下，从前端入门的话，想要动手写点东西，那么从一个博客页面开始吧。你可以做得很美观，也可能做得比较丑。但是这都不重要。重要的是你在这之中学到的东西，以及对美的追求是从前端开始变得格外重要的。程序员一向被认定为是没有美感的、没有人文素养的一批人。但是至少作为一个前端工程师，这些都应该会是你所拥有或者说你会着重培养的资质。应该为自己感到自豪。
