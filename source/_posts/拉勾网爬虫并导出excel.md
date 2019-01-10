---
title: 拉勾网爬虫并导出excel
date:
  '[object Object]': null
categories: Node.js
tags:
  - node
  - 爬虫
copyright: true
abbrlink: 870243141
---
## 前言 ##

> 之前断断续续学习了node.js，今天就拿拉勾网练练手，顺便通过数据了解了解最近的招聘行情哈！node方面算是萌新一个吧，希望可以和大家共同学习和进步。
## 一、概要 ##
我们首先需要明确具体的需求：

 1. 可以通过`node index 城市 职位`来爬取相关信息
 2. 也可以输入node index start直接爬取我们预定义好的城市和职位数组，循环爬取不同城市的不同职位信息
 3. 将最终爬取的结果存储在本地的`./data`目录下
 4. 生成对应的excel文件，并存储到本地

## 二、爬虫用到的相关模块 ##

 - fs: 用于对系统文件及目录进行读写操作
 - async：流程控制
 - superagent：客户端请求代理模块
 - node-xlsx：将一定格式的文件导出为excel

## 三、爬虫主要步骤： ##
### 初始化项目 ###
新建项目目录
> 在合适的磁盘目录下创建项目目录 node-crwl-lagou

初始化项目
> 1. 进入node-crwl-lagou文件夹下
> 2. 执行npm init，初始化package.json文件

安装依赖包

> 1. npm install async
> 2. npm install superagent
> 3. npm install node-xlsx

### 命令行输入的处理 ###
对于在命令行输入的内容，可以用`process.argv`来获取，他会返回个数组，数组的每一项就是用户输入的内容。
区分`node index 地域 职位`和`node index start`两种输入，最简单的就是判断process.argv的长度，长度为四的话，就直接调用爬虫主程序爬取数据，长度为三的话，我们就需要通过预定义的城市和职位数组来拼凑url了，然后利用async.mapSeries循环调用主程序。关于命令分析的主页代码如下：

```
if (process.argv.length === 4) {
  let args = process.argv
  console.log('准备开始请求' + args[2] + '的' + args[3] + '职位数据');
  requsetCrwl.controlRequest(args[2], args[3])
} else if (process.argv.length === 3 && process.argv[2] === 'start') {
  let arr = []
  for (let i = 0; i < defaultArgv.city.length; i++) {
    for (let j = 0; j < defaultArgv.position.length; j++) {
      let obj = {}
      obj.city = defaultArgv.city[i]
      obj.position = defaultArgv.position[j]
      arr.push(obj)
    }
  }
  async.mapSeries(arr, function (item, callback) {
    console.log('准备开始请求' + item.city + '的' + item.position + '职位数据');
    requsetCrwl.controlRequest(item.city, item.position, callback)
  }, function (err) {
    if (err) throw err
  })
} else {
  console.log('请正确输入要爬取的城市和职位，正确格式为："node index 城市 关键词" 或 "node index start" 例如："node index 北京 php" 或"node index start"')
}
```
预定义好的城市和职位数组如下：

```
{
    "city": ["北京","上海","广州","深圳","杭州","南京","成都","西安","武汉","重庆"],
    "position": ["前端","java","php","ios","android","c++","python",".NET"]
}
```

接下来就是爬虫主程序部分的分析了。

### 分析页面，找到请求地址
首先我们打开拉勾网首页，输入查询信息（比如node），然后查看控制台，找到相关的请求，如图：

