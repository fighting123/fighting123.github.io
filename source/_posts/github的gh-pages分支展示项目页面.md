---
title: github的gh-pages分支展示项目页面
tags: [github, gh-pages]
copyright: true
date: 2018-11-21 10:08:15
categories: GitHub
---
## 引言
> github上的demo别人需要预览的时候都得clone下来运行才可以，不能外网访问，不利于demo效果的展示，其实将项目打包部署到GitHub Pages上就可以完美解决这个问题了。


## 用到的库

- [gh-pages](https://www.npmjs.com/package/gh-pages)
- 安装：`yarn add gh-pages`

## 部署过程

### 创建项目仓库

常规操作创建git仓库即可，可参考：[https://blog.csdn.net/zoucanfa/article/details/77725839](https://blog.csdn.net/zoucanfa/article/details/77725839)

### 在本地的项目文件下执行以下命令


```
$ git init
$ git add .
$ git commit -m 'message'
$ git remote add origin <url>
$ git push -u origin master
```

### 修改本地的package.json文件及相关配置

由于React项目和Vue项目打包后的文件夹不一样，所以配置也稍稍有点不同

- React：
    
```
"homepage": "https://fighting123.github.io/react_manage_system",
"scripts": {
    + "predeploy": "npm run build",   // 对应的deploy之前的钩子
    + "deploy": "gh-pages -d build"  // deploy名字可以随意
 },
    ```
    
- Vue:

```
"scripts": {
    + "predeploy": "npm run dist",  // 或者yarn run dist
    + "deploy": "gh-pages -d dist"
 },
```

      并修改config/index.js:
    
```
build: {
   assetsPublicPath: ''
}
```

### 部署

```
yarn deploy   // 或npm run deploy
```
*部署过程真的感觉超级慢。。。*

部署成功后，对应远程上就有新的gh-pages分支了，修改setting上的source为gh-pages分支，然后打开https://fighting123.github.io/react_manage_system即可看到对应的页面了。

## 总结
总体来看，它的原理其实很简单，就是在当前项目仓库下自动创建一个名为gh-pages的分支，打包部署成功之后上传到这个分支的正好就是build内的静态文件，其实不怕麻烦的同学也可以不用这个库，自己一步步创建分支，上传build文件也可以实现同样的效果！

## 遇到的问题及解决方法


1. 运行yarn deploy过程中可能会报错

    ```
    fatal: A branch named 'gh-pages' already exists.
    ```
    **官方文档上的解释是：**
    
       当处理gh-pages模块生成文件.cache，如果由于某些原因如密码错误等卡住则不会自动清理
    
    **解决办法**：
    
    运行 ~node_modules/gh-pages/bin/gh-pages-clean 或者直接删除项目下的 ~node_modules/gh-pages/.cache文件即可
2. 运行yarn deploy过程中可能会报错

    ```
    fatal: The remote end hung up unexpectedly
    ```
    **官方文档上的解释是：**

    通过 HTTP 传输 POST 数据到远程系统上的最大缓冲字节数 。当请求大于这个缓冲大小时，HTTP/1.1 和 Transfer-Encoding: chunked 用来避免在本地创建过多的压缩文件。默认是 1MiB，适用于大多数的请求
    
    **解决办法**：

    ```
    git config --global http.postbuffer 1048576000
    ```
3. 运行yarn deploy过程中可能会报错
    ```
    could not read Username for 'https://github.com': No error
    ```
    
    **解决办法**：
    
    
    修改.git下的config文件的url为https://用户名:密码@github.com/fighting123/react_manage_system.git即可
4. 多个html文件的项目，如官网，用下面方法：

    ```
    git symbolic-ref HEAD refs/heads/gh-pages
    git add -A
    git commit -m "描述"
    git push origin gh-pages
    ```
**参考文章：**
- https://www.jianshu.com/p/9dcc6e68031e
- https://www.cnblogs.com/MuYunyun/p/6082359.html
- https://www.douban.com/note/668373438/
- https://www.rails365.net/movies/react-ji-qiao-2-ba-react-ying-yong-bu-shu-dao-github-pages (把 react 应用部署到 GitHub Pages的视频)