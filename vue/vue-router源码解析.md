# 参考文献

https://ustbhuangyi.github.io/vue-analysis/v2/vue-router/install.html

https://www.cnblogs.com/zhangycun/p/9403339.html

https://segmentfault.com/a/1190000018902037

# 前言

`Vue Router`是vue官方的路由管理器。包含的功能有：

- 嵌套的路由/视图表
- 模块化的、基于组件的路由配置
- 路由参数、查询、通配符
- 基于 Vue.js 过渡系统的视图过渡效果
- 细粒度的导航控制
- 带有自动激活的 CSS class 的链接
- HTML5 历史模式或 hash 模式，在 IE9 中自动降级
- 自定义的滚动条行为

# 路由模式

`Vue Router`有两种路由方式：默认为`hash`模式，以及`history`模式。

## hash模式

`hash`模式就是使用`URL`的`hash`来模拟一个完整的`URL`，通过`hashChange`事件监听`URL`的变化，匹配对应的路由规则，并且页面不会重新加载。

```
// 如下所示，就是一个hash模式的URL示例，#之后的字符串也就是URL的hash
http://localhost:8080/#/index
```

## history模式

`history`模式是`HTML5`新推出的功能，通过`history.pushState`API来完成URL的跳转而无须重新加载页面。而且`history`模式相比`hash`模式的`URL`看起来会更美观。

```
// 如下所示，就是一个history模式的URL示例
http://localhost:8080/index
```

# Vue Router源码解析

## 应用初始化

在vue应用构建的时候，我们会使用`Vue.use(VueRouter)`以插件的方式安装`VueRouter`，同时在Vue的实例上挂载路由的实例，使用`Vue.use`是为了让路由插件可以使用vue。

```
import Vue from 'vue'
import App from './App.vue'
import router from './router'

Vue.config.productionTip = false

let a = new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

## 路由注册

`Vue.use`的源码如下所示，定义在`vue/src/core/global-api/use.js`中。

```
export function initUse (Vue: GlobalAPI) {
    Vue.use = function (plugin: Function | Object) {
        // 判断重复安装插件
        const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
        if (installedPlugins.indexOf(plugin) > -1) {
            return this
        }
        const args = toArray(arguments, 1)
        // 插入 Vue
        args.unshift(this)
        // 一般插件都会有一个 install 函数
        // 通过该函数让插件可以使用 Vue
        if (typeof plugin.install === 'function') {
            plugin.install.apply(plugin, args)
        } else if (typeof plugin === 'function') {
            plugin.apply(null, args)
        }
        installedPlugins.push(plugin)
        return this
    }
}
```

`Vue.use` 接受一个 `plugin` 参数，并且维护了一个 `_installedPlugins` 数组，它存储所有注册过的 `plugin`；接着又会判断 `plugin` 有没有定义 `install` 方法，如果有的话则调用该方法，并且该方法执行的第一个参数是 `Vue`；最后把 `plugin` 存储到 `installedPlugins` 中。

可以看到 Vue 提供的插件注册机制很简单，每个插件都需要实现一个静态的 `install` 方法，当我们执行 `Vue.use` 注册插件的时候，就会执行这个 `install` 方法，并且在这个 `install` 方法的第一个参数我们可以拿到 `Vue` 对象，这样的好处就是作为插件的编写方不需要再额外去`import Vue` 了。

## 路由安装

Vue-Router 的入口文件是 `src/index.js`，其中定义了 `VueRouter` 类，也实现了 `install` 的静态方法：`VueRouter.install = install`，它的定义在 `src/install.js` 中。

```
import View from './components/view'
import Link from './components/link'

export let _Vue
export function install (Vue) {
  // 确保 install 调用一次
  if (install.installed && _Vue === Vue) return
  install.installed = true

  // 把Vue参数保存给_Vue，这样就不需要import Vue了，以免增加打包的体积
  _Vue = Vue
  
  // 判断变量是否定义
  const isDef = v => v !== undefined
  
  // 
  const registerInstance = (vm, callVal) => {
    let i = vm.$options._parentVnode
    if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
      i(vm, callVal)
    }
  }
  // 给每个组件的钩子函数混入实现
  // 可以发现在beforeCreate钩子执行时
  // 会初始化路由
  Vue.mixin({
    beforeCreate () {
      // 判断组件是否存在 router 对象，该对象只在根组件上有
      if (isDef(this.$options.router)) {
        // 根路由设置为自己
        this._routerRoot = this
        // $options.router也就是new Vue应用初始化中传进来的router实例参数
        this._router = this.$options.router
        // 初始化路由，为初次进入页面运行
        this._router.init(this)
        // 很重要，为 _route 属性实现双向绑定
        // 兼容_router属性变化，触发组件渲染
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        // 为每个组件传递根组件，以便子组件也能否访问到路由实例
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      registerInstance(this, this)
    },
    destroyed () {
      registerInstance(this)
    }
  })
  
  // vue的原型设置一个getter，通过$router能够访问根路由实例
  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this._routerRoot._router }
  })

  // vue的原型设置一个getter，通过$route能够访问路由的实例
  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this._routerRoot._route }
  })
  
  // 全局注册组件 router-link 和 router-view
  Vue.component('RouterView', View)
  Vue.component('RouterLink', Link)
  
    const strats = Vue.config.optionMergeStrategies
  // use the same hook merging strategy for route hooks
  strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = strats.created
}
```

对于路由注册来说，核心就是调用 `Vue.use(VueRouter)`，使得 `VueRouter` 可以使用 Vue。然后通过 Vue 来调用 `VueRouter` 的 `install` 函数。

在`install`函数中，核心有以下几点：

- 给每个组件混入两个钩子函数（`beforeCreate`以及`destoryed`），在两个钩子函数中，进行私有属性和路由初始化工作
- 全局注册两个路由组件`router-link` 和 `router-view`
- 为vue的原型设置了两个属性`$router`(根组件实例)和`$route`（路由实例）
- 

