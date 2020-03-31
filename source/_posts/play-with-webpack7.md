---
title: 玩转 webpack (七)
date: 2020-03-30 17:54:48
tags:
- review

categories:
- Webpack
---

## loader执行顺序

### 简易的loader代码结构

定义：loader只是一个导出为函数的Javascript模块。
```
module.exports = function (source) {
  return source;
}
```

<!--more-->

### 多loader执行顺序

**多个loader串行执行（loader处理完，传递给下一个loader），顺序从后往前。**

执行顺序的原因：webpack采用compose的函数组合方式
```
compose = (f, g) => (...args) => f(g(...args));
```

### 创建一个loader

使用webpack-cli generate-loader

------------------------------------

## 高效的loader调试方法

使用loader-runner，提供loader运行环境，可以在不安装webpack的情况下运行loaders。

**示例：开发一个raw-loader**

loader文件内容：raw-loader.js
```
module.exports = function (source) {
  const json = JSON.stringify(source)
    .replace(/\u2028/g, '\\u2028')
    .replace(/\u2029g, '\\u2029'/);

  return `export default ${json}`
}
```

调试loader，run-loader.js
```
const fs = require('fs');
const path = require('path');
const { runLoaders } = require('loader-runner');

runLoaders(
  {
    resource: './demo.txt', // 需要处理的源码
    loaders: [path.resolve(__dirname, './loader/raw-loader')],
    readResource: fs.readFile.bind(fs)
  },
  (err, result) => { err ? console.error(err) : console.log(result) }
)
```

运行查看
```
node run-loader.js
```

------------------------------------

## 复杂的loader开发场景

### loader的参数获取

通过loader-utils的getOptions获取
```
const loaderUtils = require('loader-utils');

module.exports = function (content) {
  const { name } = loaderUtils.getOptions(this);
}
```

### loader的异常处理

处理方案：
* loader内通过throw抛出；
* 通过this.callback传递错误；
```
this.callback(
  err: Error | null,
  content: string | Buffer,
  sourceMap?: SourceMap,
  meta?: any
)
```

### loader的异步处理

通过this.async来返回一个异步函数。（第一个参数是Error，第二个参数是处理的结果）
```
module.exports = function (input) {
  const callback = this.async();

  callback(null, input + input);
}
```

### 在loader中使用缓存

**webpack默认开启loader缓存。（可以使用this.cacheable(false)来关掉缓存）**

缓存条件：
* loader的处理结果在相同输入下有确定的输出；
* 有依赖的loader无法使用缓存；

### loader如何进行文件输出

通过this.emitFile进行文件写入
```
this.emitFile(url, content);
```

------------------------------------

## 实战：开发一个loader自动合成雪碧图

支持的语法：
```
background: url('a.png?__sprite');

// 转换后
background: url('sprite.png')
```

合成图片的工具：spritesmith
```
const sprites = ['./image/1.jpg', './image/2.jpg'];

Spritesmith.run({ src: sprites }, function handleResult (err, result) {
  result.image;
  result.coordinates;
  result.properties;
})
```

[demo仓库](https://github.com/HarryYin91610/sprite-loader-demo)

------------------------------------

## 插件的基本结构

插件没有loader那种独立的运行环境，只能在webpack里运行。

基本结构：
```
class MyPlugin { // 插件名称
  apply (compiler) { // 每个插件必须要有apply方法
    compiler.hooks.done.tap('MyPlugin', (stats) => {
      console.log('Hello World'); // 插件逻辑处理
    });
  }
}

module.exports = MyPlugin;
```

插件使用：
```
module.exports = {
  plugins: [new MyPlugin()]
}
```

------------------------------------

## 复杂的插件开发场景

### 插件通过其构造函数获取参数

```
module.exports = {
  constructor (options) {
    this.options = options;
  }
}
```

### 插件的错误处理

* 参数校验阶段可以直接throw抛出
```
throw new Error('Error Message');
```
* 通过compilation对象的warnings和errors接收
```
compilation.warnings.push('warning');
compilation.errors.push('error');
```

### 文件写入
* 通过Compilation上的assets进行文件写入
* 文件写入需要使用webpack-sources

示例：
```
const { RawSource } = require('webpack-sources');

module.exports = class DemoPlugin {
  constructor (options) {
    this.options = options;
  }

  apply (compiler) {
    const { name } = this.options;
    compiler.hooks.emit('plugin', (compilation, cb) => {
      compilation.assets[name] = new RawSource('demo');
      cb();
    });
  }
};
```

### 插件扩展：插件的插件

插件自身可以通过暴露hooks进行自身扩展

示例：html-webpack-plugin
```
html-webpack-plugin-after-chunks
html-webpack-plugin-before-html-generation
html-webpack-plugin-after-assets-tags
```

------------------------------------

## 实战：开发一个压缩构建资源生成zip包的插件

**要求：**
* 生成的zip包名可以通过插件传入；
* 需要使用compiler对象上特定的hook进行资源生成；

### 准备阶段

#### 使用jszip将文件压缩为zip包
```
var zip = new JSZip();

zip.file('hello.txt', 'Hello World\n');

var img = zip.folder('images');
img.file('smile.gif', imgData, {base64: true});
```

#### emit——Compiler上负责文件生成的hook

emit是一个异步hook，读取的是compilation.assets对象的值。
（我们可以将zip资源包设置到compilation.assets对象上）

[deme仓库](https://github.com/HarryYin91610/zip-plugin)

