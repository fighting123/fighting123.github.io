---
title: 基于Express&&MongoDb&&vue全家桶&&axios搭建个人博客笔记(一)
tags: [博客前后台, express个人博客, 笔记]
copyright: true
date: 2018-03-21 20:24:09
categories: 个人项目笔记
---
> - 项目前端代码地址：https://github.com/fighting123/my_blog_FE
> - 项目后台代码地址：https://github.com/fighting123/my_blog_BE
### 1.前后端分离产生的跨域问题：
#### 解决方法一：
在webpack（config/index.js）的修改为：
    
```
proxyTable: {
    '/api': {
        target: 'http://192.168.212.23:9898',
        changeOrigin: true,
        pathRewrite: {
          '^/api': '/'
        }
    }
}
```
相应的globalOptions的baseUrl也设置为:
```
 baseURL: '/api',
```
整个设置意思为：每次遇到/api的访问请求时，就会在proxyTable中自动代理到target的值中（也就是示例中的http://192.168.212.23:9898），并且在发起请求中根据pathRewrite的设置，修改请求路径（示例中是将所有的/api替换为/，也就是直接请求后台，避免后台要给每个接口添加/api字段）
#### 解决方法二：
使用koa2-cors中间件：

```
var koa = require('koa');
var app = new koa();
var router = require('koa-router')();
// CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。
// 下面以koa2-cors为例，
const cors = require('koa2-cors');

// 具体参数我们在后面进行解释
app.use(cors({
    origin: function (ctx) {
        if (ctx.url === '/test') {
            return "*"; // 允许来自所有域名请求
        }
        return 'http://localhost:8080'; / 这样就能只允许 http://localhost:8080 这个域名的请求了
    },
    exposeHeaders: ['WWW-Authenticate', 'Server-Authorization'],
    // 该字段可选，用来指定本次预检请求的有效期，单位为秒。
    // 当请求方法是PUT或DELETE等特殊方法或者Content-Type字段的类型是application/json时，服务器会提前发送一次请求进行验证
    // 下面的的设置只本次验证的有效时间，即在该时间段内服务端可以不用进行验证
    maxAge: 5,
    // 该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。
    // 当设置成允许请求携带cookie时，需要保证"Access-Control-Allow-Origin"是服务器有的域名，而不能是"*";
    credentials: true,
    allowMethods: ['GET', 'POST', 'DELETE'],
    allowHeaders: ['Content-Type', 'Authorization', 'Accept'],
}))

router.post('/', async function (ctx) {
    ctx.body = '恭喜 __小简__ 你成功登陆了'
});

app
    .use(router.routes())
    .use(router.allowedMethods());

app.listen(3000);

```
*****注意：如果前端用的axios，并且headers设置的代码如下（这时浏览器会先以options的方式请求后台，成功才会继续使用get，post请求接口，如果后台不增加设置allowMethods和allowHeaders字段，options这一步就会产生跨域问题，导致后续无法请求成功）：**
```
const globalOptions = {
  withCredentials: true,
  baseURL: 'http://192.168.212.23:9898',
  timeout: 60000,
  headers: {
    'axios-header': 'axios'
  }
};
```
**那么后台应该对应的添加字段，代码如下：**
```
 {
    ......
    allowMethods: ['GET', 'POST', 'DELETE', 'OPTIONS'],
    allowHeaders: ['Content-Type', 'Authorization', 'Accept', 'axios-header']
 }
```

再具体的相关解释参考[node.js应答跨域请求实现（以koa2-cors为例）](https://www.jianshu.com/p/5b3acded5182)

#### 解决方法三：
其他方法参考[前端跨域问题](https://www.jianshu.com/p/88271c0b88bf?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
### 2.请求到后304
在请求上添加事件戳(plugins/index.js)

```
import axios from 'axios'
function convertURL (url) {
  // 获取时间戳
  let timstamp = (new Date()).valueOf()
  // 将时间戳信息拼接到url上
  if (url.indexOf('?') >= 0) {
    url = url + '&t=' + timstamp
  } else {
    url = url + '?t=' + timstamp
  }
  return url
}
export default {
  post (url, data, config) {
    return axios.post(convertURL(url), data, config).then((res) => {
      return res
    })
  },
  get (url, params, config) {
    return axios.get(convertURL(url), config).then((res) => {
      return res
    })
  }
}

```
### 3.对于未登录与登录后看到页面显示不同的问题
用vuex控制，store存储一个loginStatus的变量，默认false，点击登录则为true，点击登出变为false
### 4.对于路由跳转问题
登录之后不能在进入登录页，未登录不能发表文章，删除文章等功能的控制，可以用router.beforeEach来判断_id是否存在，这个_id是每次登录成功后后台返回的用户的id，每次登出时候将其清除
```
router.beforeEach((to, from, next) => {
  // 如果localStorage没有_id则表示未登录，强制跳转到登录页
  if (!localStorage.getItem('_id')) {
    if (to.path !== '/signUp' && to.path !== '/signIn') {
      next('/signIn')
    } else {
      next()
    }
  } else {
    if (to.path === '/signIn') {
      next('/')
    } else {
      next()
    }
  }
})
```
### 5.关于session
在每次登录请求时候，后台返回过期时间，前台根据这个时间来设置localstorage的过期时间（localstorage本身不会失效，没有过期时间，因此需要手动设置），返回的过期时间需要和config中设置的maxAge的session过期时间一致