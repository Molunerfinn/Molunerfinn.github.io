title: jQuery——删除 动态添加的元素的方法
tags:
  - 前端
id: 318
categories:
  - Web
  - 开发
date: 2015-09-04 12:47:51
---

这两天在写JS，写一个遮罩层的时候发现了一个挺有意思的地方，就是直接运用jQ的`$`选择器的时候无法选择出动态创建的元素，因此也就无法做到删除这个动态添加的元素。然后找寻了各种方法，目前找到两种我觉得是挺好的解决办法。

* * *
<!--more-->
先看看我最初写的源代码

HTML：只对一个DIV操作。查找按钮是用来做触发事件的。ID为`mask`的DIV是拿来当做遮罩层的。

```
<form class="search">
    <input type="text" class="input-text">
    <input type="button" class="input-button" value="查找">
</form>
<!-- <div id="mask"></div> -->
```
CSS：

```
#mask{
    position: absolute;
    z-index: 100!important;
    left: 0;
    top: 0;
    height: 1000px;
    width: 100%;
    background-color: #000;
    opacity: 0.75;
    filter: alpha(opacity = 75);
}
```

JS:

```
$(document).ready(function () {
    var _mask=$("<div id='mask'></div>");
    // 点击按钮创建遮罩层
    $(".input-button").click(function(){
        $(_mask).appendTo("body");      
    });
    // 点击遮罩层时删除遮罩层
    $("#mask").click(function(){
        $(this).remove();
    })
});
```

以上代码看似能够实现我们的功能，但是实际上，当你点击遮罩层的时候，并不会删除遮罩层。因为遮罩层是我们点击查找按钮的时候动态创建的，所以直接靠`$("#mask")`是无法选择出这种动态创建的元素的，自然也就没有作用了。

* * *

## 第一种办法
- 函数封装

我们可以将`$("#mask")`这个选择器和动态创建元素的方法封装到一个函数中，然后在创建遮罩层的功能中引用这个函数，就能实现JQ识别出这个动态创建的元素。

代码如下-JS：

```
// 将方法封装进一个叫做mask的函数中
function mask(){
    var _mask=$("<div id='mask'></div>");
    $(_mask).appendTo("body");
    $("#mask").click(function(){
        $(this).remove();
})
};

// 点击按钮创建遮罩层时调用这个函数
$(".input-button").click(function(){
    mask();
    return false;
});
```

* * *

## 第二种办法

*   使用jQuery的on()方法

jQuery在1.9版本之后取消了live()方法，所以对于原先用live()实现的方法现在改用on()方法。PS:同时on()方法还能够取代bind()、delegate()方法，也是官方推荐的一种方法。

先说明一下live()方法。引用W3School的定义：

> live() 方法为被选元素附加一个或多个事件处理程序，并规定当这些事件发生时运行的函数。
>   通过 live() 方法附加的事件处理程序适用于匹配选择器的当前及未来的元素（比如由脚本创建的新元素）。

注意第二句，

> 适用于匹配选择器的当前及未来的元素（比如由脚本创建的新元素）。

因此live()方法可以用于选择动态创建的元素。在使用1.9以后版本的jQuery时，用on()方法来代替。

on()的用法：`$(selector).on(event,childSelector,data,function,map)`

当其他选项没有参数时，我们可以忽略它们。这里我们关注三个选项。`event`：事件，`childSelector`：子选择器，`function`：功能。
让on()产生live()相同的功能时，`$(selector)`里的selector要写成document,也就是绑定整个页面元素。这点很关键，通过页面元素去选择`childSelector`，即当页面元素的子元素有变化时，该方法能够实时选择出你想要的那个子元素，也就实现了动态选择。而`event`我们这里是点击，也就是click。至于function，看如下代码：

```
var _mask=$("<div id='mask'></div>");
// 点击按钮创建遮罩层
$(".input-button").click(function(){
    $(_mask).appendTo("body");
});
// 点击遮罩层时删除遮罩层，注意采用了on()方法。
$(document).on("click","#mask",function(){
    $(this).remove();
});
```

* * *

在写的时候我貌似发现原生JS由于获取元素id就是通过document.getElementById()的方法也就是直接从document里获取，貌似也不会出现选择不了动态创建元素的情况。这点以后仔细研究后再更新吧。
