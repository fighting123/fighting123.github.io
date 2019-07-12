---
title: var let const区别
categories: es6
tags:
  - es6
copyright: true
abbrlink: 3881397270
date: 2019-07-12 19:46:49
---

> 首先，分清楚js中存在两种作用域，即全局作用域和方法作用域
### var
var定义的变量是方法作用域，比如：
```
function() {
    var str = 'I am var';
    console.log(str); // I am var
}
console.log(str); // undefined
```
这是在函数中，但是需要注意的是在for循环中的问题：

```
function calcute({price: [2,4,3]}) {
    var totalPrice = [];
    for (var i = 0;i < price.length; i++) {
        var finallyPrice = price[i] * 2;
        totalPrice.push(finallyPrice);
    }
    console.log(i); // 3
    console.log(price); // [2,4,3]
    console.log(finallyPrice); // [4,8,6];
}
```
这个函数中都能访问到i、price、finallyPrice，这是我们不期望的，这就是var方法作用域的弊端
### let
let就正好解决了这个问题，它是块级作用域，即{}内可以访问
```
function calcute({price: [2,4,3]}) {
    let totalPrice = [];
    for (let i = 0;i < price.length; i++) {
        let finallyPrice = price[i] * 2;
        totalPrice.push(finallyPrice);
    }
    console.log(i); // i is not defined
    console.log(price); // [2,4,3]
    console.log(finallyPrice); // finallyPrice is not defined;
}
```
**还需要注意的是**：var定义的变量在定义之前访问是undefined，但是let定义变量在定义之前访问会报错Uncaught ReferenceError
#### const
const和let作用域一致，但是通过const赋值的变量不可再次复制（不是变量本身也不可变，只是不能再次赋值）

```
const params = {
    count: 3,
    price: 10
}
params.count = 4 // 正常
params = [] // 报错Uncaught TypeError
```
#### 总结
变量值会改变用let，不会改变用const（一般都是用来表述常量），尽量减少用var