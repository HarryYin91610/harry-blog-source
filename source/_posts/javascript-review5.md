---
title: JavaScript 基础知识复习（第十章）
date: 2019-12-25 23:14:49
tags:
- review

categories:
- JavaScript
---

## Node节点关系

### 属性
* childNodes：每个节点都有一个childNodes属性，保存着一个NodeList对象（类数组对象）
```
var firstChild = someNode.childNodes[0];
var secondChild = someNode.childNodes.item(1);
var arrayOfNodes = Array.prototype.slice.call(someNode.childNodes,0); // 将NodeList转化为数组
```

* parentNode：每个节点都有parentNode属性，指向其父节点
* previousSibling、nextSibling：每个节点都有这两个属性，指向其同胞节点，其中首子节点的previousSibling为null，最后一个子节点的nextSibling为null。
* firstChild、lastChild：父节点有这两个属性分别指向childNodes列表中的第一个和最后一个子节点。

<!--more-->

### 方法
* hasChildNodes：判断是否有子节点。

## 操作Node节点

### 方法

* appendChild()：在childNodes末尾添加一个节点，并返回添加的节点。
**注意：如果添加的节点已经是文档的一部分，该方法则会将该节点从原来位置移动到新位置。**
```
var returnedNode = someNode.appendChild(someNode.firstChild); // 将第一个节点移动到末尾
```

* insertBefore(newNode, referNode)：将节点插入到参照节点的前面（previousSibling），并将插入的节点返回。
```
var returnedNode = someNode.insertBefore(newNode, someNode.firstChild);
alert(returnedNode === someNode.firstChild); // true
```
*  replaceChild(newNode, OriginNode)：将新节点替换旧节点，同时删除并返回旧节点。
```
var returnedNode = someNode.replaceChild(newNode, someNode.firstChild);
```

* removeChild(targetChild)：移除并返回目标节点。
```
var formerFirstChild = someNode.removeChild(someNode.firstChild);
```

* cloneNode(deep)：复制一个节点，只接受一个参数表示是否执行深复制（深复制会复制节点及其整个子节点树），复制之后还需要通过appendChild、insertBefore、replaceChild才能将它添加到文档中。
```
var deepList = myList.cloneNode(true); // myList是一个ul节点，有三个li子节点，deepList则同样有三个li
```

* normalize()：处理文本节点，包括删除空文本节点，合并子文本节点。

--------------------------------------------
## node类型(nodeType)
* 1：元素类型节点
* 2: Attr类型节点
* 3：文本类型节点
* 8：注释类型节点（comment）
* 9：document类型节点
* 10：DocumentType类型节点
* 11：DocumentFragment类型节点（文档片段）

--------------------------------------------
### 元素类型节点

#### HTML元素标准特性（通过dom访问）
* id
* title
* lang：元素内容的语言代码；
* dir：语言的方向（ltr 从左往右，rtl 从右往左）；
* className: 与元素class特性对应；

#### 操作元素的特性
* getAttribute()
* setAttribute()
* removeAttribute()

#### 创建元素
* createElement(tagname)：创建一个与标签名对应的元素。（不区分大小写）
```
var div = document.createElement('div');
```
**注意：创建完元素后还需要通过appendChild、insertBefore或者replaceChild把元素添加到文档中。**

--------------------------------------------
### document类型节点

#### document对象的属性
* documentElement：指向html元素；
* body：指向body元素；
* title： 可以查看或者设置文档title标题；
* URL：当前页面的完整链接；
* domain：当前页面的域名；（松散的域名不能再设置为紧绷的）
```
document.domain = 'wrox.com'; // 松散的
document.domain = 'p2p.wrox.com' // 紧绷的
```
* referrer：请求当前页面的来源页面的链接（没有则为空字符串）。
**注意：如果当前页面URL包含子域名，如p2p.wrox.com，那么只能将domain设置为"wrox.com"**
```
// 假设页面来自p2p.wrox.com域
document.domain = 'wrox.com'; // 成功
document.domain = 'nczonline.net'; // 失败
```
* domain的用途：
**当页面中包括来自其他子域的框架或者内嵌框架时，通过设置domain为同一个主域，则能互相访问对方的javascript对象。**
```
一个页面加载自www.wrox.com,包含一个内嵌框架页面加载自p2p.wrox.com，由于domain不同不能互相通信，如果将domain设置成wrox.com则可以解决。
```

#### document获取元素的方法
* getElementById(id)
* getElementsByTagName(tagname)
* getElementsByName(name)：返回所有带有name特性的元素。

#### 文档写入
* write()
* writeln()：会在写入的字符串末尾添加换行符(\n)。
* open()：打开网页输出流。
* close()：关闭网页输出流。
**通过动态写入可以动态引入外部资源**
```
document.write('<script type="\"text/>javascript\" src=\"file.js\"">' + '<\/script>');
```

--------------------------------------------
### 文本类型节点

#### createTextNode()：创建文本节点
#### normalize()：合并相邻文本节点
```
var element = document.createElement('div');
element.className = 'message';

var textNode = document.createTextNode('Hello world');
element.appendChild(textNode);

var textNode2 = document.createTextNode('Hello world2');
element.appendChild(textNode2);

alert(element.childNodes.length); // 2

element.normalize();
alert(element.childNodes.length); // 1
```

--------------------------------------------
### 注释类型节点
#### createComment(str)：创建注释节点

--------------------------------------------
### DocumentFragment类型节点（文档片段）
文档片段不能直接添加到文档中，但可以作为“仓库”来使用，保存将来可能会添加的文档节点。
通过appendChild、insertBefore将文档片段添加到文档中时，**文档片段本身永远不会成为文档树的一部分，只会将它的所有子节点添加到文档树上。**
#### createDocumentFragment()：创建文档片段
```
var fragment = document.createDocumentFragment();
var ul = document.getElementById('myList');
var li = null;
for (var i = 0; i < 3; i++) {
  li = document.createElement('li');
  li.appendChild(document.createTextNode('item ' + (i+1)));
  fragment.appendChild(li);
}
ul.appendChild(fragment);
```
--------------------------------------------
## 动态脚本
```
function loadScript (url) {
  var script = document.createElement('script');
  script.type = 'text/javascript';
  script.src = url;
  document.body.appendChild(script);
}

loadScript('client.js');
```

## 动态样式
```
function loadStyle (url) {
  var link = document.createElement('link');
  link.rel = 'stylesheet';
  link.type = 'text/css';
  link.href = url;
  var head = document.getElementsByTagName('head')[0];
  head.appendChild(link);
}
```