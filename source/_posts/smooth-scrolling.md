---
title: 滚动动画的性能提升方案
date: 2019-02-27 15:45:00
tags:
- requestAnimationFrame
- scroll

categories:
- Algorithm

---

### 旧版方案
由于js本身的方法：window.scrollTo(offsetx, offsety) 是直接改变滚动条的位置，缺少滚动的过程会使交互显得生硬，于是
我首先想到的是利用setInterval 去实现这个滚动效果。
<!--more-->
```
// 核心方法
function moveScreenTo (start, end) {
  if (start !== end) {
    let step = 100
    let direc = start < end ? 1 : -1 // 1--> 向下, -1-->向上
    step *= direc
    var timer = setInterval(() => {
      if ((direc === 1 && start >= end) || (direc === -1 && start <= end)) {
        clearInterval(timer)
      } else {
        window.scrollTo(0, start + step)
        start = start + step
      }
    }, 10)
  }
}
```
#### 发现问题  
刚开始在自己的mac上实现的动画效果还算顺滑，但是当我换成性能较差的windows电脑时，这个滚动动画就会有明显的卡顿，刚开始我以为是滚动算法的问题，但是在几经优化后发现其实最根本的原因是setInterval被阻塞了。

#### 对于阻塞根本原因的理解
* 由于大多数浏览器的渲染频率是16.7ms/帧，所以当我设置setInterval的时间间隔小于16.7ms时，就会出现丢失帧的问题，也就是卡顿的最大原因。
* 其次是因为setInterval的运行机制，setInterval中的任务并不会在当前Event Loop(任务队列)中运行，而是会等到当前Event Loop中所有同步任务执行完，在下一次Event Loop开始时才会判断是否到了指定时间，如果到了时间就执行setInterval中的任务,如果没有到就在下一次Event Loop开始时再判断一次。这样的运行机制会导致setInterval中的任务被同步运行的任务所阻塞，就会导致卡顿现象。

--------------------------------------------

### 解决方案：requestAnimationFrame
requestAnimationFrame 是为动画而生的方法，主要用于处理 js 动画，其用法和setTimeout非常相似。
#### 优势
1. requestAnimationFrame 会把每一帧中的所有 DOM 操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率；
2. 在隐藏或不可见的元素中，requestAnimationFrame 将不会进行重绘或回流，这当然就意味着更少的 CPU、GPU 和内存使用量；
3. requestAnimationFrame 是由浏览器专门为动画提供的 API，在运行时浏览器会自动优化方法的调用，并且如果页面不是激活状态下的话，动画会自动暂停，有效节省了 CPU 开销。

#### 兼容性
支持Chrome、Firefox、ie10以上，但不兼容 ie9及以下浏览器，但是我们可以用定时器来做一下兼容，以下是兼容代码：
```
function initRequestAnimationFrame () {
  // 初始化兼容不同浏览器
  var lastTime = 0
  var vendors = ['webkit', 'moz']
  for (var x = 0; x < vendors.length && !window.requestAnimationFrame; ++x) {
    window.requestAnimationFrame = window[vendors[x] + 'RequestAnimationFrame']
    window.cancelAnimationFrame = window[vendors[x] + 'CancelAnimationFrame'] ||
                                  window[vendors[x] + 'CancelRequestAnimationFrame']
  }
  if (!window.requestAnimationFrame) {
    window.requestAnimationFrame = function (callback, element) {
      var currTime = new Date().getTime()
      var timeToCall = Math.max(0, 16.7 - (currTime - lastTime))
      var id = window.setTimeout(() => {
        callback(currTime + timeToCall)
      }, timeToCall)
      lastTime = currTime + timeToCall
      return id
    }
  }
  if (!window.cancelAnimationFrame) {
    window.cancelAnimationFrame = function (id) {
      window.clearTimeout(id)
    }
  }
}
```

--------------------------------------------

### 改造后的滚动动画
* 改造后的滚动明显比旧版顺滑很多，没有卡顿现象出现。
* 用Promise封装滚动动画，便于在外层使用async函数来控制动画流程。

#### 改造后的代码
```
function moveScreenTo (start, end) {
  return new Promise((resolve, reject) => {
    let curPos = start
    let step = 40

    const run = function () {
      if ((step > 0 && curPos >= end) || (step < 0 && curPos <= end)) {
        resolve()
      } else {
        curPos = curPos + step
        window.scrollTo(0, curPos)
        window.requestAnimationFrame(run)
      }
    }

    if (start !== end) {
      const direc = start < end ? 1 : -1 // 1--> 向下, -1-->向上
      step *= direc
      run()
    } else {
      resolve()
    }
  })
}
```
