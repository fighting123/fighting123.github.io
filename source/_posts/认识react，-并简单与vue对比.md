---
title: 认识react， 并简单与vue对比
date: 2018-11-19 13:15:36
categories: React
tags: [react, react与vue对比]
copyright : true
---
### 应用场景：
1. 负责场景下的高性能
2. 重用组件库，组件组合

中文官网：https://reactjs.org.cn/doc/installation.html

### 特点：
1. 声明式编码(不需要关心如何实现，只需要关注在哪里做什么)
2. 组件化编码
3. 高效的DOM Diff，最小化页面重绘
4. 单向数据流


### 创建一个新的app：

```
npm install -g create-react-app
create-react-app my-app

cd my-app
npm start
```

-  使用 Yarn 安装 React：


```
yarn init
yarn add react react-dom
```

-  使用npm来安装 React：


```
npm init
npm install --save react react-dom
```

### 使用antd：

根据这个搭建环境: https://ant.design/docs/react/use-with-create-react-app-cn



### react和vue一样：
-  结合各自的生态库构成MVC框架

### react和vue不一样：
-  vue双向绑定，react单项绑定
-  react每次安装新包需要重新npm install，否则会报错：
```
    'react-app-rewired' 不是内部或外部命令，也不是可运行的程序或批处理文件。
```
-  生态库：

    vue: Vue + Vue-Router + VueX + Axios + Babel + Webpack
    
    react: React + React-Router + Redux + Axios + Babel + Webpack

 

### react-router:


线上学习react地址：https://reacttraining.com/react-router/web/example/auth-workflow
**注**：如果要每个路由都是新的页面不包含上个页面，就添加exact


- hashHistory

 通过 hash 进行对应。好处是简单易用，不用多余设定。



- browserHistory

 适用于服务器端渲染，但需要设定服务器端避免处理错误。注意的是若使用 Webpack 开发用,服务器需加上 --history-api-fallback

    $ webpack-dev-server --inline --content-base . --history-api-fallback



- createMemoryHistory
 主要用于服务器渲染，使用上会建立一个存在记忆体的 history 物件，不会修改浏览器的网址位置。
    const history = createMemoryHistory(location)