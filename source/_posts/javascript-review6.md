---
title: JavaScript 基础知识复习（第十一章）
date: 2019-12-30 23:07:34
tags:
- review

categories:
- JavaScript
---

## 选择符API

* querySelector()：接收css选择符，返回与该模式匹配的第一个元素。
* querySelectorAll()：返回NodeList实例。
* matchesSelector()：接收css选择符，如果调用元素与该选择符匹配则返回true。

## 元素遍历
* childElementCount：返回子元素（不包括文本节点和注释）的个数。
* firstElementChild：指向第一个子元素。
* lastElementChild：指向最后一个子元素。
* previousElementSibling：指向前一个同胞子元素。
* nextElementSibling：指向后一个同胞子元素。

## HTML5
### getElementsByClassName()
### classList属性：是一个DOMTokenList
* add()：将给定字符串添加到列表，若已存在则不添加。
* contains()：判断是否存在给定字符串。
* remove()：删除给定字符串。
* toggle()：如果列表已经存在给定的字符串，则删除；如果不存在则添加它。