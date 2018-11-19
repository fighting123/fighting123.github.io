---
title: 基于Express&&MongoDb&&vue全家桶&&axios搭建个人博客笔记(二)
tags: [博客前后台, express个人博客, 笔记]
copyright: true
date: 2018-03-28 16:14:07
categories: 个人项目笔记
---
> - 项目前端代码地址：https://github.com/fighting123/my_blog_FE
> - 项目后台代码地址：https://github.com/fighting123/my_blog_BE

花了好几天时间终于解决了这个可能在别人看来十分简单的问题，毕竟第一次上手后台，很多东西还是不懂得，加油！先来一个参考链接：
https://www.cnblogs.com/thingk/archive/2013/11/25/3434032.html
## 首先是注册页面的图片上传

用的是element的上传组件，action参数是图片上传的地址，本身是想和其他用户注册时的其他参数一齐上传，但是这是有问题的，图片上传应该是需要一个特殊的接口，将图片保存到服务器，然后将这个图片的地址保存到数据库中，每次需要时通过数据库的地址查找对应的图片，代码如下：

前端代码：
```
    <el-upload
    ref="upload"
    class="avatar-uploader"
    action="http://localhost:8080/api/signup/image"
    name="avatar"
    :show-file-list="false"
    :auto-upload="false"
    :on-change="handleAvatarSuccess"
    :before-upload="beforeAvatarUpload">
    <img v-if="imageUrl" :src="imageUrl" class="avatar">
    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
</el-upload>
```
关于imgUrl：

```
handleAvatarSuccess (res, file) {
    this.imageUrl = URL.createObjectURL(file[0].raw)
    this.signUpForm.avatar = file[0]
},
```

需要在注册提交事件中触发：

```
vm.$refs['upload'].submit()
```

后台代码：


```
// 先把图片存储起来
router.post('/image', function (req, res, next) {
  res.send({status: 'success'})
  console.log(req.fields.avatar)
})
// 注册接口
router.post('/', checkNotLogin, function (req, res, next) {
  // 读取image文件夹取files第一个为刚才注册所用的图片
  fs.readdir('public/image', function (err, files) {
    if (err) {
      console.log(err);
      return;
    }
    // var imagePathName = ''
    // imagePathName = path.join(path.resolve(__dirname, '..'), '/public/image/', files[0])
    var name = req.fields.name
    var gender = req.fields.gender
    var bio = req.fields.bio
    var password = req.fields.password
    var repassword = req.fields.repassword

    // 明文密码加密
    password = sha1(password);
    // 待写入数据库的用户信息
    var user = {
      name: name,
      gender: gender,
      bio: bio,
      avatar: files[0],
      password: password,
      repassword: repassword
    }
    // 用户信息写入数据库
    UserModel.create(user)
      .then(function (result) {
        // 此 user 是插入 mongodb 后的值，包含 _id
        user = result.ops[0]
        // 删除密码这种敏感信息，将用户信息存入 session
        delete user.password
        req.session.user = user
        res.send({status: 'success', message: '注册成功'})
      })
      .catch(function (e) {
        // 注册失败，异步删除上传的头像
        // fs.unlink(imagePathName)
        // 用户名被占用则跳回注册页，而不是错误页
        if (e.errmsg.match('duplicate key')) {
          // req.flash('error', '用户名已被占用')
          res.send({status: 'error', message: '用户名已被占用'})
        }
        // next(e)
      })
  })
})
```
在后台的index.js文件下添加中间件：

```
// 处理表单及文件上传的中间件
app.use(require('express-formidable')({
  uploadDir: path.join(__dirname, 'public/image'), // 上传文件目录
  keepExtensions: true // 保留后缀
}))
```
## 第二部分是部分需要显示图片的页面
前端代码：

在登录时将imgUrl地址存起来，以方便后面页面请求需要：

```
localStorage.set('imgUrl', res.data.imgUrl, res.data.expTime)
```
需要获取图片的页面：
```
<img :src="imgSrc" alt="">
```
图片src设置为请求的接口：
```
created() {
    this.imgSrc = `/api/image/${localStorage.get('imgUrl')}`
}
```
后台代码：

添加路由：

```
module.exports = function (app) {
  app.get('/api/image/:url', function (req, res) {
    // 获取到图片的在后台的完整路径，否则拿不到图片
    var imagePathName = ''
    imagePathName = path.join(path.resolve(__dirname, '..'), '/public/image/', req.params.url)
    // 读取图片
    fs.readFile(imagePathName, 'binary', function (err, data) {
      if (err) {
        res.writeHead(500,{"Content-Type":"text/plain"});
        res.write(error+"\n");
        res.end();
      } else {
        res.writeHead(200,{"Content-Type":"image/png"});
        // 不能用send，否则会浏览器报错：504 (Gateway Timeout)
        // res.send(data, "binary");
        res.end(data, 'binary');
      }
    })
  })
  }
```

