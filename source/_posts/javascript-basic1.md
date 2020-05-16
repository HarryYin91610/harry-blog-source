---
title: js事件循环中的宏观任务与微观任务
date: 2020-05-16 18:09:25
tags:
- review

categories:
- JavaScript
---

### 事件循环机制

JavaScript 引擎会等待宿主环境分配宏观任务，在操作系统中，通常等待的行为都是一个事件循环，所以在 Node 术语中，也会把这个部分称为事件循环。

<!--more-->

**事件循环（简易版）：**
```
while(TRUE) {
    r = wait();
    execute(r);
}
```

* 宏观任务：由宿主环境分配，一个宏观任务就是一次事件循环，因此宏观任务的队列就相当于事件循环系统。
* 微观任务：每个宏观任务包含一个微观任务队列，事件循环会保证当前宏观任务的所有微观任务执行完毕才会进入下一轮循环。

**注意：JavaScript中Promise产生的异步代码即是微观任务。**

<img width="500px" height="auto" style="float: left;" src="./javascript-basic1/jbasic1.jpg">
<div style="clear: both"></div>

### 为什么Promise总是先于setTimeout执行？

**setTimeout是浏览器（宿主）API，属于宏观任务， 而JavaScript中Promise产生的异步代码则是微观任务。**

因此，下列代码中，虽然setTimeout只设置了0ms，但是它的回调执行，仍然需要等待下面promise执行完，最后输出”c1 c2 d“。
```
setTimeout(()=>console.log("d"), 0)
var r = new Promise(function(resolve, reject){
    resolve()
});
r.then(() => { 
    var begin = Date.now();
    while(Date.now() - begin < 1000);
    console.log("c1") 
    new Promise(function(resolve, reject){
        resolve()
    }).then(() => console.log("c2"))
});
```

### 分析异步执行顺序的方法

* 首先我们分析有多少个宏任务；
* 在每个宏任务中，分析有多少个微任务；
* 根据调用次序，确定宏任务中的微任务执行次序；
* 根据宏任务的触发规则和调用次序，确定宏任务的执行次序；
* 确定整个顺序。
