title: 【译】Having fun with Html5 Canvas
tags: 
  - 前端
categories:
  - Web
  - 开发
date: 2017-04-09 15:11:00
---
> 本文翻译自[Having fun with Html5 Canvas](http://www.dorodnic.com/blog/2014/10/17/html5-canvas-part1)，感谢作者

本篇教程我们将会构建一个齿轮系统，用HTML canvas和JavaScript描述出来。

> 本教程适合刚学习JavaScript以及对齿轮系统只有最基本了解的读者。

<!-- more -->

### 第一部分 渲染单个齿轮

这个简单的物理齿轮制作[教程](http://www.instructables.com/id/How-to-make-gears-easily/step4/Working-out-the-size-of-everything/)是一个好的开端。根据这个教程，我们的齿轮将由它的半径和齿数来定义。

首先，让我们用齿轮的有关属性来创建一个齿轮类：

```js
var Gear = function(x, y, connectionRadius, teeth, fillStyle, strokeStyle) {
    // 齿轮参数
    this.x = x;
    this.y = y;
    this.connectionRadius = connectionRadius;
    this.teeth = teeth;

    // 渲染参数
    this.fillStyle = fillStyle;
    this.strokeStyle = strokeStyle;

    // 计算属性
    this.diameter = teeth * 4 * connectionRadius; // 每个轮齿是通过两个相连的半圆组成的
    this.radius = this.diameter / (2 * Math.PI); // D = 2 PI r

    // 运动属性
    this.phi0 = 0; // 起始角度
    this.angularSpeed = 0; // 角速度cond
    this.createdAt = new Date(); // 时间戳
}
```

接着写渲染的方法：

```js
Gear.prototype.render = function (context) {
    // 更新旋转角
    var ellapsed = new Date() - this.createdAt;
    var phiDegrees = this.angularSpeed * (ellapsed / 1000);
    var phi = this.phi0 + deg2rad(phiDegrees); // 当前的角度

    // 构建渲染参数
    context.fillStyle = this.fillStyle;
    context.strokeStyle = this.strokeStyle;
    context.lineCap = 'round';
    context.lineWidth = 1;

    // 绘制齿轮轮身
    context.beginPath();
    for (var i = 0; i < this.teeth * 2; i++) {
        var alpha = 2 * Math.PI * (i / (this.teeth * 2)) + phi;
        // 计算每个轮齿的位置
        var x = this.x + Math.cos(alpha) * this.radius;
        var y = this.y + Math.sin(alpha) * this.radius;
        // 画一个半圆，随着alpha旋转
        // 在每个奇数齿，画相反的半圆
        context.arc(x, y, this.connectionRadius, -Math.PI / 2 + alpha, Math.PI / 2 + alpha, i % 2 == 0);
    }
    context.fill();
    context.stroke();

    // 画中心的圆
    context.beginPath();
    context.arc(this.x, this.y, this.connectionRadius, 0, 2 * Math.PI, true);
    context.fill();
    context.stroke();
}
```

使用方法：

```js
var canvas = document.getElementById('myCanvas');
var context = canvas.getContext('2d');
var W = canvas.width;
var H = canvas.height;
var gear = new Gear(W / 2, H / 2, 5, 12, "white", "rgba(61, 142, 198, 1)");
gear.angularSpeed = 36;
setInterval(function () {
    canvas.width = canvas.width;
    gear.render(context);
}, 20);
```

以及HTML：

```html
<canvas id="myCanvas" width="200" height="200"></canvas>
<script src="myScript.js" type="text/javascript"></script>
```

<p data-height="265" data-theme-id="0" data-slug-hash="PpgQBB" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-pen-title="PpgQBB" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/PpgQBB/">PpgQBB</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

### 第二部分 渲染两个啮合的齿轮

我们的目的是让用户控制第二个齿轮的位置。同时我们想要保证齿轮仍然啮合并且同步转动。

下面是个例子：

```js
Gear.prototype.connect = function (x, y) {
    var r = this.radius;
    var dist = distance(x, y, this.x, this.y); // 计算两个齿轮之间的距离

    // 要创建一个新的齿轮我们必须知道它的齿数
    var newRadius = Math.max(dist - r, 10);
    var newDiam = newRadius * 2 * Math.PI;
    var newTeeth = Math.round(newDiam / (4 * this.connectionRadius)); // 齿数必须是整数

    // 创建一个新的齿轮
    var newGear = new Gear(x, y, this.connectionRadius, newTeeth, this.fillStyle, this.strokeStyle);

    // 调整新齿轮的旋转方向使其与原来的方向相反
    var gearRatio = this.teeth / newTeeth;
    newGear.angularSpeed = -this.angularSpeed * gearRatio;
    return newGear;
}
```

我们可以把这个挂载在Canvas的`mousemove`的事件上。

```js
var gear2 = gear.connect(3 * (W / 4), H / 2);

// 这是一个辅助函数，用于转换鼠标在canvas内部的坐标值
function getMousePos(canvas, evnt) {
    var rect = canvas.getBoundingClientRect();
    return {
        x: evnt.clientX - rect.left,
        y: evnt.clientY - rect.top
    };
}

canvas.onmousemove = function (evnt) {
    var pos = getMousePos(canvas, evnt);
    var x = Math.min(0.7 * W, Math.max(0.3 * W, pos.x));
    var y = Math.min(0.7 * H, Math.max(0.3 * H, pos.y));
    gear2 = gear.connect(x, y);
}
setInterval(function () {
    canvas.width = canvas.width;
    gear.render(context);
    gear2.render(context);
}, 20);
```

<p data-height="381" data-theme-id="0" data-slug-hash="Mpdvwo" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-pen-title="Mpdvwo" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/Mpdvwo/">Mpdvwo</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

上面的步骤可以实现正确的旋转方向，但是并不是那么准确——齿轮没有啮合，并且每个齿轮只是在做自己的运动。

以下是我们必须解决的两个问题：

1. `gear2.radius`（齿轮2的理论半径）与我们计算的`newRadius`（实际半径）不一致。这是因为我们不得不保留齿数的缘故。
2. 齿轮2并没有和齿轮1同步转动。

要解决第一个问题，我们必须让我们的新齿轮改变它的实际位置以确保在拥有它所需要的齿数的同时还能和第一个齿轮啮合。

（下面是对于问题1的解决方案）

```js
Gear.prototype.connect = function (x, y) {
    var r = this.radius;
    var dist = distance(x, y, this.x, this.y);

    // 要创建一个新的齿轮我们必须知道它的齿数
    var newRadius = Math.max(dist - r, 10);
    var newDiam = newRadius * 2 * Math.PI;
    var newTeeth = Math.round(newDiam / (4 * this.connectionRadius));

    // 计算新齿轮的实际位置，使其能够于该齿轮啮合
    var actualDiameter = newTeeth * 4 * this.connectionRadius;
    var actualRadius = actualDiameter / (2 * Math.PI);
    var actualDist = r + actualRadius; // 距离该齿轮中心的实际距离
    var alpha = Math.atan2(y - this.y, x - this.x); // 该齿轮中心和(x,y)的角度值
    var actualX = this.x + Math.cos(alpha) * actualDist; 
    var actualY = this.y + Math.sin(alpha) * actualDist;

    // 创建一个新的齿轮
    var newGear = new Gear(actualX, actualY, this.connectionRadius, newTeeth, this.fillStyle, this.strokeStyle);

    // 调整新齿轮的旋转方向使其与原来的方向相反
    var gearRatio = this.teeth / newTeeth;
    newGear.angularSpeed = -this.angularSpeed * gearRatio;

    return newGear;
}
```

<p data-height="389" data-theme-id="0" data-slug-hash="BWedRK" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-pen-title="Gear-2-2" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/BWedRK/">Gear-2-2</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

上面的调整单独处理了一个令人讨厌的问题，我们可以在一个齿轮上放置另一个齿轮了。但是这还不足以同步两个齿轮的旋转。

现在我们必须在连接点使齿轮互相啮合：

```js
this.phi0 = alpha; // 在时间t=0,将此齿轮旋转角度α
newGear.phi0 = alpha + Math.PI + (Math.PI / newTeeth);
// 同时（t=0)，旋转新齿轮角度（180-α），对着第一个齿轮
// 并且加上一半的齿轮旋转使它们的轮齿能够啮合
newGear.createdAt = this.createdAt; // 当然，还得同步它们的时钟
```

上面的做法是有效的。然而，这种方法的缺点是我们在一直改变`this.phi`，这意味着以前与之同步的任何其他齿轮将不再同步。

每次`this.phi`都被一些`delta`更新，那么`newGear.phi`应该被更新多少？答案是`delta * (newGear.angularSpeed / this.angularSpeed)`，因为要考虑齿轮转速之比。了解到这个之后，我们可以把两个齿轮都更新一下：`delta = (this.phi0 - alpha)`，以消除这种影响：

```js
// 在时间t=0,将此齿轮旋转角度α
this.phi0 = alpha + (this.phi0 - alpha); // this.phi0，没啥用，仅供展示。
newGear.phi0 = alpha + Math.PI + (Math.PI / newTeeth) + (this.phi0 - alpha) * (newGear.angularSpeed / this.angularSpeed);
// 同时（t=0)，旋转新齿轮角度（180-α），对着第一个齿轮
// 并且加上一半的齿轮旋转使它们的轮齿能够啮合
newGear.createdAt = this.createdAt; // 当然，还得同步它们的时钟
```

通过这样的处理，`this.phi0`保持不变，不过另一个齿轮却得以同步：

<p data-height="441" data-theme-id="0" data-slug-hash="GWavzq" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-pen-title="Gear-2-3" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/GWavzq/">Gear-2-3</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

### 第三部分 3D齿轮

在上一个部分，我们已经展示了如何通过HTML5 Canvas渲染出啮合的齿轮对。本部分，我们将会探索通过使用Three.js这个库，渲染出完全3D的JavaScript齿轮，同时也会进一步扩大视图的复杂性。

#### 3D渲染

Three.js是一个用于3D图形渲染的JavaScript库，是基于WebGL的。它是免费，开放，并且不断更新的。我们将从移植我们的渲染代码开始，首先试着描绘出2D平面里的齿轮轮廓。

```js
var shape = new THREE.Shape();

// 跳转到齿轮顶部的起始位置
var x0 = this.radius;
var y0 = 0;
shape.moveTo(x0, y0);

for (var i = 0; i < this.legs * 2; i++) {
    var alpha = 2 * Math.PI * (i / (this.legs * 2)) + this.phi;
    var x = Math.cos(alpha) * this.radius;
    var y = Math.sin(alpha) * this.radius;

    createArc(shape, x, y, this.connectionRadius,
        -Math.PI / 2 + alpha,  // 起始角度 
        Math.PI / 2 + alpha, // 结束角度
        i % 2 == 0, // 顺时针还是逆时针
        3 // 每段弧的离散段数
    );
}

return shape;
```

我用了一个辅助函数把弧形段分割成离散的段：

```js
function createArc(shape, x, y, radius, from, to, sign, parts) {
    var src = sign ? from : to; // 确保我们总是沿着一个方向移动
    var trg = sign ? to : from;
    var delta = sign ? 0 : Math.PI; // 但在需要的时候可以反转角度

    for (var i = 1; i < parts; i++) {
        var t = i / parts;
        var cx = x + radius * Math.cos(delta + (src * (1 - t) + trg * t));
        var cy = y + radius * Math.sin(delta + (src * (1 - t) + trg * t));
        shape.lineTo(cx, cy);
    }
}
```

当前的代码里有一个问题——啮合的齿轮会从一个小角度phi开始旋转，而不是恰好在`(r,0)`。要正确地获取`(x0,y0)`需要进行以下调整：

```js
var sign = this.legs % 2;
var from = -Math.PI / 2 + this.phi;
var to = Math.PI / 2 + this.phi;
var src = sign ? from : to;
var trg = sign ? to : from;
var delta = sign ? 0 : Math.PI;

var x0 = Math.cos(this.phi)*this.radius + this.connectionRadius*Math.cos(delta + src);
var y0 = Math.sin(this.phi)*this.radius + this.connectionRadius*Math.sin(delta + src);
```

现在，让我们增加一些代码来渲染齿轮中心的孔：

```js
var holePath = new THREE.Path();
holePath.moveTo(this.connectionRadius, 0);
createArc(holePath, 0, 0, this.connectionRadius, 0, 2 * Math.PI, true, 10);
this.shape.holes.push(holePath);
```

一旦我们齿轮的轮廓被描绘，我们可以使用Three.js的拉伸功能使其变得更加真实起来：

```js
var extrudeSettings = {
    steps: 1,
    amount: gearDepth,
    bevelEnabled: false,
};

var geometry = new THREE.ExtrudeGeometry(shape, extrudeSettings);

var material = new THREE.MeshPhongMaterial({
    color: color,
    polygonOffset: true,
    polygonOffsetFactor: 1.0,
    polygonOffsetUnits: 4.0
});

var mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);
```

这里是结果：

<p data-height="374" data-theme-id="0" data-slug-hash="oZrExv" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-pen-title="Gear-3-1" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/oZrExv/">Gear-3-1</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

#### 生成更复杂的场景

使用第二部分的代码我们可以生成一对互相啮合的齿轮，为了生成更复杂的场景，只要我们避免碰撞，可以一次添加一个齿轮。

这个能够生成我们的齿轮啮合网络的方法应该是这样的：

1. 放置随机的齿轮
2. 找到不被任何其他齿轮占用的随机点p = (x,y)
3. 找到最接近的齿轮g
4. 创建一个新的齿轮中心点在p并且与g相连
5. 回到第二步

使用这个方法你可以生成互相啮合的齿轮的整个网络，如下所示：

<p data-height="444" data-theme-id="0" data-slug-hash="GWbbpO" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-pen-title="GWbbpO" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/GWbbpO/">GWbbpO</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

以下是冲突检测查询的一个可能的实现：

```js
function detectCollision(newGear, neighbor) {
    var result = false;
    var that = this;
    this.gears.forEach(function (gear) {
        var dist = distance(gear.x, gear.y, newGear.x, newGear.y);

        if (dist < gear.radius + newGear.radius + 2 * that.connectionRadius + 5 
            && neighbor != gear) {
            result = true;
        }
    });

    return result;
}
```

可以使用类似的方法实现冲突的近邻查询。但是你应该注意一个细节，那就是我们必须避免大齿轮以很大的角速度旋转。否则一些齿轮在动画的帧与帧之间转动得太多，将会形成混乱的情况。

下一个逻辑步骤就是增加我们模型的深度。我们可以随机放置几层，但是这样会让它们看起来不自然。这些层可以通过一些共同的转轴来进行交互。我们可以通过从一层到另一层复制圆点和角速度来模拟这种效果。

```js
function tryAddFromLayer(x, y, legs, angularSpeed) {
    // 通过所给的参数创建一个新的齿轮
    var newGear = new Gear(x, y, this.connectionRadius, 
                           legs, this.fillStyle, this.strokeStyle);

    if (!this.detectCollision(newGear, null)
        // 别忘了确保它不会转的太快
        && this.minFps * angularSpeed * newGear.radius < newGear.diameter
        ) {
        newGear.angularSpeed = angularSpeed; // 调整转速
        this.gears.push(newGear); // 添加到图层
    }
}
```

现在，我们能够堆积从以前生成的层中挑选齿轮，并从中生成新的齿轮：

```js
function addFromLayer(layers, options, maxR) {
    var retries = 0;
    while (this.gears.length < options.maxGears && retries < options.qouta) {
        // 选择一个随机的层
        var layerIdx = Math.floor((options.generator.random() * layers.length));
        var layer = layers[layerIdx];
        // 从这个层里挑选一个随机的齿轮
        var gearIdx = Math.floor((options.generator.random() * layer.gears.length));
        var gear = layer.gears[gearIdx];

        // 随机选择这个齿轮的齿数
        var legs = options.minLegs + Math.floor((options.generator.random() *
            (maxLegsFromRadius(maxR, this.connectionRadius) - options.minLegs)));

        // 向当前的图层里添加结果
        this.tryAddFromLayer(gear.x, gear.y, legs, gear.angularSpeed);

        retries++;
    }
}
```

这是算法的一个可能的输出：

<p data-height="566" data-theme-id="0" data-slug-hash="dvBBZz" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-pen-title="dvBBZz" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/dvBBZz/">dvBBZz</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

所有的源码你都可以通过查看上面的`codepen`的源码以及github[仓库源码](https://github.com/Molunerfinn/Gear-system)获取
