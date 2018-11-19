---
title: Puppeteer(操纵Chrome的API)
tags: [node, 爬虫]
copyright: true
date: 2018-07-15 20:14:36
categories: Node.js
---
## 介绍：
[英文版官方文档](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions)：https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions

Puppeteer是一组用来操纵Chrome的API，默认headless也就是无UI的chrome，也可以配置为有UI，和PhantomJS类似，但是比他更有前景，可以实现爬虫，性能分析，自动化测试等功能。

## 适用情况
简单地通过get或post就能搞定的就不需要puppeteer了。

调接口获取不到的或者涉及请求加密的用puppeteer。

## 安装
最近蓝灯又被封了，上不了谷歌，puppeter又依赖Chromium/Chrome59+，只能折中啦--用puppeteer-cn代替puppeteer，这个包会先去检测本地Chrome版本是否大于59，再决定是否通过一个国内源下载Chromium，具体使用几乎和puppeteer一致。

puppeteer下载Chromium的默认路径是google服务器，GFW的原因可能会下载失败。puppeteer-cn还是依赖了puppeteer的，只是改写了其中的launch方法，并主动检测本地chrome是否符合headless条件，不符合会使用国内源安装Chromium。

```
npm install puppeteer-cn --save
```
怎么发现这样还是报错呢，还是cnpm靠谱啊,可以直接下载puppeteer-cn：
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm i puppeteer
```
或者更换国内Chromium源（暂时没有测试）：
```
PUPPETEER_DOWNLOAD_HOST=https://storage.googleapis.com.cnpmjs.org
npm i puppeteer
```
每次新建一个项目都要重新下载chromium，很不方便，其实可以把它下载到本地，每次都在puppeteer.launch({})中指定executablePath的位置，但是一定要注意chromium的版本是**r543305**，这有这个版本可以，否则会报错，具体在自己新建文件中的配置代码如下（不好找对应版本的话，可以第一次cnpm安装后在对应的路径下找到自带的chromium，这个版本肯定是正确的，将它提出来放到本地某个文件夹下即可）：
```
const browser = await (puppeteer.launch({
    // 若是手动下载的chromium需要指定chromium地址, 默认引用地址为 /项目目录/node_modules/puppeteer/.local-chromium/
    executablePath: 'D:/app/chromium/chrome-win32/chrome.exe',
    //设置超时时间
    timeout: 15000,
    //如果是访问https页面 此属性会忽略https错误
    ignoreHTTPSErrors: true,
    // 打开开发者工具, 当此值为true时, headless总为false
    devtools: false,
    // 关闭headless模式, 会打开浏览器
    headless: false
  }));
```