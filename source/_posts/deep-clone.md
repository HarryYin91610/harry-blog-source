---
title: 深拷贝的实际运用
date: 2018-11-29 15:38:00
tags:
- deepClone

categories:
- Practice
---

### 背景
在一个实际项目中，需要在接口请求失败或没有数据时展示默认内容，以前遇到这种情况时，最常用的方法是在每一个需要展示的位置进行一次空值判断，但是在数据格式很复杂的情况下，这样的做法会显得很繁琐。于是乎，我就考虑预先设置好所需要的数据格式，并赋予其默认值，在接口有返回数据时进行一次比对，如果该字段接口返回的值不为空则覆盖默认值，否则忽略。
<!--more-->
### 尝试不同实现方式
此时就需要一种算法来合并默认的数据与接口数据，我考虑过使用Object.assign()或者JSON.parse(JSON.stringfy(X))拷贝对象的方法。但事实证明都不合适：
1. Object.assign()只是浅拷贝（只是一级属性复制）对于层次很深的数据无法完美拷贝；
2. JSON.parse(JSON.stringfy(X))在序列化JavaScript对象时，所有函数和原型成员会被有意忽略，X只能是Number, String, Boolean, Array, 扁平对象，即那些能够被 JSON 直接表示的数据结构。  


### 解决方案
最后，我选择自己实现一种深拷贝的方法。   

### 基本思路
1. 该方法有两个参数，分别为指定格式的默认数据对象、接口返回的数据对象；  
2. 返回值是一个合并后的新对象；
3. 合并的逻辑是遍历默认数据对象的所有可枚举属性，分别在接口返回的对象上搜索该属性，若存在且值不为空（不为undefined 或 空字符串）则覆盖默认值；  
4. 在遇到遍历的属性是对象时，会进一步递归调用深拷贝方法；
5. 会根据默认的数据对象的constructor来判断结果是数组还是一般对象。

### 具体实现

```
// 深度拷贝
export const deepClone = function (target, source) {
  if (typeof target !== 'object') {
    return target
  }
  const res = target.constructor === Array ? [] : {}
  for (const key in target) {
    if (source.hasOwnProperty(key) && typeof source[key] !== 'undefined' && source[key] !== '') {
      if (typeof source[key] === 'object') {
        res[key] = deepClone(target[key], source[key])
      } else {
        res[key] = source[key]
      }
    } else {
      res[key] = target[key]
    }
  }
  return res
}
```

### 测试
```
const obj1 = {
  a: 1,
  b: [1, 2, 3],
  c: {
    c1: 1,
    c2: 2
  },
  d: [
    {
      name: 'd1',
      id: 1
    },
    {
      name: 'd2',
      id: 2
    },
    {
      name: 'd3',
      id: 3
    }
  ],
  e: {
    e1: {
      name: 'e1',
      uid: 3
    },
    e2: [5, 6, 7],
    e3: 4
  }
}

const obj2 = {
  a: 5,
  b: [11, 22, 33],
  c: {
    c2: 20
  },
  d: [
    {
      name: 'new'
    }
  ],
  e: {
    e1: {
      uid: 13
    },
    e2: [15, 16, 17]
  }
}

deepClone(obj1, obj2)
```

测试结果：成功，返回的对象是我所期望的数据结构。



