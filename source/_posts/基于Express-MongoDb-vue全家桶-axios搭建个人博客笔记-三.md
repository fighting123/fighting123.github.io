---
title: 基于Express&&MongoDb&&vue全家桶&&axios搭建个人博客笔记(三)
tags:
  - 博客前后台
  - express个人博客
  - 笔记
copyright: true
categories: 个人项目笔记
abbrlink: 1580152229
date: 2018-04-01 18:22:45
---
> - 项目前端代码地址：https://github.com/fighting123/my_blog_FE
> - 项目后台代码地址：https://github.com/fighting123/my_blog_BE
### 关于主页点击跳转到详情页无法获取id问题
点击具体的文章时，无法获取到此文章的id，除非分析Dom结构，但是Vue是基于数据驱动的，应对充分尊重此点。所以在v-for生成文章列表的时候添加点击事件，将本身点击文章标题触发的事件放到整个组件上，这个时候是可以轻松获取的具体点击哪个文章的id的
```
<el-main>
      <div v-for="(message, index) in messageList" :key="index" @click.once="detailHandle(message._id)">
        <Posts :message="message" @getOnePost="showOnePost" @getPostList="showList"></Posts>
      </div>
    </el-main>
```
但是这个要注意得事子组件posts内元素的所有点击事件要加上事件修饰符.stop防止事件冒泡触发详情事件。
### 对于详情页和主页的复用问题：
1. 详情页和主页复用一个页面，但是刷新是总是存在头像不对应或者文章文章标题可以二次点击等各种问题，之后使用个比较笨的方法，在posts组件中路由变化时都强制刷新页面，这样数据可以重新渲染
++*(忽然想到vue的计算属性computed，这样既解决了img的src不能自动计算而不更新图片的问题，而且不用强制刷新页面)*++
2. 
```
computed: {
      imgSrc: function () {
        return `/api/image/${this.message.author.avatar}`
      }
    },
```

2. 为了详情页面重新刷新时仍停留在详情页面并且还是显示之前的文章页，可以这么处理：在跳转到详情页时将此文章的id加载路由中，这样页面刷新就可通过this.$route.params.id获取他的id了