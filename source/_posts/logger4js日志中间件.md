---
title: logger4js日志中间件
tags: [koa2, 后台日志]
copyright: true
date: 2018-04-18 19:20:19
categories: koa2
---
> 周报管理系统的日志模块本来是简单地console.log()出一些请求信息，但是并不能长时间保存，也不方便查看问题，但是logger4js这个中间件完美的实现了我要的功能。

先贴出他的以及两篇有关配置的参考文章：

- github地址：https://github.com/log4js-node/log4js-node/
- nodejs Log4js v1.x配置使用(主要是参考目录结构)：https://www.jianshu.com/p/6b816c609669
- nodejs Log4js v2.x配置使用：https://blog.csdn.net/llzkkk12/article/details/78165779

使用步骤：
### 1.安装

```
npm install log4js –save
```
### 2.配置
新建log_config.js，这里是他的相关配置，注意categories的level，配置日志的输出级别,共ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < MARK < OFF八个级别,default level is OFF只有大于等于日志配置级别的信息才能输出出来，可以通过category来有效的控制日志输出级别


```
var path = require('path');
var fs = require('fs');

//日志根目录
var baseLogPath = path.resolve(__dirname, '../logs')

//错误日志目录
var errorPath = "/error";
//错误日志文件名
var errorFileName = "error";
//错误日志输出完整路径
var errorLogPath = baseLogPath + errorPath + "/" + errorFileName;

//响应日志目录
var responsePath = "/response";
//响应日志文件名
var responseFileName = "response";
//响应日志输出完整路径
var responseLogPath = baseLogPath + responsePath + "/" + responseFileName;

let log_config_obj = {
  appenders: {
    out: {
      type: 'console'
    },
    errorLogger: {
      "type": "dateFile",                   //日志类型
      "filename": errorLogPath,             //日志输出位置
      "alwaysIncludePattern": true,          //是否总是有后缀名
      "pattern": "-yyyy-MM-dd-hh.log",      //后缀，每小时创建一个新的日志文件
      "path": errorPath                     //自定义属性，错误日志的根目录
    },
    resLogger: {
      "type": "dateFile",
      "filename": responseLogPath,
      "alwaysIncludePattern": true,
      "pattern": "-yyyy-MM-dd-hh.log",
      "path": responsePath
    }
  },
  categories: {
    default: {appenders: ['out'], level: 'info'},  // 必须添加default，并且配置项都要写。所以添加了out
    errorLog: { appenders: ['errorLogger'], level: 'error' },
    resLog: { appenders: ['resLogger'], level: 'info' }
  },
  baseLogPath: baseLogPath                  //logs根目录
}
module.exports = log_config_obj
```
新建log_util.js文件，这里是日志保存的内容处理，代码太长，只有错误处理部分，请求处理部分同理：

```
ar log4js = require('log4js');
var log_config = require('../config/log_config');
//加载配置文件
log4js.configure(log_config);
//日志报错的内容
var logUtil = {};
var errorLogger = log4js.getLogger('errorLog');
//封装错误日志
logUtil.logError = function (ctx, error, resTime) {
  if (ctx && error) {
    errorLogger.error(formatError(ctx, error, resTime));
  }
};
//格式化错误日志
var formatError = function (ctx, err, resTime) {
  var logText = new String();
  //错误信息开始
  logText += "\n" + "*************** error log start ***************" + "\n";
  //添加请求日志
  logText += formatReqLog(ctx.request, resTime);
  //错误名称
  logText += "err name: " + err.name + "\n";
  //错误信息
  logText += "err message: " + err.message + "\n";
  //错误详情
  logText += "err stack: " + err.stack + "\n";
  //错误信息结束
  logText += "*************** error log end ***************" + "\n";
  return logText;
};

module.exports = logUtil;

```
在index文件中添加引用：

```
const logUtil = require('./middlewares/log_util');      // 记录日志
......

// 日志接口调用信息日志输出
app.use(async (ctx, next) => {
  //响应开始时间
  const start = new Date();
  //响应间隔时间
  var ms;
  try {
    //开始进入到下一个中间件
    await next();
    ms = new Date() - start;
    //记录响应日志
    logUtil.logResponse(ctx, ms);
  } catch (error) {
    ms = new Date() - start;
    //记录异常日志
    logUtil.logError(ctx, error, ms);
  }
});
```
### 3. 保存日志的文件
这个文件手动添加太麻烦，我们可以直接用代码判断他存不存在，不存在直接新建（可以写在 log_config.js中）

```
var fs = require('fs');
/**
 * 确定目录是否存在，如果不存在则创建目录
 */
var confirmPath = function(pathStr) {
  if(!fs.existsSync(pathStr)){
    fs.mkdirSync(pathStr);
    console.log('createPath: ' + pathStr);
  }
}
/**
 * 初始化log相关目录
 */
var initLogPath = function(){
  //创建log的根目录'logs'
  if(baseLogPath){
    confirmPath(baseLogPath)
    //根据不同的logType创建不同的文件目录
    //log4js2.0的appenders是个对象，length无法判断直接长度
    let appendersArr = Object.keys(log_config_obj.appenders);
    for(var i = 0, len = appendersArr.length; i < len; i++){
      if(log_config_obj.appenders[appendersArr[i]].path){
        confirmPath(baseLogPath + log_config_obj.appenders[appendersArr[i]].path);
      }
    }
  }
}

initLogPath();
```

现在运行就可以看到生成的logs目录以及日志记录文件啦！