---
title: node-login-segmentfault
date: 2018-07-10 14:14:07
tags:
---
## 前言 ##
前段时间看的爬虫都是不需要登录直接爬取数据，这回就试试爬取需要登录的网站信息吧，说干就干，直接就拿segmentfault做为目标！

## 一、爬虫所需模块 ##

 - superagent
 - async
## 二、分析 ##
我们首先用chrome或者其他浏览器打开segmentfault的主页，找到它的登录接口，点击登录接口，记得把Preserve log勾上，否则跳转之后找不到接口，如图（重要信息打了马赛克）：

![图片描述][1]

显而易见这是一个post请求，三个参数分别是用户名、密码、和是否记住密码的标记。
### 获取Cookie里的PHPSESSID ###
仔细看请求头`header`里的`cookie`，在请求发送的时候就已经有了，我们直接把整个请求头拿过来，就直接用图中的接口和`header`登录，逐一删除`cookie`中的项，测试登录结果，最终发现只有`PHPSESSID`是必须的。这个`PHPSESSID`如何获取呢？我们可以在登录之前先访问`segmentfault`主页，将返回的`cookie`拿到，再在登录的时候加上这个`cookie`即可。
获取cookie的代码如下：

```
(cb) => {
    superagent
      .get('https://segmentfault.com')
      .end((err, res) => {
        if (err) console.log(err)
        cookie = res.headers['set-cookie'].join(',').split(';')[0]   // 获取PHPSESSID
        cb(null)
      })
  }
```
### 获取Query String Paramsters里的 _ 参数 ###
还有一个参数需要注意：接口中的`Query String Paramsters`的`_`参数，那么这个参数是怎么来的呢？在返回的`response`的`header`里寻找并没有找到它的踪迹，所以猜想它应该是隐藏在源码里，我们直接在`chrome`控制台的`source`下全局搜索`_=`（`source`顶部右键选择`search in all files`即可出现全局搜索框）,逐一排查可能性：

![图片描述][2]

![图片描述][3]
排查过程中发现箭头所指的`ajaxSend`函数好像和我们需要的有关系：它紧邻的下面的`delegate`函数内容应该就是登陆有关的内容，通过`/api/user/?do=login`和`submit`等就可以清楚的看出，这个函数中的`url`是从`n.attr('action')`拿到的，猜测这个`n.__`肯定和请求中的`Query String`参数脱不了关系。正好这个`ajaxSend`函数中就有`n.__`,也正好验证了我们刚才的推测，分析这行代码：
```
n.url.indexOf("?") === -1 ? n.url = n.url + "?_=" + i._ : n.url = n.url + "&_=" + i._
```
`n.url`默认是`n.url + "?_=" + i._` ，那么这个`i.__`应该就是最终boss，就在这个文件中找到定义`i`的代码，如上图箭头所指，继续全局搜索`SF.token`,最终在`index.html`中找了生成它的代码，包含在一个`script`中，如图：

![图片描述][4]

找到来源就简单了，我们仍然是在登录直接先访问主页，将整个主页的html代码拿到，然后将这个script的内容取出来不就行了，哈哈，好开心~
获取script的代码如下：

```
var cheerio = require('cheerio')
function getRandom(s) {
  let $ = cheerio.load(s)
  let script = $('script').eq(8).html()
  let fn = new Function('window', script + ';return window.SF.token')
  let token = fn({})
  $ = null
  return token
}
exports.getRandom = getRandom
```
到这里，登录就算完成了一大半了，接下来就是简单的用`superagent`调用接口啦，这里的请求头出了`cookie`的其他部分也是必须要设置的，可以直接从浏览器上copy下来，代码如下：

```
(cb) => {
    const username = process.argv[2]
    const password = process.argv[3]
    console.log(cookie)
    console.log(random)
    let header = {
      'accept': '*/*',
      'accept-encoding': 'gzip, deflate, br',
      'accept-language': 'zh-CN,zh;q=0.9',
      'content-length': '47',
      'content-type': 'application/x-www-form-urlencoded; charset=UTF-8',
      'cookie': `PHPSESSID=${cookie};`,
      'origin': 'https://segmentfault.com',
      'referer': 'https://segmentfault.com/',
      'user-agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36',
      'x-requested-with': 'XMLHttpRequest'
    }
    superagent
      .post(`https://segmentfault.com/api/user/login`)
      .query({'_': random})
      .set(header)
      .type('form')
      .send({
        username: username,
        password: password,
        remember: 1
      })
      .end(function(err, res) {
        if (err) {
          console.log(err.status);
        } else {
          console.log('返回状态码： ' +  res.status)
          cb(null)
        }
      })
  },
```
终于返回200了，美滋滋，然后我们继续~比如我我想用代码修改个人主页的个人描述内容，首先我们先找到相关接口，如图：
![图片描述][5]

这个post请求的参数description就是个人描述的所填写的新的内容，我们直接用superagent调用这个接口
```
(cb) => {
    superagent
      // 编辑右上角个人说明
      .post('https://segmentfault.com/api/user/homepage/description/edit')
      .query({'_': random})
      .set(header)
      .type('form')
      .send({
        description: '努力coding的小喵~~~'
      })
      .end((err, res) => {
        if(err) throw err
        let result = JSON.parse(res.text)
        if (result.status === 0) {
          console.log('编辑成功')
        } else {
          console.log('编辑失败：' + result.data)
        }
      })
    cb(null, 1)
  }
```
返回状态码200，然后直接去浏览上的主页刷新下，可以看到个人描述的内容已经成功更新了！

![图片描述][6]
## 总结 ##

 1. 打开`segmentfault`主页并登陆，找到登录请求的接口并分析
 2. 用`node`登录之前，先请求主页接口，目的是拿到`PHPSESSID`和源代码中的生成`Query String`的函数
 3. 带着这两个参数去请求登录接口，记得设置请求头
 4. 登录成功之后就可以干任何你想干的事情啦

整个登录过程耗费了很久的时间，有空就研究这个`Query String`的来源，好不容易登录成功想干点事情，又没注意到设置请求头，以为是`sf_remember`参数的问题，又折腾了许久，还好，最终总算是成功了！感谢自己的不放弃~
## 源码 ##
github地址：[https://github.com/fighting123/node_login_segmentfault.git][7]


  [1]: /img/bVbdwop
  [2]: /img/bVbdwc1
  [3]: /img/bVbdwgX
  [4]: /img/bVbdwia
  [5]: /img/bVbdxXp
  [6]: /img/bVbdyF8
  [7]: https://github.com/fighting123/node_login_segmentfault.git
