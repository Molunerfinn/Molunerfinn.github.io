title: 移动端web开发初探之Vuejs的简单实战
tags: 
  - 前端
categories:
  - Web
  - 开发
date: 2016-06-26 17:06:00
---
这段时间在做的东西，是北邮人论坛APP的注册页。这个注册页是内嵌的网页，因为打算安卓和IOS平台同时使用。因此实际上就是在做移动端的web开发了。  
在这过程中遇到了不少有意思的东西。

**DEMO的github地址在[这里](https://github.com/Molunerfinn/vue-mobile-learning-demo)**

<!-- more -->

内容提要：

- meta标签
- Vuejs的简单实战
- CSS移动端全屏背景
- CSS移动端动画初探

## meta标签

这点与在PC端写前端有着很大的区别，移动端的meta标签简直多。我就说说我所用到的标签。

```html
<!-- 1、如果支持Google Chrome Frame：GCF，则使用GCF渲染；2、如果系统安装ie8或以上版本，则使用最高版本ie渲染；3、否则，这个设定可以忽略。 -->
<meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1">
<!-- 对视窗缩放等级进行限制，使其适应移动端屏幕大小 -->
<meta name="viewport" content="width=device-width, initial-scale=1">
<!-- 当把这个网页添加到主屏幕时的标题（仅限IOS） -->
<meta name="apple-mobile-web-app-title" content="北邮人论坛注册">
<!-- 添加到主屏幕后全屏显示 -->
<meta name="apple-touch-fullscreen" content="yes" />
```

尤其是第二个meta标签，是移动端适配非常重要的一句话。

## 整体布局

整体的布局大致是4-5页横向布局。第一页是用来填写注册信息的。后面的几页是用来选择关注的版面的。

### 注册信息页

![注册信息页](https://img.piegg.cn/blog/1.png)

### 关注版面页

![关注版面页](https://img.piegg.cn/blog/2.png)
![关注版面页](https://img.piegg.cn/blog/3.png)
![关注版面页](https://img.piegg.cn/blog/4.png)

## 整体架构

前端采用的架构大致是这样：

- 滑动切换页面的效果来自[swiper.js](http://idangero.us/swiper)
- 页面内容的渲染采用[Vue.js](http://vuejs.org.cn/)
- 页面布局和样式采用纯css，部分效果采用css3
- ajax部分采用[vue-resource](https://github.com/vuejs/vue-resource)

后端支撑的框架来自php的[laravel](http://www.golaravel.com/)，当然，这不是本文的重点，仅提及一下。  

是的这次的开发中，已经看不到jquery的身影了——这也是前端以后发展后的结果——慢慢地脱离jquery的依赖。不过jquery给前端带来的改变和发展是无人能替代的。

## swiper.js的应用

引入swiper.js来进行页面的切换效果纯粹是因为这次开发的周期要求比较短，要考虑效果和兼容性兼备的情况下，我就偷懒找了一个动画库。  

不过这个动画库的效果我还是算比较满意的。而整体来说使用也相当方便。尤其是，swiper.js是可以不依赖jquery的。

使用起来也比较方便。我简要说说用法。

首先需要在页面顶部的`head`标签里加入swiperjs的css文件：

`<link rel="stylesheet" href="css/swiper.min.css')">`

然后在页面底部可以引入和写下相应的js：

```javascript
<script src="{{ asset('js/swiper.min.js') }}"></script>
  <!-- Initialize Swiper -->
<script>
  var swiper = new Swiper('.swiper-container', {
    pagination: '.swiper-pagination',
    paginationType: 'progress',
    noSwipingClass: 'swiper-no-swiping',
    allowSwipeToNext: true,
    allowSwipeToPrev: true,
  }); 
</script>
```

解释一下，创建一个swiper的对象，然后这对象的容器是class叫做`swiper-container`的一个html元素。对其的配置是：

- pagination: '.swiper-pagination', 显示页码
- paginationType: 'progress', 页码显示格式为进度条，可以参见顶部蓝白色的进度条
- noSwipingClass: 'swiper-no-swiping',  不允许进行触摸滑动的元素的class名称
- allowSwipeToNext: true, 允许向后滑动
- allowSwipeToPrev: true, 允许向前滑动

相应的HTML代码可以如下：

```html

<!-- swiper生效的容器 -->
<div class="swiper-contanier">
  <div class="swiper-wrapper">
    <!-- 具体滑动的页面 -->
    <div class="swiper-slide"></div>
    <div class="swiper-slide"></div>
    <div class="swiper-slide"></div>
    <div class="swiper-slide"></div>
  </div>
</div>
<!-- 进度条 -->
<div class="swiper-pagination"></div>

```

有些页面是不能直接让用户通过触摸来前后滑动的，而必须通过点击按钮触发。比如第一页注册页，这个页面就是必须填写完信息然后点击下一步进行验证后然后才可以滑动到后面的内容。所以只要将这个页面所在的class里加上`swiper-no-swiping`这个class就可以实现无法触摸滑动切换页面了。

然后我们可以通过`swiper.slideNext(bool,time)`这个方法来进行手动控制向后翻页的动作以及控制动画的时长。这个内容会在vue里说到。

## Vue.js的简单实战

由于本次的开发只是在原有的北邮人开放平台的项目的基础上加入一个快速注册的功能，所以Vue.js的引入并不是为了将整个项目重构，而只是为了尝试一下脱离jquery的情况下对于开发来说不同的体验是什么，能学到什么。

关于Vue.js的用法、特性什么的不是本文的重点，题外话就不说了。说说本次实现思路。因为我的Vuejs实际上也只是算入门级别，只能说会用但是还没到能驾驭的程度。

首先整个页面包在class叫做`swiper-contanier`的元素中。因为这个元素包括了滑动所需的所有东西，我们实际上可以简单的把它看成是一个SPA(Single Page Application)，用一个Vue的实例就可以接管整个页面了（这么说是不负责的，因为实际上Vue是组件化的思想）。

先创建一个基本的Vue的示例，并将其和`swiper-container`绑定起来：

```js

var vm = new Vue({
  el: ".swiper-contanier", // 将这个实例与html元素绑定起来
  data: {}, // 所需要变动、关注的数据，也是vue的核心
  ready: function(){}, // vue提供的钩子，用于在vue渲染视图完成后立即触发
  methods: {} // 方法，用于操作、更新、改变数据而改变视图
})

```

### 注册页实现

上面所说，我们的页面是构建在一个Vue的实例上的。因此不同类型的两种页面如何用一个实例来接管呢？在这里我的实现方式是用两类数据来分别表示。  

我们分析一下注册页：

![注册信息页](https://img.piegg.cn/blog/1.png)

实际上注册页的中间部分是重复的元素，他们都是input标签+显示文字标签（对，尤其注意这里并不是用placeholder实现的）。效果：

![效果](https://img.piegg.cn/blog/5.gif)

所以这中间的部分实际上可以看成是一个列表，可以用Vue的v-for来渲染。列表里所不同的只是显示的文字不同以及input框的类型不同（有text类型的，有password类型的），所以用数据绑定的方式我们可以将这个页面的数据格式安排如下：

```
data: {
  main: [
    {"name":"username","info":"用户名(以英文开头+英文数字)","type":"text"},
    {"name":"passwd","info":"设置密码","type":"password"},
    {"name":"passwd_confirm","info":"在输入一遍密码","type":"password"}，
    {"name":"gwno","info":"校园网账户(默认是学号)","type":"text"},
    {"name":"gwpwd","info":"校园网密码(默认是身份证后六位)","type":"password"},
  ],
}
```

同时我们创建一个便于和前端视图进行双向绑定的数据对象`userInfo`：

```
data: {
  main: [...],
  userInfo: {
    username: "",
    passwd: "",
    passwd_confirm: "",
    gwno: "",
    gwpwd: ""
  }
}

```

而在前端的话我们就可以用这个数据来进行视图渲染：

```html

<ul class="user-info">
  <li v-for="item in main" style="position: relative;">
    <!-- input输入框 -->
    <!-- 此处用了v-model将数据和视图进行了双向绑定 -->
    <input class="effect" type="{{item.type}}" v-model="userInfo[item.name]">
    <!-- 提示信息 -->
    <label>
      <span>{{item.info}}</span>
    </label>
    <!-- input框的底下的线条 -->
    <span class="focus-border cube"></span>
  </li> 
</ul>

<a href="#"><button>下一步</button></a>

```

于是一个输入的列表就很容易做出来了。然后既然是表单，就需要验证。而此处做的验证实际上有这么几点：

- 用户名是否合法/重复？
- 两次输入的密码是否相同？
- 校园网账户的密码是否正确？

其中只有第二点两次密码输入是否相同可以用前端直接判断，而第一、三点都是需要通过ajax的方式向后台发送验证请求的。为了能够体现和辨别错误与否，我们在main下的每个条例里加入了一个error属性，并规定如下三种状态：

- true，代表有错误，提示错误
- false，代表正确，提示正确
- normal，代表默认，显示默认值

于是我们可以在method下写一些方法来进行判断。

```js
checkUserId: function(msg){
  if (msg !== ""){
    this.$http.post('url'+msg,function(data){
      if (data.success){
        this.main[0].error = false;
      } else{
        this.main[0].error = true;
      }
    })
  } else{
      this.main[0].error = "normal";
  }
},
checkUserPwd: function(){
    if (this.userInfo.passwd_confirm !== ""){
        this.userInfo.passwd == this.userInfo.passwd_confirm && this.userInfo.passwd_confirm != "" ? this.main[2].error = false : this.main[2].error = true;
    }
},
```

当然这个只是一个引入的功能，我们再聚合一下：

```js
check: function(msg,i){
  var index = i;
  this.userInfo[msg] != "" ? this.main[index].effect = true : this.main[index].effect = false;
  switch (msg){
    case "username":
      this.checkUserId(this.userInfo[msg]);                                      
      break;
    case "passwd":                                                                 
      this.userInfo.passwd !== "" ? this.main[1].error = false : this.main[1].error = "normal";       
      this.checkUserPwd();                                                       
      break;                                                                     
    case "passwd_confirm":
      this.checkUserPwd();                                                       
      break;
    case "gwno":
      this.userInfo.gwno !== "" ? this.main[3].error = false : this.main[3].error = "normal";     
      break;
    case "gwpwd":
      this.userInfo.gwno !== "" ? this.main[4].error = false : this.main[4].error = "normal";     
      break; 
  }     
}
```
这样通过一个check的method我们就可以将整个表单的验证的方法容纳进来了。（此处对于校园网账号的验证会放到提交表单的函数中）。为了能在视图中体现正确、错误、正常的不同形态，我们需要对前端的一些结构进行一些修改。我们需要给main下的每个条例加入错误信息，也即errorInfo。

所以至此整个main的结构是这样：

```
main: [
  {"name":"","info":"","type":"","error":"","errorInfo":""},
  ...
]
```

然后我们需要加入提交验证的部分：

```js
submitReg: function(){
  var flag = 0;
  // 用于判断表单是否都是正确的
  this.main.map(function(obj){
    obj.error == false ? flag += 1 : flag +=0;
  })
  if (flag == 5){
    this.$http.post('url',this.userInfo)
    .then(function(res){
      if (res.success){
        swiper.slideNext(false,300); // 验证正确就可以进入下一页
      } else{
        this.main[4].error = true;
      }
    })
  }
},

```

将视图部分修改如下：

```html
<ul class="user-info">
  <li v-for="item in main" style="position: relative;">
    <!-- 绑定blur事件 -->
    <input @blur="check(item.name,$index)" class="effect" type="{{item.type}}" v-model="userInfo[item.name]">
    <!-- 根据error类型切换不同的标签显示 -->
    <label>
      <!-- 此处用到了v-show的方法 -->
      <span v-show="item.error == 'normal'">{{item.info}}</span>
      <span style="color: red;" v-show="item.error == true">{{item.errorInfo}}</span>
      <span v-show="item.error == false">{{item.info}}√</span>
    </label>
    <span class="focus-border cube"></span>
  </li>
</ul>
<!-- 点击下一步的同时提交表单信息 -->
<a href="#"><button @click="submitReg">下一步</button></a>
```

至此，整个注册页的布局和功能性的部分都已经做完了。不过这只是一块比较简单的部分，我们用到了vuejs的的v-for进行列表渲染，用到了v-model进行数据的双向绑定，用到了method进行一些数据的处理，用到了v-show进行条件显示，一个基本的页面下我们已经能尝试这么多vue的特性了。而相比于jquery的dom操作，我觉得vue在此处最好的地方在于，表单的提交很方便。由于双向绑定了数据，只需要后台把数据格式规定好给我，我按照后端的数据结构整理一下我前端的数据结构然后就可以直接提交给后端了。而且省去了很多dom操作的地方。

不过除了vue的部分，我还想来说说两个东西，跟css有关：

- 全屏背景
- 输入框文字提升效果，并涉及到移动端动画效果的处理

#### 全屏背景

不仅仅是简单的`background-size: cover`那么简单了，还需要进行小小的处理。先说说我希望实现的效果吧。我希望的效果是整个背景能够填充整个页面，并且在页面元素上下滚动的情况下，背景固定而不随着元素滚动。

放到往常我可能会这么写：

```css

body,html{
  height: 100%;
}

body{
  background: url(bg.png) center 0 no-repeat;
  background-size: cover;
}

```

但是这样的话在移动端会出现比较严重的后果，那就是一旦页面元素的高度大于整个页面后，滚动页面元素的时候，背景也会随之而动。而且背景会被撑开。这不是我所希望的。

这里用到一个小技巧，用上`:before`的方法。

```css
body:before {
  content: "";
  position: fixed;
  z-index: -1;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background: url(bg.png) center 0 no-repeat;
  background-size: cover;
}
```
这个用上before的伪元素的方法是一个很有奇效的小技巧。大家不妨可以试试。这样的话在移动端也能完美实现背景固定而且显示全屏。


#### 移动端动画简单初探

在做PC端的web的动画效果的时候，由于PC端的性能足够，所以在写一些效果的时候往往没有考虑到性能的问题。这次在开发的时候就遇到了动画效果实现上不小的问题。大家先来看看这两个动画的比较：上面一个动画和下面一个动画：

第一个：

<p data-height="550" data-theme-id="dark" data-slug-hash="zBGrxx" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/zBGrxx/">transform effect</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

第二个：

<p data-height="551" data-theme-id="dark" data-slug-hash="oLXjKz" data-default-tab="result" data-user="molunerfinn" data-embed-version="2" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/molunerfinn/pen/oLXjKz/">repaint effect</a> by molunerfinn (<a href="http://codepen.io/molunerfinn">@molunerfinn</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

你会发现这两个东西的效果相差甚大。实际上，二者实现的最终效果是一致的，都是让小球按一条口型路线运动。但是为何显示上来说，第一个这么流畅，而第二个有明显卡顿呢？

这里涉及到很多东西，不光光是重绘（repaint），还有软硬件性能上面的问题。感兴趣的话可以参考这些文章：

- [Web动画性能指南](http://alexorz.github.io/animation-performance-guide/)
- [高性能CSS3动画](https://www.qianduan.net/high-performance-css3-animations/)
- [CSS动画的性能优化](http://zencode.in/14.CSS%E5%8A%A8%E7%94%BB%E7%9A%84%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.html)

在移动端上的效果如果没有优化的话，实际上大致就是第二种的效果——让人看起来有所卡顿。简单来说，在移动端，有些效果是由浏览器来渲染的——那么这些效果如果比较复杂，而移动端的浏览器性能又不太足够的情况下，效果就比较卡顿。有些效果可以由GPU来渲染，那么这些效果渲染起来就相对来说比较流畅。而我们还可以人为触发GPU硬件渲染，通过 `transform`的`translate3d`属性就能实现硬件渲染从而俗称硬件加速。  

举个简单例子。如果让一个元素从(0,0)->(100px，0)，正常思路是`left: 0`->`left: 100px`，然后再用`transition`属性进行过渡。不过这样的话在移动端上效果就很感人，因为涉及到性能问题。但是如果我们用另外一种方式: `transform: translate3d(100px,0,0)`的话，就可以让这个动画效果由GPU去渲染，那么这样的话，在移动端的效果也是完全可以接受的流畅。

实际上，能够触发GPU渲染的动作有`opacity,transform,transition,animation`等等。但是像`top,left,color,size`等属性的变化则不会触发GPU渲染。  

本文中描述的实例，是当焦点集中到input框的时候，文本上移并且有颜色变化。本来想实现的是文本大小还有变化。但是由于上面说的情况，涉及到size变化的时候，效果就会大打折扣。无奈之下我只能砍掉这个效果了。而实现文本上移实际上就是采用了transform的translate3d的方式，将其往上移动，并配合transition进行了一下过渡处理罢了。

具体的CSS实现大致如下：

```CSS

input:focus ~ label,.trans {                                                                 
  color: #fff;                                                                                 
  transition: 0.3s;                                                                            
  -webkit-transform: translate3d(0, -20px, 0);                                                 
  -moz-transform: translate3d(0, -20px, 0);                                                    
  -ms-transform: translate3d(0, -20px, 0);                                                     
  transform: translate3d(0, -20px, 0);                                                         
}   

```
实际上也不难不是么？

### 关注版面页的实现

实际上从注册页的实现中已经可以瞅见关注版面页实现的方法了。实际上关注版面页不过也是列表的渲染，在数据里定义好相应的属性就行了。然后用一个`picked`的属性来看看是否被选中即可。最后完成注册的时候，将所有选中的列表组装成相应的数组提交到后台就行了。因为注册页里这些方法已经说过了，所以就不再赘述了。

## 总结

简单总结一下，这次移动端的开发中学习到的东西。

- Vuejs的简单使用，从DOM操作->数据绑定
- 全屏背景的实现
- 流畅动画效果的实现

实际上，兼容性方面还有不少的东西需要诉说，不过限制于篇幅以及本文的主要内容并不是在纠结移动端的兼容性的所以并没有在兼容性方面进行记录。  

在排版上我还是没有用上比较流行的flex，对于Vue还没有使用组件化开发的思路。这些都是需要改进的部分。不过这次的开发过程中经过美工的指点，将我之前写页面的时候注意不到的很多细节部分，比如线条尺寸，元素间隔，颜色搭配等东西给指了出来，这些东西都不是简简单单就实现的。总之，有时间的话，我想我可以将设计方面的知识融入到我的前端开发中，想必能让我的作品更加地符合审美和用户的体验吧~


**DEMO的github地址在[这里](https://github.com/Molunerfinn/vue-mobile-learning-demo)**








