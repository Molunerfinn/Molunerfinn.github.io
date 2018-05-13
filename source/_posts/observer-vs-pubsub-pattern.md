title: 【NOTE】观察者模式VS订阅发布模式
tags: 
  - note
  - Nodejs
  - JS
categories:
  - 笔记
date: 2018-05-12 23:34:00
---
最近在看了一篇[《不好意思，观察者模式跟发布订阅模式就是不一样》](https://juejin.im/post/5af05d406fb9a07a9e4d2799)的文章之后对于这两个模式产生了比较浓厚的兴趣。不过奈何我的水平有限，看完那篇文章还是不能理解。不过在和朋友讨论之后，我想我应该是弄懂了。所以特地记下一篇笔记，以便回头翻阅的时候能够想起来。如果理解有误，欢迎在下方评论指出，一起讨论！

<!-- more -->
## 概述

有人说这两个模式其实是一个模式。我想这句话的对错对半分吧。它们有类似的地方，不过也不能说完全一致。先来一张图，这张图解释了`观察者模式`和`发布订阅模式`在流程上的一些区别：

![](https://img.piegg.cn/observer-pubsub.png?imageslim)

左边是观察者模式，右边是订阅发布模式。

简单阐述二者的模型：

观察者模式里，观察者（Observer）直接订阅（subscribe）主题（Subject），而当主题被激活的时候，会触发（fire）观察者里的事件。

订阅发布模式里，订阅者（Subscriber）通过监听（on）事件总线（Event Bus）里的事件，当事件总线里的事件被触发（emit）的时候，订阅者将会执行相应的操作。而这里需要注意的是，事件总线里的事件是通过发布者（Publisher）进行发布（publish）和 通知事件总线 **触发** 的。

> 注：事件总线也有说法叫为调度中心。本质上是一样的。不过因为写Vue时候习惯用Event Bus来说了，所以本文的调度中心皆以事件总线称呼。

所以事件总线本身不独自发布和触发事件，它会借由发布者来操作。这是跟观察者模式有着比较大的区别的地方。

当然只看这两张图和上面的解释，应该还是无法很好的理解。下面这张图能把流程讲得更清楚点。

![](https://img.piegg.cn/observer-vs-pubsub-2.png?imageslim)

这个例子可以理解为这样：左边是微信里的`微商-顾客`之间的关系。右边是`商家-淘宝-顾客`之间的关系。

观察者模式：顾客关注了微商的商品，微商会记住顾客关注的商品，一旦上新就直接 **私聊** 通知所有关注这个商品的顾客。这里的顾客就相当于观察者，这里的微商就相当于主题。
订阅发布模式：顾客通过淘宝（APP或者网站）关注了商家的商品，商家一旦上新就通过淘宝（APP或者网站）向关注了它的顾客 **群发** 消息。这里的顾客就是订阅者，这里的淘宝就是事件总线，这里的商家就是发布者。

所以可以看出，观察者模式的模型跟发布订阅模型里，差距就差在有没有一个中央的事件总线。如果有这个事件总线，我们就可以认为是个发布订阅模型。如果没有，那么就可以认为是个观察者模型。因为其实它们都实现了一个关键的功能：发布事件-订阅事件并触发事件。

下面用代码简单解释一下。

## 观察者模式

> 由于最近在学习TypeScript，所以下面的代码也会用TypeScript来书写。

我们先写一个定义观察者和主题的文件。

```ts
// observer-pattern.ts

interface Subjects {
  [key: string]: any
}
// 观察者
class Observer {
  subject: string
  constructor (subject: string) {
    this.subject = subject
  }
  notify () {
    console.log(`This ${this.subject} was fired!`)
    this.subject = `Done`
  }
}

// 主题
class Subject {
  // 根据主题的不同收集相应的订阅者
  subjects: Subjects = {}
  // 订阅
  add (subject, observer: Observer): void {
    if (!this.subjects[subject]) {
      this.subjects[subject] = []
    }
    this.subjects[subject].push(observer)
  }
  // 解除订阅
  remove (subject, observer: Observer): void {
    this.subjects[subject].forEach((item, index) => {
      if (item === observer) {
        this.subjects[subject].splice(index, 1)
      }
    })
  }
  // 触发事件
  fire (subject): void {
    this.subjects[subject].forEach(item => item.notify())
  }
}

export {
  Observer,
  Subject
}
```

于是在调用的时候，是这样调用的：

```ts
import * as op from './observer-pattern'
let observer = new op.Observer('click')
let subjects = new op.Subject()
subjects.add('click', observer)
subjects.fire('click') // subjects 主动通知
```
经过上述调用，subjects触发观察者订阅的click事件，`observer.subject`的值将会变为`Done`（原先为`click`）。

## 订阅发布模式

接下来我们来实现一些订阅发布模式。订阅发布模式最关键的地方就在于中间的`Event Bus`部分。它接管着事件总线的订阅和发布。

```ts
// pubsub.ts

interface Subjects {
  [key: string]: any
}

// 定义Event Bus
class EventBus {
  subjects: Subjects = {}
  on (subject, callback): void {
    /* istanbul ignore next */
    if (!this.subjects[subject]) {
      this.subjects[subject] = []
    }
    this.subjects[subject].push(callback)
  }
  off (subject, callback = null): void {
    if (callback === null) {
      this.subjects[subject] = []
    } else {
      this.subjects[subject].forEach((item, index) => {
        /* istanbul ignore next */
        if (item === callback) {
          this.subjects[subject].splice(index, 1)
        }
      })
    }
  }
  emit (subject, data = null): void {
    this.subjects[subject].forEach(item => item(data))
  }
}

export default new EventBus()
```

可以看出在这里的`EventBus`和观察者模式里的`Subject`几乎一致对吧。但是需要注意的是，最后一行里，我们`export default new EventBus()`，所以我们在项目里不同的地方`import`它，都会指向同一个`Event Bus`实例，这样的话就可以起到一个事件总线的作用了。它不在乎谁来监听，谁来发布。只要有人监听了，就把它放进监听队列中。只要有人发布了事件，就从相应的监听队列中触发回调。不过所有相关的事件都必须经过`Event Bus`这个实例，而不能越过它直接由发布者通知监听者。

> 再次祭出这张图

![](https://img.piegg.cn/observer-vs-pubsub-2.png?imageslim)

所以在订阅发布模型里，发布者或者订阅者的身份已经被弱化。发布者可以在任何时候发布事件，而订阅者可能只是一个回调函数。而最关键的事件总线部分，则是发布订阅模型的核心。

如果你用过Vue的`Event Bus`，相信不会陌生。接下来我们来用用我们刚才写的简单的`Event Bus`。

```ts
import bus from './pubsub.ts'
const people = function (val) {
  console.log('我收到了新的商品通知：', val) // 收到消息
}

bus.on('newItem', people) // 订阅newItem这个消息

const merchant = function (val) { // 由商户向event bus发布新商品
  const item = {
    item: val
  }
  bus.emit('newItem', item)
}

merchant('Book') // 发布
```
所以你可以看到，这个事件总线是可以单独抽离出来的。如果要把我们这个文件丢到一个现有的项目里也是完全没问题的。

其实在写Vue组件通信的时候，你如果用到了`Event Bus`的话，也是一样的。在全局声明一个`new Vue()`做`Event Bus`总线，然后在不同的组件里只要引入了这个事件总线，就能订阅或者发布不同的消息。这个就是一个非常典型的订阅发布模型。

而如果只是Vue的父子组件通信，子组件用的是`this.$emit`来触发事件，父组件用的是`this.$on`这样的方式去订阅事件，那么你可以认为这个就是一个简单的观察者模型。因为它们之间的联系是紧密耦合的。

## 总结

不管是观察者模式也好，订阅发布模式也好，关键在于实现了在某个特定时间触发某个特定事件，从而触发监听这个特定事件的组件进行相应操作的功能。这个设计模式在很多时候非常有用。平时只是用到了它，但是没有深入去看看如何实现，这次借由这个机会把二者的关系和区别记录下来，也算是给自己加深了印象。

本文的代码你可以在我的学习仓库[FE-Learning](https://github.com/Molunerfinn/FE-Learning/tree/master/design-pattern)找到。如有错误欢迎指出！

## 参考资料

https://www.zcfy.cc/article/observer-vs-pub-sub-pattern-hacker-noon

http://blog.zxbing0066.com/design-patterns/2016/09/12/observer-pattern.html

https://juejin.im/post/5af05d406fb9a07a9e4d2799

https://www.cnblogs.com/weebly/p/5279952.html

https://www.jianshu.com/p/3098b1176357

https://www.zhihu.com/question/23486749/answer/314072549
