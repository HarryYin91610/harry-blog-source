---
title: play-with-webpack3
date: 2020-03-22 22:39:33
tags:
- review

categories:
- Webpack
---

## 自动清理构建目录

### 方法一：通过npm scripts清理构建目录
```
rm -rf ./dist && webpack
```

### 方法二：使用clean-webpack-plugin
**默认会删除output指定的输出目录**

安装clean-webpack-plugin
```
npm i -D clean-webpack-plugin
```

webpack.config.js配置：
```
module.exports = {
  plugins: [
    new CleanWebpackPlugin()
  ]
}
```

-----------------------------------------
 
## 自动补齐CSS3前缀

**使用postcss-loader和autoprefixer插件：打包css后置处理器**

安装插件
```
npm i postcss-loader autoprefixer -D
```

webpack.config.js配置：
```
module.exports = {
  module: {
    rules: [
      {
        test: /.less$/,
        use: [
          MiniCssExtractPlugin.loader, // 后执行
          'css-loader',
          'less-loader', // 先执行
          {
            loader: 'postcss-loader',
            options: {
              plugins: () => [
                require('autoprefixer')({
                  // last 2 version：最近两个版本，>1%：指浏览器使用人数占比，ios 7：ios7以上版本
                  browsers: ['last 2 version', '>1%', 'ios 7']
                })
              ]
            }
          }
        ]
      }
    ]
  }
}
```

-----------------------------------------
 
## px自动转换成rem

* rem：font-size of the root element
* rem是相对单位，px是绝对单位

**使用px2rem-loader，在页面渲染时动态计算根元素font-size值。（成熟方案：手淘的lib-flexible库）**

安装px2rem-loader、lib-flexible：
```
npm i px2rem-loader -D
npm i lib-flexible -S
```

webpack.config.js配置：
```
module.exports = {
  module: {
    rules: [
      {
        test: /.less$/,
        use: [
          MiniCssExtractPlugin.loader, // 后执行
          'css-loader',
          'less-loader', // 先执行
          {
            loader: 'postcss-loader',
            options: {
              plugins: () => [
                require('autoprefixer')({
                  // last 2 version：最近两个版本，>1%：指浏览器使用人数占比，ios 7：ios7以上版本
                  browsers: ['last 2 version', '>1%', 'ios 7']
                })
              ]
            }
          },
          {
            loader: 'px2rem-loader',
            options: {
              remUnit: 75, // 表示1rem = 75px， 视觉稿750px
              remPrecision: 8 // 保留小数点的位数
            }
          }
        ]
      }
    ]
  }
}
```

**注意：需要在head引入lib-flexible动态计算font-size**

-----------------------------------------
 
## 静态资源内联至html

资源内联的意义：
* 减少http请求（小图片或字体：url-loader）；
* 页面框架初始化脚本；
* 上报埋点脚本；
* css内联避免页面闪动；

## html和js内联

使用raw-loader
* 内联html片段（例如移动端meta片段）
* 内联js脚本（例如lib-flexible）

安装raw-loader
```
npm i raw-loader@0.5.1 -D
```

在html内：
```
<head>
  ${ require('raw-loader!./meta.html') }
  <title>Document</title>
  <script>${ require('raw-loader!babel-loader!../../node_modules/lib-flexible/flexible.js') }</script>
</head>
```
**注意：如果js语法使用了es6或更新的语法需要使用babel-loader**

## css内联

### 方法一：借助style-loader
```
module.exports = {
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          {
            loader: 'style-loader',
            options: {
              insertAt: 'top', // 样式插入到head
              singleton: true // 将所有style标签合成一个
            }
          },
          'css-loader',
          'sass-loader'
        ]
      }
    ]
  }
}
```

### 方法二：html-inline-css-webpack-plugin

将打包好的css文件插入到html对应的位置