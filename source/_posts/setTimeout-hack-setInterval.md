---
title: 用setTimeout和clearTimeout简单实现setInterval与clearInterval
tags: 
  - 前端
  - 随笔
categories:
  - Web
  - 开发
  - 随笔
date: 2019-05-08 10:21:00
---

这个问题其实是前一段时间舍友的一道面试题。我觉得类似用`reduce实现map`、用`xxx实现yyy`的题目其实都挺有意思，考察融会贯通的本领。不过相比之下这道题可能更有实际意义。比如我们经常会用 `setTimeout` 来实现倒计时。下面来说说我对这个问题的思考。

<!-- more -->

## 简单版本

首先我们先用 `setTimeout` 实现一个简单版本的 `setInterval`。

`setInterval` 需要不停循环调用，这让我们想到了递归调用自身：

```js
const mySetInterval = (cb, time) => {
  const fn = () => {
    cb() // 执行传入的回调函数
    setTimeout(() => {
      fn() // 递归调用自己
    }, time)
  }
  setTimeout(fn, time)
}
```

让我们来写段代码测试一下：

```js
mySetInterval(() => {
  console.log(new Date())
}, 1000)
```

![setTimeout-1](https://blog-1251750343.cos.ap-beijing.myqcloud.com/setTimeout-1.gif)

嗯，没啥问题，实现了我们想要的功能。。。等一下，怎么停下来？总不能执行了就不管了吧。。。

## clearInterval的实现

平时如果用到了 `setInterval` 的同学应该都知道 `clearInterval` 的存在（不然你怎么停下 `interval` 呢）。

`clearInterval` 的用法是 `clearInterval(id)`。而这个 `id` 是 `setInterval`的返回值，通过这个 `id` 值就能够清除指定的定时器。

```js
const id = setInterval(() => {
  // ...
}, 1000)
// ...
clearInterval(id)
```

不过你有没有想到 `clearInterval` 是如何实现的？回答这个问题之前，我们需要先实现 `mySetInterval` 的返回值。

### mySetInterval的返回值

回到我们简单版本的 `mySetInterval`：

```js
const mySetInterval = (cb, time) => {
  const fn = () => {
    cb() // 执行传入的回调函数
    setTimeout(() => {
      fn() // 递归调用自己
    }, time)
  }
  setTimeout(fn, time)
}
```

现在它的返回值因为没有显示指定，所以是 `undefined`。因此第一步，我们先要返回一个 `id` 出去。

那么直接 `return setTimeout(fn, time)` 可以吗？因为我们知道 `setTimeout` 也会返回一个id，那么初步构想就是通过 `setTimeout` 返回的 `id`，然后调用 `clearTimeout(id)` 来实现我们的 `myClearInterval`。

如下：

```js
const mySetInterval = (cb, time) => {
  const fn = () => {
    cb() // 执行传入的回调函数
    setTimeout(() => { // 第二个、第三个...
      fn() // 递归调用自己
    }, time)
  }
  return setTimeout(fn, time) // 第一个setTimeout
}

const id = mySetInterval(() => {
  console.log(new Date())
}, 1000)

setTimeout(() => { // 2秒后清除定时器
  clearTimeout(id)
}, 2000)
```

这显然是不行的。因为 `mySetInterval` 返回的 `id` 是第一个 `setTimeout` 的 `id`，然而2秒后，要 `clearTimeout` 时，递归执行的第二个、第三个 `setTimeout`  等等的 `id` 已经不再是第一个 `id` 了。因此此时无法清除。

所以我们需要每次执行 `setTimeout`的时候把新的 `id` 存下来。怎么存？我们应该会想到用闭包：

```js
const mySetInterval = (cb, time) => {
  let timeId
  const fn = () => {
    cb() // 执行传入的回调函数
    timeId = setTimeout(() => { // 闭包更新timeId
      fn() // 递归调用自己
    }, time)
  }
  timeId = setTimeout(fn, time) // 第一个setTimeout
  return timeId
}
```

很不错，到这步我们已经能够将 `timeId` 进行更新了。不过还有问题，那就是执行 `mySetInterval` 的时候返回的 `id` 依然不是最新的 `timeId`。因为 `timeId` 只在 `fn` 内部被更新了，在外部并不知道它的更新。那有什么办法让 `timeId` 的更新也让外部知道呢？

有的，答案就是用全局变量。

```js
let timeId // 全局变量
const mySetInterval = (cb, time) => {
  const fn = () => {
    cb() // 执行传入的回调函数
    timeId = setTimeout(() => { // 闭包更新timeId
      fn() // 递归调用自己
    }, time)
  }
  timeId = setTimeout(fn, time) // 第一个setTimeout
  return timeId
}
```

但是这样有个问题，由于 `timeId` 是`Number`类型，当我们这样使用的时候：

```js
const id = mySetInterval(() => { // 此处id是Number类型，是值的拷贝而不是引用
  console.log(new Date())
}, 1000)

setTimeout(() => { // 2秒后清除定时器
  clearTimeout(id)
}, 2000)
```

由于 `id` 是 `Number` 类型，我们拿到的是全局变量 `timeId` 的值拷贝而不是引用，所以上面那段代码依然无效。不过我们已经可以通过全局变量 `timeId` 来清除计时器了：

```js
setTimeout(() => { // 2秒后清除定时器
  clearTimeout(timeId) // 全局变量 timeId
}, 2000)
```

但是上面的实现，不仅与我们平时使用的 `clearInterval` 的用法有所出入，并且由于 `timeId` 是一个 `Number` 类型的变量，导致同一时刻全局只能有一个 `mySetInterval` 的 `id` 存在，也即无法做到清除多个 `mySetInterval` 的计时器。

所以我们需要一种类型，既能支持多个 `timeId` 存在，又能实现 `mySetInterval` 返回的 `id` 能够被我们的 `myClearInterval` 使用。你应该能想到，我们要用一个全局的 `Object` 来做。

修改代码如下：

```js
let timeMap = {}
let id = 0 // 简单实现id唯一
const mySetInterval = (cb, time) => {
  let timeId = id // 将timeId赋予id
  id++ // id 自增实现唯一id
  let fn = () => {
    cb()
    timeMap[timeId] = setTimeout(() => {
      fn()
    }, time)
  }
  timeMap[timeId] = setTimeout(fn, time)
  return timeId // 返回timeId
}
```

我们的 `mySetInterval` 依然返回了一个 `id` 值。只不过这个 `id` 值是全局变量 `timeMap` 里的一个键的内容。

我们每次更新 `setTimeout` 的 `id` 并不是去更新 `timeId`，相应的，我们去更新 `timeMap[timeId]` 里的值。

这样实现后，我们调用 `mySetInterval` 虽然获取到的 `timeId` 是不变的，但是我们通过 `timeMap[timeId]` 获取到的真正的 `setTimeout` 的 `id` 值是会一直更新的。

另外为了保证 `timeId` 的唯一性，在这里我简单用了一个自增的全局变量 `id` 来保证唯一。

好了，`id` 值有了，剩下的就是 `myClearInterval` 的实现了。

### myClearInterval实现

由于我们的 `mySetInterval` 返回的 `timeId` 并不是真正的 `setTimeout` 返回的 `id` ，所以并不能简单地通过 `clearTimeout(timeId)` 来清除计时器。

不过其实原理也是很类似的，我们只要能拿到真正的 `id` 就行了：

```js
const myClearInterval = (id) => {
  clearTimeout(timeMap[id]) // 通过timeMap[id]获取真正的id
  delete timeMap[id]
}
```

测试一下：

![](https://blog-1251750343.cos.ap-beijing.myqcloud.com/setTimeout-2.gif)

没毛病~

至此我们就用 `setTimeout` 和 `clearTimeout` 简单实现了 `setInterval` 与`clearInterval`。当然本文说的是简单实现，毕竟还有一些东西没有完成，比如`setTimeout` 的 `args` 参数、Node和浏览器端的 `setTimeout` 差异等等。也只是一个抛砖引玉，重点在一步步如何实现。感谢阅读~
