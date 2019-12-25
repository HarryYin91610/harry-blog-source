---
title: JavaScript 基础知识复习（第三章）
date: 2019-12-08 15:20:55
tags:
- review

categories:
- JavaScript
---

## script元素常用属性
* async: （可选）表示加载和渲染后续文档元素的过程将和 script.js 的加载与执行并行进行（异步）。
* defer: （可选）表示加载后续文档元素的过程将和 script.js 的加载并行进行（异步），但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。
* type: （可选）默认text/javascript。

![async与defer比较](./javascript-review/script01.jpeg)

<!--more-->

**注意：没有 defer 或 async，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。**
**（defer和async只适用于外部脚本文件，会忽略嵌入的脚本设置的defer或async）**
**在现实中，defer的脚本不一定会按照顺序执行，也不一定在DOMContentLoaded事件触发之前执行，因此最好只包含一个defer脚本。**

--------------------------------------------

## 数据类型

### 基本数据类型
1. #### Undefined
2. #### Null：表示空对象指针
3. #### Boolean
4. #### Number：包括整数和双精度数值，保存浮点数的空间是整数的两倍
```
var octalNum = 070 // 八进制：第一位是0，其余位是0~7
var hexNum = 0xA // 十六进制：前两位是0x,其余是0~9及A~F
var floatNum = 3.125e7 // 等于3.125 * Math.pow(10, 7)
```
* 浮点数值最高精度是17位小数，计算存在舍入误差问题（基于IEEE754数值的浮点计算通病）[关于浮点数问题的解决方案](https://github.com/camsong/blog/issues/9)
```
0.1+0.2 === 0.3 // false
0.15+0.15 === 0.3 // true
```
* ECMAScript能表示的最小数值：Number.MIN_VALUE(5e-324)，ECMAScript能表示的最大数值：Number.MAX_VALUE(1.7976931348623157e+308)，超出范围：Infinity 或 -Infinity
```
var result = Number.MAX_VALUE + Number.MAX_VALUE
alert(isFinite(result)) // false
```
* NaN与任何值都不相等，包括自己本身
```
NaN === NaN // false
```
5. #### String
* 转换成字符串有两种方式：toString和String
* Null和Undefined没有toString()方法
* toString(x) 可以传参数x为转换的基数
```
var num = 10
alert(num.toString()) // 10
alert(num.toString(2)) // 1010
alert(num.toString(8)) // 12
alert(num.toString(10)) // 10
alert(num.toString(16)) // a
```
6. #### Symbol
* ES6 引入了一种新的原始数据类型Symbol，表示独一无二的值。它是 JavaScript 语言的第七种数据类型。  
* Symbol函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。
```
const s1 = Symbol('abc');
const s2 = Symbol('abc');
s1 === s2; // false
typeof s; // "symbol"
s1.description; // 直接返回描述：'abc'
```
* Symbol作为属性名
```
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"
```
7. #### BigInt
* BigInt数据类型的目的是支持比Number数据类型范围更大的整数值。
* BigInt 只用来表示整数，没有位数的限制，任何位数的整数都可以精确表示。
* BigInt 类型的数据必须添加后缀n
```
const a = 2172141653n;
const b = 15346349309n;
// BigInt 的运算
1n + 2n // 3n
0b1101n // 二进制
0o777n // 八进制
0xFFn // 十六进制
```
* BigInt 与普通整数是两种值，它们之间并不相等。
```
42n === 42 // false
typeof 123n // 'bigint'
```
* BigInt 可以使用负号（-），但是不能使用正号（+）
```
-42n // 正确
+42n // 报错
```
### 复杂数据类型
1. #### Object
* hasOwnProperty(name)：用于检查给定的属性名是否存在于当前对象中（不是在原型中）
* propertyIsEnumerable(name)：用于检查给定属性是否可使用for-in枚举
* valueOf()：返回对象的字符串、数值、布尔值表示（通常与toString返回值相同）

2. #### Set
* ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。
```
const s = new Set();

[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

for (let i of s) {
  console.log(i);
}
// 2 3 5 4
```
* 去除数组重复成员的方法
```
[...new Set(array)]
```
* 去除重复字符
```
[...new Set('ababbc')].join('')
```

3. #### Map
* ES6 提供了 Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。
```
const m = new Map();
const o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```
* Map 也可以接受一个数组作为参数。该数组的成员是一个个表示键值对的数组。
```
const map = new Map([
  ['name', '张三'],
  ['title', 'Author']
]);

map.size // 2
map.has('name') // true
map.get('name') // "张三"
map.has('title') // true
map.get('title') // "Author"
```
--------------------------------------------

### typeof操作符
* "undefined": 表示这个数值未定义，undefined派生自Null。
* "boolean"
* "string"
* "number"
* "object": 表示这个数值是对象或者Null
* "function"

**注意：typeof对未初始化和未声明的变量都返回"undefined"**
```
var message;
alert(typeof message); // "undefined"
alert(typeof age); // "undefined"
```

--------------------------------------------

## 一元操作符
### 位操作符：将64位的数值转化为32位进行计算，表示数值最底层操作，速度更快。
对于有符号的整数，32位前31位表示数值，第32位表示符号：0为正号，1为负号。
负数以二进制补码格式存储：先求这个数值绝对值二进制码，再求其反码（1和0互换），将得到的反码加1.
* #### 按位非(~)：返回数值的反码，等同于操作数的负值减1。
* #### 按位与(&)
* #### 按位或(|)
* #### 按位异或(^)：两个操作数对应位相同为0， 不同为1。
* #### 左移(<<)：不会影响符号位，空位填充0。
* #### 有符号的右移(>>)：保留符号位，用符号位的值填充空位。
* #### 无符号的右移(>>>)：空位用0来填充
```
var oldv = 2; // 二进制：10
var newv = oldv << 5; // 二进制：1000000，等于十进制64
```