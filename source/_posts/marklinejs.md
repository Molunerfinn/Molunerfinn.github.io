---
title: markline.js——轻量级canvas绘制标记线的库
tags: 
  - JS
categories:
  - Web
  - 开发
date: 2016-11-5 23:23:00
---

这段时间要做的是一个数据可视化的小型项目。其中最基本要求是实现两点之间的迁徙关系（比如同一个用户不同时间上网的地点）用一条有向线段（markline）联系在一起。很自然的我一开始想的就是采用百度的[echarts](http://echarts.baidu.com/echarts2/doc/example/map21.html)里的一个地图工具实现这个方案，并且百度给出的方案里默认的样式已经很漂亮了。  
But，除了基本要求之外，还有一些深层次的要求：1.用户自定义背景图案，2.鼠标触摸上markline时，显示的内容可以自定义等等。  
于是我发现了echarts不支持自定义比如jpg的背景图案——就这点就必须让我放弃使用echarts了。于是我必须考虑自己实现一下echarts里markline的样式了。

<p align="center">
  <a href="https://github.com/Molunerfinn/markline.js">
    <img width="300px" height="300px" src="https://raw.githubusercontent.com/Molunerfinn/markline.js/master/img/markline.gif">
  </a>
<p>

<!--more-->

## 面临的问题

第一次自己手动实现一个canvas的小型效果库，有点紧张。由于之前做过一个热图的项目，用了[jCanvas](http://projects.calebevans.me/jcanvas/)这个依赖于jquery的，所以想着这次能不能继续用用这个库。

主要要实现几个东西：

1. 画曲线而不是简单的画直线。并且A->B和B->A应该是不能重合的两条曲线
2. 线上应该要有类似ehcarts的标记小球在运动，指示线的方向，并且效果要好，不能直接拿个小圆球死板的运动
3. 不光有标记线，还要有标记点
4. 标记线和标记点的hover提示信息应该能够自定义

那么重点来了。

画曲线不难，我们有二阶贝塞尔曲线可以拿来画曲线。

运动的小球，貌似也不太难——但是要做出小球身后的扫尾效果不容易——一开始我是没有思路的。

hover信息自定义的话，就涉及到判断鼠标的位置在不在所画的元素上了——也即坐标计算。

### jCanvas最后并没有采用

其实如果没有第二点要求的话，用jCanvas来画的话就蛮快的了。只需要画出所需要的线，加个mouseover、mousein、mouseout事件就行了。但是整个项目最难的地方在于：让线上的小球动起来、并且运动要有扫尾。

并且，让小球沿着二阶贝塞尔曲线运动的方法，我一开始也没有很好的思路。

由于我在尝试使用画小球扫尾的时候，jCanvas自定义的layer会导致扫尾失效。无奈，我只能放弃采用jCanvas而采用自己手动书写的方式了。

## 手写markline.js

在放弃了jCanvas的支持后我就在想，能否自己写一个专门的库来做这种效果呢——一开始还在想要不要用上jquery，后来再想想，用上了jquery好像也就是用用dom选择器而已。那还是用原生的js写个独立的库吧。

### 二阶贝塞尔曲线

首先二阶贝塞尔曲线的画线没有难度，（听说过不了解或者没听说过的可以参考张鑫旭前辈对于[贝塞尔曲线](http://www.zhangxinxu.com/wordpress/2014/06/deep-understand-svg-path-bezier-curves-command/)的解释。）我们只需要确定始末点坐标以及控制点坐标即可。

那么控制点坐标应该如何确定？为了画出的曲线是对称的曲线，我的想法很简单，取始末两点线上的垂直平分线上的某点作为控制点即可。那这个某点该如何确认？从A->B和从B->A应该是两条不重合的线，那么两条线控制点就应该分别在始末点的两侧。我用一种比较原始的方法来确定控制点的位置：  

设：始末点连线`Line`的长度是`L`，始末点连线的中点是`P`，始末点连线的垂直平分线是`VLine`。

那么控制点就是在`VLine`上距离`P`点`0.2L`长度的地方。

至于控制点在`Line`的左侧还是`Line`的右侧，需要仔细研究一下：否则出现在同一侧的话，A->B和B->A的线不就重合了么。

#### 数学运算时间

为了计算控制点的坐标，我写了一个函数：

```js
/**
 * 计算二阶贝塞尔曲线的控制点
 * @param  sx     起点x坐标
 * @param  sy     起点y坐标
 * @param  dx     终点x坐标
 * @param  dy     终点y坐标
 * @return point  控制点坐标 
 */
function calControlPoint(sx,sy,dx,dy){
  var a,x,y,X,Y,len;
  X = (sx + dx) / 2;
  Y = (sy + dy) / 2;
  len = 0.2 * Math.sqrt(Math.pow((dy - sy),2) + Math.pow((dx - sx),2)); // 控制贝塞尔曲线曲率
  a = Math.atan2(dy - sy, dx - sx);
  return {x: X - len * Math.sin(a),y: Y + len * Math.cos(a)}
}
```

其中计算`Line`的长度`L`用的是基本的计算两点之间距离公式。`Line`的中点`P`的坐标`(X,Y)`也很简单用两点的X\Y坐标之和的一半可以算出。
而至于控制点在`Line`的哪一侧就靠始末点相对于坐标轴x的夹角了，计算夹角的函数是`Math.atan2()`，可以返回夹角度数（弧度制）。然后再用`Math.sin()`和`Math.cos()`来计算控制点相对于中点`P`的横纵坐标偏移量。

通过这样的方式我们就能够确定从A->B以及从B->A各自的控制点并且不会发生重叠的情况了。

### 箭头方向的确立

由于是有向线段，所以需要在终点处给出线段的箭头。其实画箭头也没什么难度，就是两条相交的线而已。不过确定箭头的方向以及正确画出还需要一些计算。

#### 数学运算时间

首先箭头的方向，就是控制点到终点方向连线的方向，换言之就是先得得到这个角度。

我写了一个用于计算两点之间（A->B）的角度计算的小函数：

```js
/**
 * 计算bezier曲线尾端角度
 * @param  cx   控制点x坐标       
 * @param  cy   控制点y坐标
 * @param  dx   线段终点x坐标
 * @param  dy   线段终点y坐标
 * @return      返回角度
 */
function calcAngle(cx,cy,dx,dy){
  return Math.atan2((dy - cy) , (dx - cx));
}
```

将这个角度应用到下面这个画箭头的函数里：

```js
/**
 * 画箭头
 * @param  ctx    canvas绘画上下文
 * @param  dx     线段终点x坐标
 * @param  dy     线段终点y坐标
 * @param  angle  箭头角度
 * @param  sizeL  箭头长度
 * @param  sizeW  箭头宽度
 */
function drawArrow(ctx,dx,dy,angle,sizeL,sizeW,color){
  var al = sizeL / 2;
  var aw = sizeW / 2;
  ctx.save();
  ctx.translate(dx,dy);
  ctx.rotate(angle);
  ctx.translate(-al,-aw);

  ctx.beginPath();
  ctx.moveTo(0,0);
  ctx.lineTo(al,aw); // 画箭头
  ctx.lineTo(0,sizeW); // 画箭头
  ctx.strokeStyle = color;
  ctx.stroke();

  ctx.restore();
}
```

其中用到的两个比较重要的方法`ctx.save()`和`ctx.restore()`是为了不破坏之前绘图的情况而进行的操作，save是用于保存当前的绘图情况，相当于将绘图的信息入栈，然后在“崭新”的画布上进行绘制，然后用restore方法将之前保存的绘图情况还原，相当于出栈。通过这样的方式就可以绘制不同线的箭头而互相不干扰了。然后做一些相应的旋转变换就能够符合箭头的朝向预期了。

### 运动点运动效果

这个是重头戏。这个库基本上就是围绕这个运动点而生的。

首先，如何让一个点沿着已有的贝塞尔曲线运动呢？

这里可以从这个codepen里参考一二：

<p data-height="311" data-theme-id="0" data-slug-hash="vyBQGN" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-pen-title="vyBQGN" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/vyBQGN/">vyBQGN</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

这里的关键是创建一个svg的二阶贝塞尔曲线的path。（同样还是可以参考张鑫旭前辈的[贝塞尔曲线](http://www.zhangxinxu.com/wordpress/2014/06/deep-understand-svg-path-bezier-curves-command/)）当然这个path可以不插入dom里，而只是创建它。后面要用到这个svg的path的一些方法：而实际上小球是沿着这个svg的path提供的坐标而运动的，而不是沿着canvas所画的贝塞尔曲线运动。

创建一条二阶贝塞尔曲线的svg的path方法不难：

```js
/**
 * svg
 * @param  x1   曲线起点x坐标
 * @param  y1   曲线起点y坐标
 * @param  cx   曲线控制点x坐标
 * @param  cy   曲线控制点y坐标
 * @param  x2   曲线终点x坐标
 * @param  y2   曲线终点y坐标
 */
var path = document.createElementNS('http://www.w3.org/2000/svg','path');
path.setAttribute('d','M' + x1 + ' ' + y1 + ' ' + 'Q' + cx + ' ' + cy + ' ' + x2 + ' ' + y2);
```

创建这条路径的作用在哪里呢？主要是要获取这条路径上的点坐标信息。首先对于SVG的这个path有几个方法我们需要知道：

```js
path.getTotalLength(); // 获取整条path的长度
path.getPointAtLength(length); // 获取path在长度为length处的点的信息
path.getPointAtLength(length).x // 获取path在长度为length处的点的x坐标
path.getPointAtLength(length).y // 获取path在长度为length处的点的y坐标
```

这有什么用呢？简单来说，我们可以获取整条path的长度`len`，然后可以通过计算运动时间百分比`percent`乘上`len`就构成了`path.getPointAtLength(length);`里的`length`。然后就可以通过这个方法来获取x、y坐标了。

换句话说，我只需要把小球运动的过程映射成`percent`就能够得到当前小球应该要出现的位置。

那么如何实现小球的运动呢？

伪代码如下：

```js

var percent = 0; // 声明一个全局变量
var len = path.getTotalLength(); // 获取path的总长

function animation(){
  
  percent >= 100 ? percent = 0 : percent += 0.3;
  var point = path.getPointAtLength(len * (percent / 100));
  X = point.x; // 获取小球的X坐标
  Y = point.y; // 获取小球的Y坐标
  
  Ball.paint(); // 画球 

  window.requestAnimationFrame(animation) // 循环

}

```

如上的伪代码提供了一个思路，通过累加percent来达到小球沿着曲线运动的效果。并且当percent累加到100时重新从0开始计算，就能够实现小球周而复始在线上运动了。

### 运动点拖尾效果

说到这个拖尾效果，就比较复杂了。在我查看了网上很多非直线运动点拖尾效果的例子（比如这个[彗星扫尾](http://wow.techbrood.com/fiddle/8268)）。

总结一下，主要采用的办法基本上有两种：

- 重绘+透明背景色填充——ctx.fillStyle = 透明色
- 重绘+整体透明度填充——ctx.globalAlpha = 透明度

什么意思呢，举个例子：

A时刻，我在画布上画了一个点(x,y)；

B时刻，我在画布上花了第二个点(x + c,y + c)，并进行透明背景色\整体透明度填充。

那么B时刻的画面覆盖了A时刻的画面，但是是加上了透明色\透明度的覆盖，那么就会让A时刻的点颜色变淡。（类似于加上了一层遮罩一样）

C时刻重复之前的做法，D时刻重复之前的做法……以此无穷循环往复，就可以让一个运动点出现拖尾的效果了。

那么markline.js里采用的是哪一种方法呢？

为了能够让用户自定义背景，通过实验我采用了第二种`重绘+整体透明度填充——ctx.globalAlpha = 透明度`的方法。这种方法的好处是，你的背景在重绘的同时还能保持亮度不变，并且`ctx.globalAlpha`比`ctx.fillStyle`更加灵活，可以不参与绘制图形的着色而只是改变整体透明度，因而可以随时在画的时候通过`ctx.save()`和`ctx.restore()`的方式来改变整体透明度，达到合理控制局部绘制的效果。

伪代码如下：

```js

var percent = 0; // 声明一个全局变量
var len = path.getTotalLength(); // 获取path的总长

function animation(){
  
  ctx.globalAlpha = 0.2; // 值是0-1,0代表完全透明，1代表完全不透明

  percent >= 100 ? percent = 0 : percent += 0.3;
  var point = path.getPointAtLength(len * (percent / 100));
  X = point.x; // 获取小球的X坐标
  Y = point.y; // 获取小球的Y坐标
  
  Ball.paint(); // 画球 

  window.requestAnimationFrame(animation) // 循环

}

```

那么每次在绘图的同时就会加上一层透明度，盖住上一次绘图的痕迹，也就能够实现拖尾的效果了。

### 背景

重绘的时候必须要有背景跟着重绘，否则会出现点、线重叠渲染的情况。并且还需要借助一个属性：`ctx.globalCompositeOperation`（可以参考这篇[文章](http://www.cnblogs.com/jenry/archive/2012/02/11/2347012.html)），这个属性用于描述后画上的图形跟原先在画布上的图形的图层叠加关系应该是什么样的。

这里我们采用`ctx.globalCompositeOperation = 'source-over';`，将后绘制的图形叠加到原先有的图形上。

伪代码如下：

```js

var percent = 0; // 声明一个全局变量
var len = path.getTotalLength(); // 获取path的总长

function animation(){

  ctx.globalCompositeOperation = 'source-over';
  ctx.globalAlpha = 0.2; // 值是0-1,0代表完全透明，1代表完全不透明

  percent >= 100 ? percent = 0 : percent += 0.3;
  var point = path.getPointAtLength(len * (percent / 100));
  X = point.x; // 获取小球的X坐标
  Y = point.y; // 获取小球的Y坐标
  
  Ball.paint(); // 画球 

  window.requestAnimationFrame(animation) // 循环

}

```

### 线、球对象构建

小球的运动依赖于SVG的path。而SVG的path和Canvas的markline都依赖于所提供的始末点和计算出来的控制点。因此，可以在创建一条markline的同时生成一条SVG的path。然后把这条path“通知”给小球。然后再进行运动循环即可让小球运动起来。

于是里目前我们创建了三个对象：

- MarkLine——用于开放接口连接外部
- Line——用于保存markline的信息，提供绘制markline的方法
- Ball——用于保存ball的信息，提供绘制ball的方法

同时，在这个库里还需要创建两个数组：lines和balls。分别用于存放生成的Line、Ball的实例——用于之后调用它们的方法。

于是Line的创建大致是如下：

```js
var Line = function(i,option,canvas){
  this.id = i;
  this.canvas = canvas;
  this.ctx = canvas.getContext("2d");
  this.x1 = option.from.x;
  this.y1 = option.from.y;
  this.x2 = option.to.x;
  this.y2 = option.to.y;
  this.style = option.style || "#fff";
  this.info = option.info || "";
  this.init(); // 初始化
  
  lines[lineCount] = this; // 用于存放Line的实例
  lineCount++;
  this.createBall(); // 通过Line创建Ball
}

Line.prototype = {
    init: function(){
      var cPoint = calControlPoint(this.x1,this.y1,this.x2,this.y2);
      this.cx = cPoint.x;
      this.cy = cPoint.y;
      this.angle = calcAngle(this.cx,this.cy,this.x2,this.y2);
      // 创建小球运动的svg路线
      this.path = document.createElementNS('http://www.w3.org/2000/svg','path');
      this.path.setAttribute('d','M' + this.x1 + ' ' + this.y1 + ' ' + 'Q' + this.cx + ' ' + this.cy + ' ' + this.x2 + ' ' + this.y2);
      this.len = this.path.getTotalLength();
    },
    paint: function(){
      var ctx = this.ctx;
      ctx.save();
      ctx.beginPath();
      ctx.globalAlpha = 1;
      ctx.shadowOffsetX = 0;
      ctx.shadowOffsetY = 0;
      ctx.shadowBlur = 10;
      ctx.shadowColor="rgba(255,255,255,0.3)";
      this.draw(1);
      ctx.strokeStyle = this.style;
      ctx.stroke();
      drawArrow(ctx,this.x2,this.y2,this.angle,20,10,this.style);
      ctx.closePath();
      ctx.restore();
    },
    draw: function(width){
      var ctx = this.ctx;
      ctx.moveTo(this.x1,this.y1);
      ctx.quadraticCurveTo(this.cx,this.cy,this.x2,this.y2);
      ctx.lineWidth = width;
      ctx.lineCap   = 'round';
    },
    createBall: function(){
      var obj = {
        ctx: this.ctx,
        x1: this.x1,
        y1: this.y1,
        path: this.path,
        len: this.len,
        style: this.style
      }
      new Ball(this.id,obj);
    },
  }
```

上述过程就是创建了一个叫做Line的类，这个类包含着基本的`init()`初始化，`draw()`绘制，`paint()`渲染以及`createBall()`等方法。

我们在实例化一个Line的对象的时候，就会将当前实例的对象push一份到lines的数组里。这样是为了之后能够方便地在`animation()`这个函数里调用创建好的对象的方法。

举个例子：

```js

function animation(){
  ......
  for(var i = 0; i < lines.length; i++){
    lines[i].paint(); // 绘制markline
  }
  ......
}
```

那么Ball的创建也是同理，通过Line的创建，然后自动触发Line.createBall()的方法创建小球。创建的同时将小球所需要的基本信息传入。同样，实例化一个小球的同时会将这个对象push一份到balls的数组里。

通过这样的方式，就可以实现绘制动态运动小球了。

### 拖拽和缩放的实现——事件的监听

接着我们来实现一下事件的监听。因为canvas以及canvas上的元素跟dom元素有着本质的区别，对于浏览器而言，canvas上绘制的任何东西，都只跟canvas这个画布本身有关，不具备dom元素自带的事件、属性。所以如果要识别canvas上的元素，进行事件监听的话，只能对于整个canvas进行事件监听，用鼠标的位置、鼠标对于整个canvas的事件来进行canvas上的元素识别、改变。

对于canvas而言，我们能够监听的鼠标事件大致有这几个：

- mousedown
- mouseup
- mousemove
- mouseover
- contextmenu

#### 拖拽

拖拽事件可以拆解成：首先`mousedown`，然后在此基础上，`mousemove`，然后`mouseup`结束拖拽。

于是写法上结构应该是这样：

```js
canvas.addEventListener("mousedown",function(e){
  ...... // 鼠标按下
  canvas.addEventListener("mousemove",function(e){
  ...... // 鼠标按下之后进行移动
  });
  canvas.addEventListener("mouseup",function(e){
  ...... // 鼠标按下之后松开鼠标
  });
})
```

结构很简单。不过要如何实现拖拽的效果呢？

思路是：

1. 首先在`mousedown`的时候，记录这个瞬间的坐标值（相对于canvas画布的）
2. 在`mousemove`的时候，同样记录`mousemove`瞬间的坐标值，然后将这个坐标值与`mousedown`的坐标值进行比较，得到`坐标变换公式`，然后将这个应用到这个画布上所有绘制的元素的坐标上
3. 清空画布
4. 元素重绘

这样就能够实现在视觉上拖拽的效果了。

然而这里面有很关键的东西：坐标变换公式。

#### 数学运算时间

> 写canvas就是在跟数学打交道啊

来实现一下背景的移动：

```js
canvas.addEventListener("mousedown",function(e){

  // 获取当前鼠标点击的坐标
  var mouse = {
    x : e.clientX - canvas.getBoundingClientRect().left,
    y : e.clientY - canvas.getBoundingClientRect().top
  }; 

  // 计算偏移函数
  function offset(mouse,x,y){
    return {
      x : mouse.x - x,
      y : mouse.y - y
    } 
  }

  var imgOffset = offset(mouse,imgPosition.x,imgPosition.y); // 背景图偏移量

  canvas.addEventListener("mousemove",function(e){

    // 获取当前鼠标移动的坐标
    var mouse = {
      x : e.clientX - canvas.getBoundingClientRect().left,
      y : e.clientY - canvas.getBoundingClientRect().top
    };

    imgPosition.x = mouse.x - imgOffset.x;
    imgPosition.y = mouse.y - imgOffset.y;

    cleanCvs(); // 清除画布
    paintBg(); // 绘制背景图
  });
  canvas.addEventListener("mouseup",function(e){
  ...... // 鼠标按下之后松开鼠标
  });
})
```

通过不停的用当前的坐标与之前的坐标进行对比，就可以得到真正应该出现的地方的坐标了。

#### 缩放

其实缩放有很多种，（主要是缩放中心点不同分成很多种缩放效果）。最好的缩放效果自然是跟随鼠标的位置进行缩放——可以参考PC端各种地图的缩放效果。

缩放的时候就需要统一一下坐标了。简单来说，我们可以通过先实现背景的缩放，然后其他的元素可以相对于背景的缩放进行缩放。这样，坐标就统一到背景上了。

缩放的算法比拖拽难度高一些。基本代码如下：

```js

function scaleCvs(scale,e){
    scale > 1 ? scaleFlag += 1 : scaleFlag -=1;
    e = e || window.event;
    // 获取鼠标的位置
    var mouse = {
      x : e.clientX - canvas.getBoundingClientRect().left, // 距离canvas左侧的距离
      y : e.clientY - canvas.getBoundingClientRect().top  // 距离canvas顶部的距离
    };

    // 获取鼠标中心点与背景的左上角坐标偏移量
    var offset = {
      x : mouse.x - imgPosition.x,
      y : mouse.y - imgPosition.y
    }

    // 缩放的值
    var translate = {
      x: (1 - scale) * offset.x,
      y: (1 - scale) * offset.y,
    }

    // 背景偏移最后坐标值
    var imgTransX = imgPosition.x + translate.x;
    var imgTransY = imgPosition.y + translate.y;

    // 计算曲线端点缩放后的位置
    for(var i = 0; i < lines.length; i++){
      var tempOption = {
        x1 : scale * (lines[i].x1 - imgPosition.x) + imgTransX,
        y1 : scale * (lines[i].y1 - imgPosition.y) + imgTransY,
        x2 : scale * (lines[i].x2 - imgPosition.x) + imgTransX,
        y2 : scale * (lines[i].y2 - imgPosition.y) + imgTransY
      }
    }

    // 背景位置、长宽
    imgPosition.x = imgTransX;
    imgPosition.y = imgTransY;
    canWidth = scale * canWidth;
    canHeight = scale * canHeight;

    cleanCvs();
    paintBg();

  }
```

### window.requestAnimationFrame

在这个方法出现之前，我们只能用setTimeout(function(){},1000 / 60)的方式进行模拟60帧的绘制，并且效率低下，资源占用高。然而有了[window.requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)这个属性了之后，浏览器层面的支持的动画渲染，能够有效提升效率，资源占有率低。

当然这个属性并不是完美支持所有浏览器。所以有人写了一个这个方法的[polyfill](https://github.com/darius/requestAnimationFrame)，能够兼容绝大多数浏览器（在不支持这个方法的浏览器里使用setTimeout）。markline.js实际上也引入了这个库。在此感谢。

其实整个markline.js的绘制实际上就是构建在一个window.requestAnimationFrame的不停的刷新的函数中。

## 结尾

写完这个库的时候我是长出一口气，之前我从未接触过面向对象的编程，到这次实现这个面向对象的库真的不容易。学到了不少，也让我更加爱上了原生的js、canvas。

如果我的这篇文章能够帮助你理解一些canvas的东西或者面向对象的一些写法的话，我表示十分荣幸。

如果你喜欢这库，请给个star啦，也欢迎做做contributor~

markline.js的github[地址](https://github.com/Molunerfinn/markline.js)
