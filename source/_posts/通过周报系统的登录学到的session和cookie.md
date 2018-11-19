---
title: 通过周报系统的登录学到的session和cookie
date: 2018-04-15 11:35:36
categories: koa2
tags: [session和cookie, 用户免登陆]
copyright : true
---
## 关于session和cookie基础介绍：

### session和cookie关系
session是基于cookie实现的,session在被创建后,会生成一个**唯一的**sessionid返回给浏览器， 并保存在浏览器的cookie中,接下来客户再次调用服务端接口，服务器便会从客户端发送过来的cookie中查找name为sessionid的cookie是否存在,若是存在则通过该cookie的值来找到用户之前创建的session,若是不存在则创建一个新的session。

然而保存sessionId的cookie默认是会话级别的,是保存在浏览器的内存中的,当浏览器关闭时这个cookie也就消失了,所以再次打开一个新的浏览器由于这个时候并不存在名为sessionid的cookie,所以服务器便会创建一个新的session,但是原来的session还是存在的!也就是说 **这时候服务器中一共存在两个session**。

**设置cookie的过期时间，必须关闭浏览器的随浏览器关闭而清除cookie的功能**

### cookie过期时间
但是cookie也可以设置过期时间，不设置的话默认随着浏览器的关闭而清楚，因为未设置过期时间的cookie只能存在浏览器，设置过期时间的话就会存到本地硬盘上，浏览器关闭就不影响cookie（除非到了过期时间它自动清除），这个时候当浏览器再次打开，会从本地读取cookie，发送给服务端，在重复执行过程一。**这个可用于一定时间内免用户登录的需求等**

### session过期时间
session也有过期时间，默认为20分钟，20分钟后自动清除session。当session过期后，浏览器发送过来的cookie就无法在服务器端找到对应的session，因为session已经不存在，此时需要重新登录并设置session，重复过程一


## 结合具体项目代码谈谈：

#### 项目具体代码
在周报登录中，使用的是`koa-session-minimal`和`koa-mysql-session`中间件，先贴代码：
```
const config = require('../config/default');        // 数据库相关的配置文件
const session = require('koa-session-minimal');         // 处理数据库中间件
const MysqlStore = require('koa-mysql-session');    // 处理数据库中间件

let createSession = (app) => {
  // session数据库存储配置
  const sessionMysqlConfig= {
    user: config.database.USERNAME,
    password: config.database.PASSWORD,
    database: config.database.DATABASE,
    host: config.database.HOST,
    port: config.database.PORT
  };

  // session/cookie 属性配置
  let sessionOptions = {
    key: 'session-id',              // cookie 中存储 session-id 时的键名, 默认为 koa:sess
    cookie: {                       // 与 cookie 相关的配置
      domain: '',                   // 写 cookie 所在的域名
      path: '/',                    // 写 cookie 所在的路径
      maxAge: 1000 * 60 * 10,       // cookie 有效时长(单位：ms)
      httpOnly: true,               // 是否只用于 http 请求中获取
      overwrite: true               // 是否允许重写
    },
    store: new MysqlStore(sessionMysqlConfig)
  };

  app.use(async (ctx, next) => {
    // 获取hostname，设置cookie的domain属性值
    sessionOptions.cookie.domain = ctx.request.hostname;
    await next();
  });

  app.use(session(sessionOptions));
};


module.exports = createSession;
```
在每次用户登录的时候，都会设置session：
```
ctx.session = {
    userId: '???'
};
```
存储session的时候就会通过这个文件中的配置，给设置的数据库中添加一个名字为`_mysql_session_store`的表并存储下面的数据：
```
id: 'session-id：自动生成的序列号'，expires： 'session设置的过期时间',data: '登录时存储的值'
```
同时向前端返回值为session-id的cookie，并存在客户端，每次请求时都会将cookie同时发送给服务端，我们只在每次请求时判断`ctx.session.userId`是否存在，能取到则是sessio存在且未过期，则可以继续执行相应操作

**注意：**

其实在取值时中间件已经帮我们做了很多工作：进行这一步时候中间件先是通过客户端发过来的cookie对应的值去找数据库中对应的数据，如果存在且未过期的话则返回对应的data，即本文中的userId

### 关于这个中间件中session和cookie的过期时间：

中间件中默认session过期时间为一天，cookie是随着浏览器关闭而关闭，只有在maxAge值大于0时，cookie和session的过期时间保持一致为`maxAge`的值。