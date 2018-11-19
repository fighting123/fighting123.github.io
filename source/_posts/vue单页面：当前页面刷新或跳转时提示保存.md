---
title: vue单页面：当前页面刷新或跳转时提示保存
date: 2018-11-01 19:49:36
categories: Vue
tags: [Vue, Vue-Router]
copyright : ture
---

### 前言

> 最近公司vue项目中有一个需求，需要在当前页面刷新或跳转时提示保存并可取消刷新，以防止填写的表单内容丢失。刚开始思考觉得很简单，直接在Router的钩子中判断就好了，但是会发现还有新的问题存在，浏览器刷新和当前页面关闭的时候无法监听，最终用window.onbeforeunload成功解决，所以用这篇文章简单记录下整个解决过程。


### vue-Router的钩子：

路由钩子可以分为**全局的，单个路由独享的以及组件级别的**，解决上述需求只用到了组件级别的路由钩子，所以本文只介绍组件级别的路由钩子，全局的和单个路由独享的路由钩子有需要的同学可以去[vue-router官网](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)查看介绍：

组件级别路由钩子分为三种：

- beforeRouteEnter：当成功获取并能进入路由(在渲染该组件的对应路由被 confirm 前)
- beforeRouteUpdate：在当前路由改变，但是该组件被复用时调用
- beforeRouteLeave：导航离开该组件的对应路由时调用

具体的介绍和写法如下：
```
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
    // 可以通过传一个回调给 next来访问组件实例
    next(vm => { 
            // 通过 `vm` 访问组件实例
        })
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
    // 不支持传递回调(因为this实例已经创建可以获取到，所以没必要)
    next()
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
    // 该导航可以通过 next(false) 来取消。
    const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
    if (answer) {
        next()
    } else {
    // 不支持传递回调(因为this实例已经创建可以获取到，所以没必要)
        next(false)
    }
  }
}
```
**注意**：在刷新当前页面时候，beforeRouteLeave不会触发，它只在进入到其他页面时候才会触发，但是beforeRouteEnter会在刷新的时候触发。

通过beforeRouteLeave这个路由钩子，我们就可以在用户要离开此页面时候进行提示了！

```
beforeRouteLeave (to, from, next) {
    const answer = window.confirm('当前页面数据未保存，确定要离开![image](http://note.youdao.com/favicon.ico)？')
    if (answer) {
        next()
    } else {
        next(false)
    }
  }
```
显示的提示框如下：

![image](http://p5tstjsfi.bkt.clouddn.com/vue-router1luyou.png)

### 监听浏览器的刷新、页面关闭事件

但是，这个时候就实现了我们最终的需求了么？并没有，还有一步：用**window.onbeforeunload**监听浏览器的刷新事件，当然为了防止从当前单页面跳到其他页面之后，在刷新所在新的页面还会触发window上的onbeforeunload的问题，我们不仅要及时的添加onbeforeunload事件，更要及时删除此事件，下面有两种解决方法可供选择：

1. 通过判断它的路由是否是当前需要添加禁止刷新的页面

```
mounted() {
    window.onbeforeunload = function (e) {
      if(_this.$route.fullPath =="/layout/add"){
          e = e || window.event;
          // 兼容IE8和Firefox 4之前的版本
          if (e) {
            e.returnValue = '关闭提示';
          }
          // Chrome, Safari, Firefox 4+, Opera 12+ , IE 9+
          return '关闭提示';
      }else{
        window.onbeforeunload =null
      }
}
};
```

2. 在destory或者beforeDestory的生命周期中直接将onbeforeunload事件置空
```
mounted() {
    window.onbeforeunload = function (e) {
        e = e || window.event;
        // 兼容IE8和Firefox 4之前的版本
        if (e) {
            e.returnValue = '关闭提示';
        }
        // Chrome, Safari, Firefox 4+, Opera 12+ , IE 9+
        return '关闭提示';
    }
};
destroyed() {
      window.onbeforeunload = null
    }
```

显示的提示框如下：

![image](http://p5tstjsfi.bkt.clouddn.com/vue-router1%E8%B7%AF%E7%94%B1%E9%92%A9%E5%AD%90.png)
### 总结
最终，在beforeRouteLeave和onbeforeunload的共同作用下，这个刷新、跳转或者关闭等情况下需要提示保存的需求完美解决！但是，还有一点点小遗憾，就是onbeforeunload中弹框的自定义提示语设置始终无法生效，大家要是有更加合适的处理办法，欢迎多多交流指正！