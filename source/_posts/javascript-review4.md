---
title: JavaScript 基础知识复习（第七章）
date: 2019-12-23 22:25:56
tags:
- review

categories:
- JavaScript
---

## 函数表达式
定义函数有两种方式：
1. 函数声明
最重要的特征是**函数声明提升**，意思是在执行代码之前会先读取函数声明，这意味着可以把函数声明放在调用语句之后。

2. 函数表达式
使用前必须先赋值
```
sayHi(); // 会报错
var sayHi = function () {
  alert('Hi!');
};
```

<!--more-->

### 递归
```
function factorial (num) {
  if (num <= 1) {
    return 1;
  } else {
    return num * arguments.callee(num - 1);
  }
}
```

### 闭包
* 闭包是指有权访问另一个函数作用域中变量的函数。
* 作用域链本质上是一个指向变量对象的指针列表。
* 由于闭包会携带包含它的函数的作用域，因此会比其他函数占用更多内存，过渡使用闭包可能导致内存占用过多。

**注意：下面的写法会导致每个匿名函数的作用域中都保存同一个活动变量对象的i，都是10**
```
function createFunctions () {
  var result = new Array();

  for (var i = 0; i < 10; i++) {
    result[i] = function () {
      return i;
    };
  }

  return result;
}
```
改进方法如下：
```
function createFunctions () {
  var result = new Array();

  for (var i = 0; i < 10; i++) {
    result[i] = function (num) {
      return function () {
        return num;
      }
    }(i);
  }

  return result;
}
```
* **关于this对象**
由于匿名函数的执行环境具有全局性，this通常指向window。
具体原因是因为每个函数在调用时会自动取得两个变量：this和arguments。内部函数在搜索这两个变量时只会搜索到自己的活动对象为止，所以永远不可能访问到外部函数的这两个变量。
改进方式：
```
var name = "The Window";
var object = {
  name: "My Object",
  getNameFunc: function () {
    var that = this;
    return function () {
      return that.name;
    }
  }
};

alert(object.getNameFunc()()); // "My Object"
```

### 静态私有变量
```
// 私有作用域：一个立即执行的匿名函数
(function () {
  // 私有变量
  var privateVariable = 10;
  // 私有函数
  function privateFunction () {
    return false;
  }
  
  // 构造函数
  MyObject = function () {};
  // 特权方法：有权访问私有变量和函数的方法
  MyObject.prototype.publicMethod = function () {
    privateVariable++;
    return privateFunction();
  };
})();
```

### 模块模式
* 单例：只有一个实例的对象
```
var singleton = {
  name: "values",
  method: function () {}
};
```
* 模块模式为单例添加私有变量和特权方法
```
var singleton = function () {
  var privateVariable = 10;
  function privateFunction () {
    return false;
  }

  return {
    publicMethod: function () {
      privateVariable++;
      return privateFunction();
    }
  };
}();
```