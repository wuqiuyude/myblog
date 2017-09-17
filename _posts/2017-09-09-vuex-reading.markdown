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
&emsp;&emsp;VuexInit就是在组件初始化的时候注入$store，我们在创建Vue实例的是时候,通过这样的方式new Vue({store}），将store作为参数传入，在vuexInit函数中，从 this.$options中取出store 赋值给组件的$store的。<br>
## Store对象
&emsp;&emsp;接下来将Store。感觉光这样看代码很费解，那么让我们来假定有这样一个简单的store实例：
```javascript
new Vuex.Store({
  actions,
  getters,
  state: {
      bags: []
  },
  modules: {
    book: {
      namespaced: true,
      state：{
        name: ''
      }
    },
    pen: {
      state: {
          color: ''
      }
    }
  },
  strict: debug,
  plugins: debug ? [createLogger()] : []
})
```
vuex2.0中的Store使用的es6的class类。前三行代码很简单就是看看window对象上是否有vue对象了，如果已经有了则自动安装vuex。然后就是用断言判断，在不是生产模式下，必须有先install vuex，必须支持promise,并且this指向的不是Store对象。
```javascript
if (!Vue && typeof window !== 'undefined' && window.Vue) {
    install(window.Vue)
}

if (process.env.NODE_ENV !== 'production') {
  assert(Vue, `must call Vue.use(Vuex) before creating a store instance.`)
  assert(typeof Promise !== 'undefined', `vuex requires a Promise polyfill in this browser.`)
  assert(this instanceof Store, `Store must be called with the new operator.`)
}
const {
    plugins = [],
    strict = false
} = options

let {
  state = {}
} = options
if (typeof state === 'function') {
  state = state()
}
// store internal state
  this._committing = false
  this._actions = Object.create(null)
  this._mutations = Object.create(null)
  this._wrappedGetters = Object.create(null)
  this._modules = new ModuleCollection(options)
  this._modulesNamespaceMap = Object.create(null)
  this._subscribers = []
  this._watcherVM = new Vue()
```
&emsp;&emsp;接下来的几行赋值不难理解，结合我们上面的store实例，这里的state={bags:[]},plugins=[createLogger()]。
主要是看看注释store internal state,看到这里大家应该很熟悉，这不是我们经常用的commit,actions, mutations吗。是的，这里就是创建vuex的内部熟悉。<br>
1、this._committing是vuex中用于标记提交状态的，作用是保证state只能在mutations中修改。初始值为false<br>
2、this._actions是用于存储所有定义的actions的。<br>
3、this._mutations是用于存储所有定义的mutations的。<br>
4、this._wrappedGetters是存储所有定义的getters。<br>
5、this._modules用于是一个ModuleCollection实例，用来存储modules<br>
6、this._modulesNamespaceMap用来存储modules的命名空间映射表。<br>
7、this._subscribers 用来存储所有对 mutation 变化的订阅者。<br>
8、this._watcherVM 是一个 Vue 对象的实例，主要是利用 Vue 实例方法 $watch 来观测变化的。
```javascript
export default class ModuleCollection {
  constructor (rawRootModule) {
    // register root module (Vuex.Store options)
    this.register([], rawRootModule, false)
  }

  get (path) {
    return path.reduce((module, key) => {
      return module.getChild(key)
    }, this.root)
  }

  getNamespace (path) {
    let module = this.root
    return path.reduce((namespace, key) => {
      module = module.getChild(key)
      return namespace + (module.namespaced ? key + '/' : '')
    }, '')
  }

  update (rawRootModule) {
    update([], this.root, rawRootModule)
  }

  register (path, rawModule, runtime = true) {
    if (process.env.NODE_ENV !== 'production') {
      assertRawModule(path, rawModule)
    }

    const newModule = new Module(rawModule, runtime)
    if (path.length === 0) {
      this.root = newModule
    } else {
      const parent = this.get(path.slice(0, -1))
      parent.addChild(path[path.length - 1], newModule)
    }

    // register nested modules
    if (rawModule.modules) {
      forEachValue(rawModule.modules, (rawChildModule, key) => {
        this.register(path.concat(key), rawChildModule, runtime)
      })
    }
  }

  unregister (path) {
    const parent = this.get(path.slice(0, -1))
    const key = path[path.length - 1]
    if (!parent.getChild(key).runtime) return

    parent.removeChild(key)
  }
}
```
先看看this._modules长什么样。this._modules是ModuleCollection对象的实例，我们在module文件夹下找到了这个module-collection.js。在ModuleCollection的构造函数中会递归调用register方法注册modules,第一次调用register方法时，传入的是一个空的path数组，则会在this.root绑定一个newModule，这个newModule是一个module实例。再看下面这段👇:
```javascript
// register nested modules
    if (rawModule.modules) {
      forEachValue(rawModule.modules, (rawChildModule, key) => {
        this.register(path.concat(key), rawChildModule, runtime)
      })
    }
```
在我的实例中，if判断为true,则执行forEachValue,forEachValue是作者写的一个util方法，用于遍历的处理对象中的每一个属性。
```javascript
export function forEachValue (obj, fn) {
  Object.keys(obj).forEach(key => fn(obj[key], key))
}
```
这里forEachValue用于遍历注册每一个module。当传入的path不为空数组时，会执行else里面的代码，先找到当前module的parent module，然后将当前module添加到他的parent module的子集中。可以看看vue-devtools里面的state是怎么样的：
![图片](/img/in-post/vuex-2.png)
```javascript
    // bind commit and dispatch to self
    const store = this
    const { dispatch, commit } = this
    this.dispatch = function boundDispatch (type, payload) {
      return dispatch.call(store, type, payload)
    }
    this.commit = function boundCommit (type, payload, options) {
      return commit.call(store, type, payload, options)
    }

    // strict mode
    this.strict = strict
```
&emsp;&emsp;接着看，将this对象绑定到了一个局部常量store上，<br>并且把this对象也就是当前的store实例的dispatch,和commit的指针指向一个局部的dispath和commit。
```javascript
 // init root module.
    // this also recursively registers all sub-modules
    // and collects all module getters inside this._wrappedGetters
    installModule(this, state, [], this._modules.root)

    // initialize the store vm, which is responsible for the reactivity
    // (also registers _wrappedGetters as computed properties)
    resetStoreVM(this, state)

    // apply plugins
    plugins.forEach(plugin => plugin(this))
```

&emsp;&emsp;接下来，先看看注释：<br>
&emsp;&emsp;初始化根模块，同时递归的注册所有的子模块，并且将所有的getters方法包装到this._wrappedGetters中。嗯，这句注释解释了下面这段代码干了什么，让我们具体看看installModule代码吧。
```javascript
function installModule (store, rootState, path, module, hot) {
  const isRoot = !path.length
  const namespace = store._modules.getNamespace(path)

  // register in namespace map
  if (module.namespaced) {
    store._modulesNamespaceMap[namespace] = module
  }

  // set state
  if (!isRoot && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      Vue.set(parentState, moduleName, module.state)
    })
  }

  const local = module.context = makeLocalContext(store, namespace, path)

  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })

  module.forEachAction((action, key) => {
    const namespacedType = namespace + key
    registerAction(store, namespacedType, action, local)
  })

  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })

  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
}
```

&emsp;&emsp;installModule一共有五个参数，store, rootState, path, module, hot，基本看名字都知道是什么了。结合 installModule(this, state, [], this._modules.root)这句方法的调用以及前面的实例。
```javascript
const isRoot = !path.length
const namespace = store._modules.getNamespace(path)
```

&emsp;&emsp;结合 installModule(this, state, [], this._modules.root)这句方法的调用以及前面的实例.第一句代码就是判断path的长度，确认是不是根目录。这里传入的空数组，所以在这里isRoot应该是true。然后看到调用了_modules实例的getNamespace方法，让我们看看这个方法是干啥。
```javascript
  getNamespace (path) {
    let module = this.root
    return path.reduce((namespace, key) => {
      module = module.getChild(key)
      return namespace + (module.namespaced ? key + '/' : '')
    }, '')
  }
```
嗯，就是根据path拼出module的namespace。&emsp;&emsp;在this._modules初始化的时候传入的参数是options，而this.root是一个Module实例。由于这里getNamespace传入的path是[],所以得到namespece是空字符串。默认情况下，模块内部的 action、mutation 和 getter 是注册在全局命名空间的——这样使得多个模块能够对同一 mutation 或 action 作出响应。如果希望你的模块更加自包含或提高可重用性，你可以通过添加 namespaced: true 的方式使其成为命名空间模块。当模块被注册后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名。
接下来看这段代码：
```javascript
  // set state
  if (!isRoot && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      Vue.set(parentState, moduleName, module.state)
    })
  }
```
&emsp;&emsp;在isRoot为false且非热跟新的状态下，递归初始化子状态。看一下geNestedState函数。
```javascript
function getNestedState (state, path) {
  return path.length
    ? path.reduce((state, key) => state[key], state)
    : state
}
```
这个方法很简单，就是根据 path 查找 state 上的嵌套 state。在这里就是传入rootState 和 path，计算出当前模块的父模块的state，由于模块的 path 是根据模块的名称 concat 连接的，所以 path的最后一个元素就是当前模块的模块名，最后调用:
```javascript
store._withCommit(() => {
    Vue.set(parentState, moduleName, module.state)
})
```
把当前模块的 state 添加到 parentState 中。这里用来store的_withCommit方法，让我们来看一下：
```javascript
  _withCommit (fn) {
    const committing = this._committing
    this._committing = true
    fn()
    this._committing = committing
  }
```
由于我们是在修改state，Vuex 中所有对 state 的修改都会用 _withCommit函数包装，保证在同步修改 state 的过程中 this._committing 的值始终为true。这样当我们观测 state 的变化时，如果 this._committing 的值不为 true，则能检查到这个状态修改是有问题的。<br>
接着看installModule方法：
```javascript
  const local = module.context = makeLocalContext(store, namespace, path)

  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })

  module.forEachAction((action, key) => {
    const namespacedType = namespace + key
    registerAction(store, namespacedType, action, local)
  })

  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })

  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
```
先看一下makeLocalContext函数：
```javascript
/**
 * make localized dispatch, commit, getters and state
 * if there is no namespace, just use root ones
 */
function makeLocalContext (store, namespace, path) {
  const noNamespace = namespace === ''

  const local = {
    dispatch: noNamespace ? store.dispatch : (_type, _payload, _options) => {
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args

      if (!options || !options.root) {
        type = namespace + type
        if (process.env.NODE_ENV !== 'production' && !store._actions[type]) {
          console.error(`[vuex] unknown local action type: ${args.type}, global type: ${type}`)
          return
        }
      }

      return store.dispatch(type, payload)
    },

    commit: noNamespace ? store.commit : (_type, _payload, _options) => {
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args

      if (!options || !options.root) {
        type = namespace + type
        if (process.env.NODE_ENV !== 'production' && !store._mutations[type]) {
          console.error(`[vuex] unknown local mutation type: ${args.type}, global type: ${type}`)
          return
        }
      }

      store.commit(type, payload, options)
    }
  }

  // getters and state object must be gotten lazily
  // because they will be changed by vm update
  Object.defineProperties(local, {
    getters: {
      get: noNamespace
        ? () => store.getters
        : () => makeLocalGetters(store, namespace)
    },
    state: {
      get: () => getNestedState(store.state, path)
    }
  })

  return local
}
```
这段代码的意思就是为module设置局部的 dispatch、commit方法以及getters和state，如果命名空间存在则做特殊的兼容处理。现在每一个module都有自己的dispatch、commit方法以及getters和state了。接下来的几个循环处理分别是为module注册mutations、actions以及getter。分别看下几个注册函数：
```javascript
function registerMutation (store, type, handler, local) {
  const entry = store._mutations[type] || (store._mutations[type] = [])
  entry.push(function wrappedMutationHandler (payload) {
    handler.call(store, local.state, payload)
  })
}
```
registerMutation函数是注册当前命名空间（type）下的mutations使用wrappedMutationHandler包装成功，给原函数传入了state。在执行 commit('xxx', payload) 的时候，type为 xxx 的mutation的所有handler都会接收到state以及payload，这就是在handler里面拿到state的原因。
```javascript
function registerAction (store, type, handler, local) {
  // 取出对应type的actions-handler集合
  const entry = store._actions[type] || (store._actions[type] = [])
  // 存储新的封装过的action-handler
  entry.push(function wrappedActionHandler (payload, cb) {
    // 传入 state 等对象供我们原action-handler使用
    let res = handler({
      dispatch: local.dispatch,
      commit: local.commit,
      getters: local.getters,
      state: local.state,
      rootGetters: store.getters,
      rootState: store.state
    }, payload, cb)
    // action需要支持promise进行链式调用，这里进行兼容处理
    if (!isPromise(res)) {
      res = Promise.resolve(res)
    }
    if (store._devtoolHook) {
      return res.catch(err => {
        store._devtoolHook.emit('vuex:error', err)
        throw err
      })
    } else {
      return res
    }
  })
}
```
action handler比mutation handler以及getter wrapper多拿到dispatch和commit操作方法，因此action可以进行dispatch action和commit mutation操作。
```javascript
function registerGetter (store, type, rawGetter, local) {
  // getters只允许存在一个处理函数，若重复需要报错
  if (store._wrappedGetters[type]) {
    console.error(`[vuex] duplicate getter key: ${type}`)
    return
  }

  // 存储封装过的getters处理函数
  store._wrappedGetters[type] = function wrappedGetter (store) {
    // 为原getters传入对应状态
    return rawGetter(
      local.state, // local state
      local.getters, // local getters
      store.state, // root state
      store.getters // root getters
    )
  }
}
```
注册完了根组件的actions、mutations以及getters后，递归调用自身，为子组件注册其state，actions、mutations以及getters等。
```javascript
module.forEachChild((child, key) => {
  installModule(store, rootState, path.concat(key), child, hot)
})
```
未完待续。。。<br>
[1]参考[Vuex框架原理与源码分析](http://geek.csdn.net/news/detail/196495?locationNum=12&fps=1)<br>
[2]参考[Vuex文档](https://vuex.vuejs.org/zh-cn/api.html1)<br>
[3]参考[Vuex 源码解析（如何阅读源代码实践篇）](http://www.jianshu.com/p/8273b19c9d74)<br>
[4]参考[ Vuex 2.0 源码分析](http://blog.csdn.net/sinat_17775997/article/details/62231288)