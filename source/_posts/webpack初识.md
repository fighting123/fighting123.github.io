---
title: webpack初识
date: 2018-03-19 16:57:00
categories: webpack
tags: [打包, Webpack]
copyright : true
---
## 安装webpack

```
//全局安装
npm install -g webpack
//安装到项目目录
npm install --save-dev webpack
//全局安装（同时需要安装webpack-cli，否则提示The CLI moved into a separate package: webpack-cli.
）
npm install webpack-cli -g
//安装到项目目录
npm install --save-dev webpack-cli
```

区别：安装到项目目录运行时需要加上在node_modules中的地址，如：node_modules/.bin/webpack

## 使用webpack前的准备工作
### 创建如图所示的文件：
![目录结构](https://raw.githubusercontent.com/fighting123/hexo_images/master/oldArticleImages/webpack%E5%88%9D%E8%AF%861.png)

首先用npm init生成package.json文件，然后创建文件，public是存放浏览器读取的文件，app是存放未打包前的模块文件

Greeter.js模块:

```
const greeter = () => {
  let greet = document.createElement('div')
  greet.textContent = 'hello'
  return greet
}

module.exports = greeter()
```
main.js引入模块的文件：

```
let greeter = require('./Greeter')
document.querySelector("#body").appendChild(greeter)
```
index.html主页面:

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>webpack_sample_practice</title>
</head>
<body id="body">
</body>
<script src="bundle.js"></script>
</html>
```
bundle.js打包后存储的文件（index.html引入的文件）
## （一）使用命启动令webpack

```
webpack app/main.js public/bundle.js // webpack 入口文件 打包后文件
```
这时会提示warning，这是因为没有设置mode,所以需要：
```
webpack app/main.js public/bundle.js --mode development（未压缩）
// 或者
webpack app/main.js public/bundle.js --mode production（压缩过）
```
为方便起见，我们将这两句命令放入package.json文件中：
```
"scripts": {
    "dev": "webpack app/main.js public/bundle.js --mode development",//（未压缩）
    "build": "webpack app/main.js public/bundle.js --mode production" //（压缩过）
  },
```
这样，在每次启动是只需要输入npm run dev 或者npm run build

最后，打开浏览器页面就可以访问啦！

## （二）通过配置webpack.config.js来使用webpack
我们也可以通过更加方便简洁的方式使用webpack，这样也比较不容易出错。

在主目录新建webpack.config.js配置文,先简单的只配置下入口和出口文件目录:

```
module.exports = {
  entry:  __dirname + "/app/main.js",//已多次提及的唯一入口文件
  output: {
    path: __dirname + "/public",//打包后的文件存放的地方
    filename: "bundle.js"//打包后输出文件的文件名
  }
}
```
同意为方便我们配置package.json文件：

```
"scripts": {
    "dev": "webpack --mode development",
    "build": "webpack --mode production"
  },
```
输入npm run dev 或者 npm run build同样可以正常启动啦！

[参考：https://www.jianshu.com/p/42e11515c10f](https://www.jianshu.com/p/42e11515c10f)