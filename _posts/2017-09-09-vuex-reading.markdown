---
layout:     post
title:      "vuex源码阅读"
subtitle:   "vuex源码阅读，向大神学习"
date:       2017-09-09 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/vuex-reading.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - vuex
    - 源码阅读
    - 原创
---
> 之前在使用react搭建第一个H5项目的时候, 遇到了一个问题，当页面跳转回之前的feed流页面的时候，要求保持滚动条的数据和feeds里的数据和状态，这个时候就涉及到单页应用状态保持的问题，React可以使用Redux,在vue中也有Vuex。Vuex在中大型单页面项目中可以很好的实现页面状态的管理。<br>

## 什么是Vuex
&emsp;&emsp;Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。<br>
&emsp;&emsp;在一个简单的vue项目中，如果我们要在多个组件中共享一个状态，我们会使用[global event bus](http://nvie.com/posts/a-successful-git-branching-model/)解决（解释一下event bus是指两个非父子组建在通信时，可以使用一个空的vue实例作为中央事件总线）。但是复杂的系统中如果多个组件依赖于一个状态，组件之间的关系会越来越复杂，状态的操作会越来越烦琐，并且也是不安全的，对代码的维护也会越来越困难，这个时候就需要用专门的状态管理模式。Vuex就是专门为vue服务的状态管理插件。<br>
&emsp;&emsp;Vuex将所有的公共状态抽离出来，形成一个独立的状态树，任何组件可以访问和修改状态。这使得项目的状态更容易管理，也更清晰。Vuex借鉴了 Flux、Redux、和 The Elm Architecture。
## Vuex的结构
&emsp;&emsp; Vuex主要State,Getters,Mutations,Actions,Modules五个部分组成。<br>
1、State:<br>
&emsp;&emsp;Vuex的核心部分就State对象，用于存储应用中的公共状态。 Vuex的State对象改变会相应地触发相关组件的变化，但是对于State中存储的状态不能之间通过组件去修改，只能通过vuex提供的方法改变。这种方式可以更好的追踪状态的修改。<br>
2、Getters:<br>
&emsp;&emsp;Getters方法可以对State的状态进行一些操作<br>
3、Mutations<br>
&emsp;&emsp;Mutations是改变State的状态唯一方法，Mutations中只能进行同步操作。<br>
4、Actions:<br>
&emsp;&emsp;Actions用于提交Mutations，但是不能直接改变state,Actions可以进行同步操作<br>
5、Modules:<br>Modules用于将一个vuex拆分成多个模块。<br>
&emsp;&emsp;
![图片](/img/in-post/vuex.png)
## Vuex源码阅读
&emsp;&emsp;接下来开始正式阅读源码，首先看一看Vuex2.0的目录结构：
![图片](/img/in-post/vuex-modal.png)
&emsp;&emsp;看到index.js就知道这个应该是入口文件了，让我们先来看看index.js。
```javascript
import { Store, install } from './store'
import { mapState, mapMutations, mapGetters, mapActions, createNamespacedHelpers } from './helpers'

export default {
  Store,
  install,
  version: '__VERSION__',
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}
```
&emsp;&emsp;只有几行代码，但是目的已经很清晰了，就是将vuex的几个对象暴露出来。我们看到有一个install方法，这是vue的插件必须提供的一个公开方法，该方法会在你使用该插件，也就是Vue.use(Vuex) 时被调用，相当于是一个插件的注册或者声明。install 接受 Vue 构造器作为第一个参数，并且有一个可选的选项对象作为第二个参数，让我们看一看vuex的install方法。
```javascript
export function install (_Vue) {
  if (Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```
&emsp;&emsp;if语句用语判断vue是否依旧注册了vuex。如果没有则调用applyMixin，传入Vue对象。是的，我们看到这个方法叫做applyMixin，我们在官方文档中也可以看到，其实vuex是使用mixin的方法实现的。除了mixin，插件的开发方式有很多中，有兴趣的可以去官方[文档](https://cn.vuejs.org/v2/guide/plugins.html)看看。接下来看一看applyMixin方法，在mixin.js中。
```javascript
export default function (Vue) {
  const version = Number(Vue.version.split('.')[0])

  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    const _init = Vue.prototype._init
    Vue.prototype._init = function (options = {}) {
      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit
      _init.call(this, options)
    }
  }

  /**
   * Vuex init hook, injected into each instances init hooks list.
   */

  function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}

```

&emsp;&emsp;对1.x版本的做了一个backwards，1.x版本中是在init方法中初始化Vuex,2.0则是采用mixin的方法在beforeCreate，这段代码不难理解。让我们看看vuexInit方法干了什么吧，首先我们看到将this.$options实例，拷贝了一份到options,$options是Vue实例的一个只读实例，用于当前 Vue 实例的初始化选项，需要在选项中包含自定义属性时会有用处(来自官方文档)：
```javascript
new Vue({
  store: 'store',
  created: function () {
    console.log(this.$options.store) // => 'store'
  }
})

```
VuexInit就是在组件初始化的时候注入$store，我们在创建Vue实例的是时候,通过这样的方式new Vue({store}），将store作为参数传入，在vuexInit函数中，从 this.$options中取出store 赋值给组件的$store的。<br>
结下来将Store对象。未完待续。。。。






















