---
title: 完美的全屏背景视频展示
date: 2018-11-30 15:45:00
tags:
- fullscreen

categories:
- Algorithm

---

### 背景
在公司的一个年终活动项目中，设计要求我们实现一个全屏背景视频的效果，按常理来说，这并不是一件麻烦事，fixed定位即可实现；可仔细一想，面对着各式各样大大小小的屏幕尺寸，以及用户自定义的浏览器尺寸，该如何在不改变基本宽高比的情况下完美地去展示视频背景？此处的完美应该是尽可能多地展示整个原始视频内容，不能留白。

<!--more-->

### 思考
参考css中background-size的一个属性cover,这个属性的作用是让背景图占满整个指定大小的区域，同时不改变背景图的宽高比，且尽可能多地展示背景图内容，超出部分会被截掉。这不正是我们所需的展示效果吗？可是视频或者canvas元素又如何去实现呢？

### 解决方案
1. 要不改变视频的宽高比，就不能同时让宽高为100%;
2. 在浏览器大小初始化和改变时，需要进行判断：到底是让宽还是高为100%？
   我判断的方式是，首先让宽为100%，计算出视频尺寸在等比下的高度，若高度小于浏览器的高度，则需要让高为100%；

### 具体实现
最核心的一个方法就是判断是否该撑满浏览器的高度，若为false则撑满浏览器宽度。
```
const isFullHeight = function (ratio) {
  // 浏览器宽度最小尺寸1200
  let winWidth = document.documentElement.clientWidth
  winWidth = winWidth < 1200 ? 1200 : winWidth
  const winHeight = document.documentElement.clientHeight
  const sdHeight = winWidth * ratio
  if (sdHeight < winHeight) {
    // 高度撑满
    return true
  }
  return false
}
```
有了这个方法我们就可以通过改变视频或者canvas元素的类来控制其宽高的样式。

