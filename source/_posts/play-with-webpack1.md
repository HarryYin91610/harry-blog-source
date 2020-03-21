---
title: 玩转 webpack (一)
date: 2020-03-20 15:44:05
tags:
- review

categories:
- Webpack
---

## 安装webpack

### 安装nvm（node版本管理工具）
```
两种方式：
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

### 安装nodejs和npm
```
nvm install v10.15.3

检查版本：
node -v
npm -v

列举安装的node所有版本：
nvm list
```

<!--more-->

### 创建空目录和package.json
-y：默认都选择yes
```
mkdir project1
cd project1
npm init -y
```

### 安装webpack
```
npm i -D webpack webpack-cli

查看版本：
./node_modules/.bin/webpack -v
```

## webpack配置组成
```
module.exports = {
  entry: './src/index.js, // 打包入口文件
  output: './dist/main.js', // 打包的输出
  mode: 'production', // 环境
  module: {
    rules: [
      {
        test: /\.txt$/, use: 'raw-loader' // loader配置
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html' // 插件配置
    })
  ]
}
```

## 运行webpack

构建命令：
```
npm run build
```

原理：模块局部安装会在node_modules/.bin 目录下创建软链接
```
package.json:
{
  ...,
  "scripts": {
    "build": "webpack"
  }
  ...
}
```
