---
title: hexo搭建简记
date:
  '[object Object]': null
categories: Hexo
tags:
  - hexo
  - Next
copyright: true
abbrlink: 266327918
---

## 博客搭建步骤简记

网上基于hexo搭建博客的教程非常多，本文只是结合自己博客的需求对网上的文章进行了过滤和总结，大致可以囊括常用的所有功能

1. 按照教程配好基础的博客并修改个人信息，可参考：

    https://www.jianshu.com/p/d49e4684e62b

    https://hexo.io/zh-cn/docs/configuration.html

    https://www.jianshu.com/p/9f0e90cc32c2
    
    https://www.jianshu.com/p/f054333ac9e6(酷炫效果的总结)
    
2. 更换主题为next（参考1）
3. 添加分类。标签功能（参考1）
4. 添加搜索功能，next官网上有很多第三方搜索服务，我用的是Local Search，具体可以按照教程来
5. 添加统计，同上，我用的LeanCloud和不蒜子
6. 添加评论功能，DISQUS官网进不去，所以我用的是LeanCloud
7. 首页文章以摘要形式显示：将主题设置的auto_excerpt的enable为true
8. 设置首页文章显示篇数(参考：http://www.jeyzhang.com/next-theme-personal-settings.html)
9. 设置404页面,用的是腾讯公益404页面(直接在站点的source目录下新建404.html)
10. 设置头像(两张存储位置对应不用的文件夹名称，需要注意，然后将主题的avatar的注释放开并且填写对应位置)
11. 设置icon，在阿里妈妈矢量库寻找合适图像，下载16 * 16和32 * 32的png格式，将next主题下的favicon图片更换掉
12. 解决github+Hexo的博客多终端同步问题(https://www.jianshu.com/p/6fb0b287f950)
13. 剩下的各种详情都参考第一条（有时间在统一整理）
14. 添加动态背景（https://huur.cn/res/413.html）注意：这篇文章的写法有问题，更正之后可以运行，代码如下：
    ```
    {% if (theme.canvas_nest) %}
        <script type="text/javascript" color="0,0,0" opacity='0.4' zIndex="-2" count="66" src="//cdn.bootcss.com/canvas-nest.js/1.0.0/canvas-nest.min.js"></script>
    {% endif %}
    ```
    并把themes/next/_config.yml文件下的两个canvas_nest都设置为true即可
15. 社交连接等icon都是出自fontawesome的官网https://fontawesome.com/?from=io，在这里寻找合适的icon在config里设置即可
16. 添加点击出现小心心（https://asdfv1929.github.io/2018/05/25/baidu-share/）,效果没有透明度不美观，我在源码中将透明度改成了0.8
17. 之前文章的图片都是存放在七牛云上，最近七牛云的测试域名全部回收，还好我的图片有备份（坑），索性直接新建个git仓库来专门存放文章的图片（省的在糟心），每个图片点击Download就可以获取链接了
最后的最后，要注意得是：
- 发布文章要用gitbash，用终端会报错

- 发布文章完成后记得上传到git的hexo分支，保存源代码