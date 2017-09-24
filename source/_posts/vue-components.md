title: Vue组件的三种调用方式
tags: 
  - 前端
  - Vue
categories:
  - Web
  - 开发
date: 2017-06-15 14:23:00
---

最近在写[fj-service-system](https://fj.teamsz.xyz)的时候，遇到了一些问题。那就是我有些组件，比如`Dialog`、`Message`这样的组件，是引入三方组件库，比如`element-ui`这样的，还是自己实现一个？虽然它们有按需引入的功能，但是整体风格和我的整个系统不搭。于是就可以考虑自己手动实现这些简单的组件了。

<!-- more -->

通常我们看Vue的一些文章的时候，我们能看到的通常是讲Vue单文件组件化开发页面的。单一组件开发的文章相对就较少了。我在做[fj-service-system](https://fj.teamsz.xyz)项目的时候，发现其实单一组件开发也是很有意思的。可以写写记录下来。因为写的不是什么ui框架，所以也只是一个记录，没有github仓库，权且看代码吧。

主要讲三种方式调用组件：

- `v-model`或者`.sync`显式控制组件显示隐藏
- 通过js代码调用
- 通过Vue指令调用

在写组件的时候很多写法、灵感来自于[element-ui](https://github.com/ElemeFE/element)，感谢。

### Dialog

我习惯把这个东西叫做对话框，实际上还有叫做modal(弹窗)组件的[叫法](https://www.zhihu.com/question/35820643)。其实就是在页面里，弹出一个小窗口，这个小窗口里的内容可以定制。通常可以用来做登录功能的对话框。

![dialog](https://ws1.sinaimg.cn/large/8700af19ly1fgoih3m6apg20kw0do4pv.gif)

这种组件就很适合通过`v-model`或者`.sync`的方式来显式的控制出现和消失。它可以直接写在页面里，然后通过data去控制——这也是最符合Vue的设计思路的组件。

为此我们可以写一个组件就叫做`Dialog.vue`

```html
<template>
  <div class="dialog">
    <div class="dialog__wrapper" v-if="visble" @clcik="closeModal">
      <div class="dialog">
        <div class="dialog__header">
          <div class="dialog__title">{{ title }}</div>
        </div>
        <div class="dialog__body">
          <slot></slot>
        </div>
        <div class="dialog__footer">
          <slot name="footer"></slot>
        </div>
      </div>
    </div>
    <div class="modal" v-show="visible"></div>
  </div>
</template>

<script>
  export default {
    name: 'dialog',
    props: {
      title: String,
      visible: {
        type: Boolean,
        default: false
      }
    },
    methods: {
      close() {
        this.$emit('update:visible', false) // 传递关闭事件
      },
      closeModal(e) {
        if (this.visible) {
          document.querySelector('.dialog').contains(e.target) ? '' : this.close(); // 判断点击的落点在不在dialog对话框内，如果在对话框外就调用this.close()方法关闭对话框
        }
      }
    }
  }
</script>
```

CSS什么的就不写了，跟组件本身关系比较小。不过值得注意的是，上面的`dialog__wrapper`这个class也是全屏的，透明的，主要用于获取点击事件并锁定点击的位置，通过DOM的`Node.contains()`方法来判断点击的位置是不是dialog本身，如果是点击到了dialog外面，比如半透明的`modal`层那么就派发关闭事件，把dialog给关闭掉。

当我们在外部要调用的时候，就可以如下调用：

```html
<template>
  <div class="xxx">
    <dialog :visible.sync="visible"></dialog> 
    <button @click="openDialog"></button>
  </div>
</template>

<script>
  import Dialog from 'Dialog'
  export default {
    components: {
      Dialog
    },
    data() {
      return {
        visible: false
      }
    },
    methods: {
      openDialog() {
        this.visible = true // 通过data显式控制dialog
      }
    }
  }
</script>
```

为了Dialog开启和关闭好看点，你可试着加上`<transition></transition>`组件配合上过渡效果，简单的一点过渡动效也将会很好看。

### Notice

这个组件类似于`element-ui`的[message](http://element.eleme.io/#/zh-CN/component/message)（消息提示）。它吸引我的最大的地方在于，它不是通过显式的在页面里写好组件的html结构通过v-model去调用的，而是通过在js里通过形如`this.$message()`这样的方法调用的。这种方法虽然跟Vue的数据驱动的思想有所违背。不过不得不说在某些情况下真的特别方便。

![notice](https://ws1.sinaimg.cn/large/8700af19ly1fgoiiwtkymg20ju07m46t.gif)

对于Notice这种组件，一次只要提示几个文字，给用户简单的消息提示就行了。提示的信息可能是多变的，甚至可以出现叠加的提示。如果通过第一种方式去调用，事先就得写好html结构，这无疑是麻烦的做法，而且无法预知有多少消息提示框。而通过js的方法调用的话，只需要考虑不同情况调用的文字、类型不同就可以了。

而之前的做法都是写一个Vue文件，然后通过`components`属性引入页面，显式写入标签调用的。那么如何将组件通过js的方法去调用呢？

这里的关键是Vue的`extend`[方法](https://vuejs.org/v2/api/#Vue-extend)。

文档里并没有详细给出`extend`能这么用，只是作为需要手动`mount`的一个Vue的组件构造器说明了一下而已。

通过查看`element-ui`的源码，才算是理解了如何实现上述的功能。

首先依然是创建一个`Notice.vue`的文件

```html
<template>
  <div class="notice">
    <div class="content">
      {{ content }}
    </div>
  </div>
</template>

<script>
  export default {
    name: 'notice',
    data () {
      return {
        visible: false,
        content: '',
        duration: 3000
      }
    },
    methods: {
      setTimer() {
        setTimeout(() => {
          this.close() // 3000ms之后调用关闭方法
        }, this.duration)
      },
      close() {
        this.visible = false
        setTimeout(() => {
          this.$destroy(true)
          this.$el.parentNode.removeChild(this.$el) // 从DOM里将这个组件移除
        }, 500)
      }
    },
    mounted() {
      this.setTimer() // 挂载的时候就开始计时，3000ms后消失
    }
  }
</script>
```

上面写的东西跟普通的一个单文件Vue组件没有什么太大的区别。不过区别就在于，没有props了，那么是如何通过外部来控制这个组件的显隐呢？

所以还需要一个js文件来接管这个组件，并调用`extend`方法。同目录下可以创建一个`index.js`的文件。

```js
import Vue from 'vue'

const NoticeConstructor = Vue.extend(require('./Notice.vue')) // 直接将Vue组件作为Vue.extend的参数

let nId = 1

const Notice = (content) => {
  let id = 'notice-' + nId++

  const NoticeInstance = new NoticeConstructor({
    data: {
      content: content
    }
  }) // 实例化一个带有content内容的Notice

  NoticeInstance.id = id
  NoticeInstance.vm = NoticeInstance.$mount() // 挂载但是并未插入dom，是一个完整的Vue实例
  NoticeInstance.vm.visible = true
  NoticeInstance.dom = NoticeInstance.vm.$el
  document.body.appendChild(NoticeInstance.dom) // 将dom插入body
  NoticeInstance.dom.style.zIndex = nId + 1001 // 后插入的Notice组件z-index加一，保证能盖在之前的上面
  return NoticeInstance.vm
}

export default {
  install: Vue => {
    Vue.prototype.$notice = Notice // 将Notice组件暴露出去，并挂载在Vue的prototype上
  }
}
```

这个文件里我们能看到通过`NoticeConstructor`我们能够通过js的方式去控制一个组件的各种属性。最后我们把它注册进Vue的prototype上，这样我们就可以在页面内部使用形如`this.$notice()`方法了，可以方便调用这个组件来写做出简单的通知提示效果了。

当然别忘了这个相当于一个Vue的插件，所以需要去主js里调用一下`Vue.use()`方法：

```js
// main.js

// ...
import Notice from 'notice/index.js'

Vue.use(Notice)

// ...

```

### Loading

在看`element-ui`的时候，我也发现了一个很有意思的组件，是`Loading`，用于给一些需要加载数据等待的组件套上一层加载中的样式的。这个loading的调用方式，最方便的就是通过`v-loading`这个指令，通过赋值的`true/false`来控制Loading层的显隐。这样的调用方法当然也是很方便的。而且可以选择整个页面Loading或者某个组件Loading。这样的开发体验自然是很好的。

![loading](https://ws1.sinaimg.cn/large/8700af19ly1fgoija6xrkg20k6082js0.gif)

其实跟Notice的思路差不多，不过因为涉及到`directive`，所以在逻辑上会相对复杂一点。

平时如果不涉及Vue的`directive`的开发，可能是不会接触到`modifiers`、`binding`等概念。参考[文档](https://vuejs.org/v2/guide/custom-directive.html)

简单说下，形如`:v-loading.fullscreen="true"`这句话，`v-loading`就是`directive`，`fullscreen`就是它的`modifier`，`true`就是`binding`的`value`值。所以，就是通过这样简单的一句话实现全屏的loading效果，并且当没有`fullscreen`修饰符的时候就是对拥有该指令的元素进行`loading`效果。组件通过`binding`的`value`值来控制`loading`的开启和关闭。（类似于`v-model`的效果）

其实loading也是一个实际的DOM节点，只不过要把它做成一个方便的指令还不是特别容易。

首先我们需要写一下`loading`的Vue组件。新建一个`Loading.vue`文件

```html
<template>
  <transition
    name="loading"
  	@after-leave="handleAfterLeave">
    <div
      v-show="visible"
      class="loading-mask"
      :class={'fullscreen': fullscreen}>
      <div class="loading">
        ...
      </div>
      <div class="loading-text" v-if="text">
        {{ text }}
      </div>
    </div>
  </transition>
</template>
<script>
export default {
  name: 'loading',
  data () {
    return {
      visible: true,
      fullscreen: true,
      text: null
    }
  },
  methods: {
    handleAfterLeave() {
      this.$emit('after-leave');
    }
  }
}
</script>
<style>
.loading-mask{
  position: absolute; // 非全屏模式下，position是absolute
  z-index: 10000;
  background-color: rgba(255,235,215, .8);
  margin: 0;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  transition: opacity .3s;
}
.loading-mask.fullscreen{
  position: fixed; // 全屏模式下，position是fixed
}
// ...
</style>
```

Loading关键是实现两个效果：

1. 全屏loading，此时可以通过插入body下，然后将Loading的position改为fixed，插入body实现。
2. 对所在的元素进行loading，此时需要对当前这个元素的的position修改：如果不是`absolute`的话，就将其修改为`relatvie`，并插入当前元素下。此时Loading的position就会相对于当前元素进行绝对定位了。

所以在当前目录下创建一个`index.js`的文件，用来声明我们的`directive`的逻辑。

```js
import Vue from 'vue'
const LoadingConstructor = Vue.extend(require('./Loading.vue'))

export default {
  install: Vue => {
    Vue.directive('loading', { // 指令的关键
      bind: (el, binding) => {
        const loading = new LoadingConstructor({ // 实例化一个loading
          el: document.createElement('div'),
          data: {
            text: el.getAttribute('loading-text'), // 通过loading-text属性获取loading的文字
            fullscreen: !!binding.modifiers.fullscreen 
          }
        })
        el.instance = loading; // el.instance是个Vue实例
        el.loading = loading.$el; // el.loading的DOM元素是loading.$el
        el.loadingStyle = {};
        toggleLoading(el, binding);
      },
      update: (el, binding) => {
        el.instance.setText(el.getAttribute('loading-text'))
        if(binding.oldValue !== binding.value) {
          toggleLoading(el, binding)
        }   
      },
      unbind: (el, binding) => { // 解绑
        if(el.domInserted) {
          if(binding.modifiers.fullscreen) {
              document.body.removeChild(el.loading);
          }else {
            el.loading &&
            el.loading.parentNode &&
            el.loading.parentNode.removeChild(el.loading);
          }
        }
      }
    })

    const toggleLoading = (el, binding) => { // 用于控制Loading的出现与消失
      if(binding.value) { 
        Vue.nextTick(() => {
          if (binding.modifiers.fullscreen) { // 如果是全屏
            el.originalPosition = document.body.style.position;
            el.originalOverflow = document.body.style.overflow;
            insertDom(document.body, el, binding); // 插入dom
          } else {
            el.originalPosition = el.style.position;
            insertDom(el, el, binding); // 如果非全屏，插入元素自身
          }
        })
      } else {
        if (el.domVisible) {
          el.instance.$on('after-leave', () => {
            el.domVisible = false;
            if (binding.modifiers.fullscreen && el.originalOverflow !== 'hidden') {
              document.body.style.overflow = el.originalOverflow;
            }
            if (binding.modifiers.fullscreen) {
              document.body.style.position = el.originalPosition;
            } else {
              el.style.position = el.originalPosition;
            }
          });
          el.instance.visible = false;
        }
      }
    }

    const insertDom = (parent, el, binding) => { // 插入dom的逻辑
      if(!el.domVisible) {
        Object.keys(el.loadingStyle).forEach(property => {
          el.loading.style[property] = el.loadingStyle[property];
        });
        if(el.originalPosition !== 'absolute') {
          parent.style.position = 'relative'
        }
        if (binding.modifiers.fullscreen) {
          parent.style.overflow = 'hidden'
        }
        el.domVisible = true;
        parent.appendChild(el.loading) // 插入的是el.loading而不是el本身
        Vue.nextTick(() => {
          el.instance.visible = true;
        });
        el.domInserted = true;
      }
    }
  }
}
```

同样，写完整个逻辑，我们需要将其注册到项目里的Vue下：

```js
// main.js

// ...
import Loading from 'loading/index.js'

Vue.use(Loading)

// ...
```

至此我们已经可以使用形如

```html
<div v-loading.fullscreen="loading" loading-text="正在加载中">
```

这样的方式来实现调用一个loading组件了。

### 总结

在用Vue写我们的项目的时候，不管是写页面还是写形如这样的功能型组件，其实都是一件很有意思的事情。本文介绍的三种调用组件的方式，也是根据实际情况出发而实际操作、实现的。不同的组件通过不同的方式去调用，方便了开发人员，也能更好地对代码进行维护。当然也许还有其他的方式，我并没有了解，也欢迎大家在评论里指出！

最后再次感谢[element-ui](https://github.com/ElemeFE/element)的源码给予的极大启发。