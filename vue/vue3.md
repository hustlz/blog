[TOC]

# vue 3.0 

## 值得注意的新特性

vue3中需要关注的一些新功能包括：

- 组合式API
- teleport
- 片段
- 触发组件选项
- `createRenderer`API来自`@vue/runtime-core`创建自定义渲染器
- 单文件组件组合式API语法糖
- 单文件组件状态驱动的css变量
- 单文件组件`<style scoped>`现在可以包含全局规则，或值针对插槽内容的规则

## 非兼容更改

### v-for中的Ref数组

**vue2**

Vue2中在`v-for`里使用`ref`属性时，会用ref数组填充响应的`$refs`变量，这种行为，当存在嵌套的`v-for`时，不仅不准确，而且效率低。

**vue3**

在vue3中，这样的用法将不支持，如果想要从单个绑定获取多个ref，需要将`ref`绑定到一个灵活的函数上（新特性）

```vue
// template
<div v-for="item in list" :ref="setItemRef">
```

结合选项式API的写法如下：

```js
export default {
  data() {
    return {
      itemRefs: []
    }
  },
  methods: {
    setItemRef(el) {
      this.itemRefs.push(el)
    }
  },
  beforeUpdate() {
    this.itemRefs = []
  },
  updated() {
    console.log(this.itemRefs)
  }
}
```

结合组合式API写法如下：

```
import { ref, onBeforeUpdate, onUpdated } from 'vue'

export default {
  setup() {
    let itemRefs = []
    const setItemRef = el => {
      itemRefs.push(el)
    }
    onBeforeUpdate(() => {
      itemRefs = []
    })
    onUpdated(() => {
      console.log(itemRefs)
    })
    return {
      itemRefs,
      setItemRef
    }
  }
}
```

注意：

- `itemRef`不需要是数组，也可以是一个对象
- 如果需要，`itemRef`也可以是响应式的且可以被监听

### attribute强制行为

> 这是一个低级的内部API更改，不会影响大多数开发人员。

**vue2**

在vue2的版本中，v-bind的attribute的值会有几个策略进行强制转换：

- 对于有些attribute对，vue强制必须使用对应的attribute绑定，比如：input、select、textarea等组件的value属性
- 对于`布尔attribute`和`xlinks`，如果设置的值是`falsy`，vue会移除这些属性的设置（比如：undefined，null，or false）
- 







## 新增特性

### 异步组件

**vue2**

在大型应用里面，一般会将模块分割出来进行按需加载，也就是异步组件。

```js
// ES6
() => import('./my-async-component')

// 这个特殊的 `require` 语法将会告诉 webpack，自动将你的构建代码切割成多个包
require(['./my-async-component'], resolve)
```

异步组件工厂函数也可以携带选项：

```js
const AsyncComponent = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 3000
})
```

**vue3**

在vue3中，由于函数式组件被定义为纯函数，因此定义异步组件，需要通过新增的特性``defineAsyncComponent` `来显式定义。

```js
import { defineAsyncComponent } from 'vue'
import ErrorComponent from './components/ErrorComponent.vue'
import LoadingComponent from './components/LoadingComponent.vue'

// 不带选项的异步组件
const asyncPage = defineAsyncComponent(() => import('./NextPage.vue'))

// 带选项的异步组件
const asyncPageWithOptions = defineAsyncComponent({
  // component选项修改为loader，而且loader不再接受resolve和reject参数，且必须始终返回promise
  loader: () => import('./NextPage.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```



