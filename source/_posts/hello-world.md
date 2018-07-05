---
title: Hello World
categories: initTest1
tags: test1
---

## 博客搭建步骤简记
1. 按照教程配好基础的博客并修改个人信息
2. 更换主题为next
3. 添加分类。标签功能
4. 添加搜索功能，next官网上有很多第三方搜索服务，我用的是Local Search，具体可以按照教程来
5. 添加统计，同上，我用的不蒜子
6. 添加评论功能，同上，我用的是DISQUS(参考：https://www.jianshu.com/p/d68de067ea74?open_source=weibo_search)
7. 首页文章以摘要形式显示：将主题设置的auto_excerpt的enable为true
8. 设置首页文章显示篇数(参考：http://www.jeyzhang.com/next-theme-personal-settings.html)
9. 设置404页面,用的是腾讯公益404页面(直接在站点的source目录下新建404.html)
10. 设置头像(两张存储位置对应不用的文件夹名称，需要注意，然后将主题的avatar的注释放开并且填写对应位置)
11. 解决github+Hexo的博客多终端同步问题(https://www.jianshu.com/p/6fb0b287f950)

发布文章要用gitbash，用终端报错，很神奇
发布文章完成后记得上传到git的hexo分支，保存源代码