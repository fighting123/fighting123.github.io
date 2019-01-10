---
title: hexo基本操作
date:
  '[object Object]': null
categories: Hexo
tags:
  - hexo
copyright: true
abbrlink: 3662755565
---

### 创建文章

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### 本地运行

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### 编译

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### 发布

``` bash
$ hexo deploy
```
### 同步项目源文件到Github
```
// 添加源文件
git add .
// git提交
git commit -m ""
// 先拉原来Github分支上的源文件到本地，进行合并
// 分支名后面的“--allow-unrelated-histories”是为了弹出“fatal: refusing to merge unrelated histories.”的错误
git pull origin 分支名 --allow-unrelated-histories
// 比较解决前后版本冲突后，push源文件到Github的分支
git push origin 分支名

```