![图片描述](https://raw.githubusercontent.com/fighting123/hexo_images/master/oldArticleImages/%E6%8B%89%E5%8B%BE%E7%BD%91%E7%88%AC%E8%99%AB%E5%B9%B6%E5%AF%BC%E5%87%BAexcel1.png)

这个post请求`https://www.lagou.com/jobs/positionAjax.json?needAddtionalResult=false`就是我们所需要的，通过三个请求参数来获取不同的数据，简单的分析就可得知：参数`first`是标注当前是否是第一页，true为是，false为否；参数`pn`是当前的页码；参数`kd`是查询输入的内容。

### 通过superagent请求数据 ###
首先需要明确得是，整个程序是异步的，我们需要用async.series来依次调用。
查看分析返回的response：

![图片描述](https://raw.githubusercontent.com/fighting123/hexo_images/master/oldArticleImages/%E6%8B%89%E5%8B%BE%E7%BD%91%E7%88%AC%E8%99%AB%E5%B9%B6%E5%AF%BC%E5%87%BAexcel2.png)

可以看到content.positionResult.totalCount就是我们所需要的总页数
我们用superagent直接调用post请求，控制台会提示如下信息：

    {'success': False, 'msg': '您操作太频繁,请稍后再访问', 'clientIp': '122.xxx.xxx.xxx'}
这其实是反爬虫策略之一，我们只需要给其添加一个请求头即可，请求头的获取方式很简单，如下：

![图片描述](https://raw.githubusercontent.com/fighting123/hexo_images/master/oldArticleImages/%E6%8B%89%E5%8B%BE%E7%BD%91%E7%88%AC%E8%99%AB%E5%B9%B6%E5%AF%BC%E5%87%BAexcel3.png)]

然后在用superagent调用post请求，主要代码如下：

```
// 先获取总页数
    (cb) => {
      superagent
        .post(`https://www.lagou.com/jobs/positionAjax.json?needAddtionalResult=false&city=${city}&kd=${position}&pn=1`)
        .send({
          'pn': 1,
          'kd': position,
          'first': true
        })
        .set(options.options)
        .end((err, res) => {
          if (err) throw err
          // console.log(res.text)
          let resObj = JSON.parse(res.text)
          if (resObj.success === true) {
            totalPage = resObj.content.positionResult.totalCount;
            cb(null, totalPage);
          } else {
            console.log(`获取数据失败:${res.text}}`)
          }
        })
    },
```
拿到总页数后，我们就可以通过`总页数/15`获取到pn参数，循环生成所有url并存入urls中：

```
(cb) => {
      for (let i=0;Math.ceil(i<totalPage/15);i++) {
        urls.push(`https://www.lagou.com/jobs/positionAjax.json?needAddtionalResult=false&city=${city}&kd=${position}&pn=${i}`)
      }
      console.log(`${city}的${position}职位共${totalPage}条数据，${urls.length}页`);
      cb(null, urls);
    },
```
有了所有的url，在想爬到所有的数据就不是难事了，继续用superagent的post方法循环请求所有的url，每一次获取到数据后，在data目录下创建json文件，将返回的数据写入。这里看似简单，但是有两点需要注意：
 1. 为了防止并发请求太多而导致被封IP：循环url时候需要使用async.mapLimit方法控制并发为3， 每次请求完都要过两秒在发送下一次的请求
 2. 在async.mapLimit的第四个参数中，需要通过判断调用主函数的第三个参数是否存在来区分一下是那种命令输入，因为对于`node index start`这个命令，我们使用得是async.mapSeries，每次调用主函数都传递了`(city, position, callback)`，所以如果是`node index start`的话，需要在每次获取数据完后将null传递回去，否则无法进行下一次循环

主要代码如下：

```
// 控制并发为3
    (cb) => {
      async.mapLimit(urls, 3, (url, callback) => {
        num++;
        let page = url.split('&')[3].split('=')[1];
        superagent
          .post(url)
          .send({
            'pn': totalPage,
            'kd': position,
            'first': false
          })
          .set(options.options)
          .end((err, res) => {
            if (err) throw err
            let resObj = JSON.parse(res.text)
            if (resObj.success === true) {
              console.log(`正在抓取第${page}页，当前并发数量：${num}`);
              if (!fs.existsSync('./data')) {
                fs.mkdirSync('./data');
              }
              // 将数据以.json格式储存在data文件夹下
              fs.writeFile(`./data/${city}_${position}_${page}.json`, res.text, (err) => {
                if (err) throw err;
                // 写入数据完成后，两秒后再发送下一次请求
                setTimeout(() => {
                  num--;
                  console.log(`第${page}页写入成功`);
                  callback(null, 'success');
                }, 2000);
              });
            }
          })
      }, (err, result) => {
        if (err) throw err;
        // 这个arguments是调用controlRequest函数的参数，可以区分是那种爬取（循环还是单个）
        if (arguments[2]) {
          ok = 1;
        }
        cb(null, ok)
      })
    },
    () => {
      if (ok) {
        setTimeout(function () {
          console.log(`${city}的${position}数据请求完成`);
          indexCallback(null);
        }, 5000);
      } else {
        console.log(`${city}的${position}数据请求完成`);
      }
      // exportExcel.exportExcel() // 导出为excel
    }
```
导出的json文件如下：
![图片描述](https://raw.githubusercontent.com/fighting123/hexo_images/master/oldArticleImages/%E6%8B%89%E5%8B%BE%E7%BD%91%E7%88%AC%E8%99%AB%E5%B9%B6%E5%AF%BC%E5%87%BAexcel4.png)
### json文件导出为excel ###
将json文件导出为excel有多种方式，我使用的是`node-xlsx`这个node包，这个包需要将数据按照固定的格式传入，然后导出即可，所以我们首先做的就是先拼出其所需的数据格式：

```
function exportExcel() {
  let list = fs.readdirSync('./data')
  let dataArr = []
  list.forEach((item, index) => {
    let path = `./data/${item}`
    let obj = fs.readFileSync(path, 'utf-8')
    let content = JSON.parse(obj).content.positionResult.result
    let arr = [['companyFullName', 'createTime', 'workYear', 'education', 'city', 'positionName', 'positionAdvantage', 'companyLabelList', 'salary']]
    content.forEach((contentItem) => {
      arr.push([contentItem.companyFullName, contentItem.phone, contentItem.workYear, contentItem.education, contentItem.city, contentItem.positionName, contentItem.positionAdvantage, contentItem.companyLabelList.join(','), contentItem.salary])
    })
    dataArr[index] = {
      data: arr,
      name: path.split('./data/')[1] // 名字不能包含 \ / ? * [ ]
    }
  })

// 数据格式
// var data = [
//   {
//     name : 'sheet1',
//     data : [
//       [
//         'ID',
//         'Name',
//         'Score'
//       ],
//       [
//         '1',
//         'Michael',
//         '99'
//
//       ],
//       [
//         '2',
//         'Jordan',
//         '98'
//       ]
//     ]
//   },
//   {
//     name : 'sheet2',
//     data : [
//       [
//         'AA',
//         'BB'
//       ],
//       [
//         '23',
//         '24'
//       ]
//     ]
//   }
// ]

// 写xlsx
  var buffer = xlsx.build(dataArr)
  fs.writeFile('./result.xlsx', buffer, function (err)
    {
      if (err)
        throw err;
      console.log('Write to xls has finished');

// 读xlsx
//     var obj = xlsx.parse("./" + "resut.xls");
//     console.log(JSON.stringify(obj));
    }
  );
}
```
导出的excel文件如下，每一页的数据都是一个sheet，比较清晰明了：
![图片描述](https://raw.githubusercontent.com/fighting123/hexo_images/master/oldArticleImages/%E6%8B%89%E5%8B%BE%E7%BD%91%E7%88%AC%E8%99%AB%E5%B9%B6%E5%AF%BC%E5%87%BAexcel5.png)

我们可以很清楚的从中看出目前西安.net的招聘情况，之后也可以考虑用更形象的图表方式展示爬到的数据，应该会更加直观！
## 总结 ##
其实整个爬虫过程并不复杂，注意就是注意的小点很多，比如async的各个方法的使用以及导出设置header等，总之，也是收获满满哒！ 
## 源码 ##
gitbug地址： [https://github.com/fighting123/node_crwl_lagou][6]

  [6]: https://github.com/fighting123/node_crwl_lagou