| 版本      | 起止日期  | 责任人 | 备注     |
| :-------- | --------- | ------ | -------- |
| V0.1/草稿 | 2020/8/19 | 李政   | 创建文档 |

[TOC]

# 介绍

本文参考`uni-app`[官网](https://uniapp.dcloud.io/README)学习记录所写。

## 什么是`uni-app`

`uni-app` 是一个使用 [Vue.js](https://vuejs.org/) 开发所有前端应用的框架，开发者编写一套代码，可发布到iOS、Android、H5、以及各种小程序（微信/支付宝/百度/头条/QQ/钉钉/淘宝）、快应用等多个平台。

## 开发环境

### `HBuilderX`

可视化开发`uni-app`的方式比较简单，直接使用`HBuilderX`开发即可，内置支持`uni-app`，开箱即用，只需要安装`HBuilderX`工具即可。

但是`HBuilderX`版本有区别：

- 标准版，在运行或者开发`uni-app`时，会提示安装`uni-app`插件，插件下载完成后方可使用
- APP开发版，开箱即用

#### 创建`uni-app`

1. 点击工具栏中的【文件->新建->项目】

2. 选择【`uni-app`类型->输入工程名称->选择`uni-app项目`模板->点击创建按钮】，即可成功创建。

#### 运行`uni-app`

由于后续开发主要是微信小程序，因此运行`uni-app`也只总结到微信开发者工具即可。

- 浏览器运行：点击工具栏的【运行->运行到浏览器->选择浏览器】，即可在浏览器中体验H5版本。

- 真机运行：连接手机，开启USB调试，进行项目点击工具栏【运行->运行到手机或模拟器->选择运行的设备】，即可在设备中体验`uni-app`

- 在微信开发者工具里运行：进入项目点击工具栏【运行->运行到小程序模拟器->微信开发者工具】，即可在微信开发者工具体验`uni-app`

  **注意**：如果是第一次使用，需要先配置小程序ide的相关路径，才能运行成功。如下图，需在输入框输入微信开发者工具的安装路径。 若HBuilderX不能正常启动微信开发者工具，需要开发者手动启动，然后将uni-app生成小程序工程的路径拷贝到微信开发者工具里面，在HBuilderX里面开发，在微信开发者工具里面就可看到实时的效果。

  uni-app默认把项目编译到根目录的unpackage目录。

  ![img](https://img-cdn-qiniu.dcloud.net.cn/uniapp/doc/weixin-setting.png)

#### 发布`uni-app`

目前只考虑微信小程序发布，其他发布见`uni-app`官网相关资料。

**打包成原生App（云端）**

工具栏点击【-发行->原生App-云打包】，在出现的界面总，点击打包即可。

**打包成原生App（离线）**

工具栏点击【发行->原生App-本地打包】点击即可生成离线打包资源

**发布为微信小程序**

1. 申请微信小程序AppID
2. 在`HBuilderX`中点击工具栏【发行->小程序-微信->输入小程序名称和AppID点击发行】，即可在`unpackage/dist/mp-weixin`生成微信小程序项目代码
3. 在微信小程序开发者工具中，导入生成的微信小程序项目，测试项目代码运行正常后，点击“上传”按钮，之后按照“提交审核”、“发布”小程序标准流程，逐步操作即可，详细查看：[微信官方教程](https://developers.weixin.qq.com/miniprogram/dev/quickstart/basic/release.html)。

### `vue-cli`

除了通过`HBuilderX`可视化界面，也可以使用`vue-cli`脚手架创建`uni-app`项目。

具体见[uni-app官网](https://uniapp.dcloud.io/README)。

## 框架简介

### 开发语言

`uni-app`使用的是vue的语法，不是小程序自定义的语法，所以如果了解vue，可以直接上手`uni-app`。

### 开发规范

为了实现多端兼容，综合考虑编译速度、运行性能等因素，`uni-app`约定了如下开发规范：

- 页面文件遵循[Vue 单文件组件(SFC)规范](https://vue-loader.vuejs.org/zh/spec.html)
- 组件标签靠近小程序规范，详见[uni-app组件规范](https://uniapp.dcloud.io/component/README)
- 接口能力靠近微信小程序规范，但是需要将前缀`wx`替换为`uni`，详见[uni-app接口规范](https://uniapp.dcloud.io/api/README)
- 数据绑定以及事件处理同`vue.js`规范，同时补充了App以及页面的生命周期
- 为兼容多端运行，建议使用`flex`布局进行开发

### 目录结构

`uni-app`工程，默认包含以下目录以及文件：

```
┌─cloudfunctions        云函数目录（阿里云为aliyun，腾讯云为tcb，详见uniCloud）
│─components            符合vue组件规范的uni-app组件目录
│  └─comp-a.vue         可复用的a组件
├─hybrid                存放本地网页的目录
├─platforms             存放各平台专用页面的目录
├─pages                 业务页面文件存放的目录
│  ├─index
│  │  └─index.vue       index页面
│  └─list
│     └─list.vue        list页面
├─static                存放应用引用静态资源（如图片、视频等）的目录，注意：静态资源只能存放于此
├─wxcomponents          存放小程序组件的目录
├─main.js               Vue初始化入口文件
├─App.vue               应用配置，用来配置App全局样式以及监听 应用生命周期
├─manifest.json         配置应用名称、appid、logo、版本等打包信息
└─pages.json            配置页面路由、导航条、选项卡等页面类信息
```

#### 注意事项

- `static`目录下的`js`文件不会被编译，如果里面有`es6`的代码，不经过转换直接运行，在手机设备上会报错
- `css`、`less/scss`等资源同样不要放在`static`目录下，建议这些公共的资源放在`common`目录下
- `HBuilderX`支持在根目录创建`ext.json`、`sitemap.json`文件

### 资源路径引入



# 框架

## 配置

### `pages.json`

`pages.json`文件用于对`uni-app`进行全局配置，决定页面文件的路径、窗口样式、原生的导航栏、底部的原生tabbar等。

类似于微信小程序中`app.json`中的页面管理部分。注意定位权限申请等原属于`app.json`的内容，在`uni-app`中是在`manifest`中配置。

**配置项列表**

| 属性                                                         | 类型         | 必填 | 描述                    | 平台兼容   |
| :----------------------------------------------------------- | :----------- | :--- | :---------------------- | :--------- |
| [globalStyle](https://uniapp.dcloud.io/collocation/pages?id=globalstyle) | Object       | 否   | 设置默认页面的窗口表现  |            |
| [pages](https://uniapp.dcloud.io/collocation/pages?id=pages) | Object Array | 是   | 设置页面路由及窗口表现  |            |
| [easycom](https://uniapp.dcloud.io/collocation/pages?id=easycom) | Object       | 否   | 组件自动引入规则        | 2.5.5+     |
| [tabBar](https://uniapp.dcloud.io/collocation/pages?id=tabbar) | Object       | 否   | 设置底部 tab 的表现     |            |
| [condition](https://uniapp.dcloud.io/collocation/pages?id=condition) | Object       | 否   | 启动模式配置            |            |
| [subPackages](https://uniapp.dcloud.io/collocation/pages?id=subpackages) | Object Array | 否   | 分包加载配置            |            |
| [preloadRule](https://uniapp.dcloud.io/collocation/pages?id=preloadrule) | Object       | 否   | 分包预下载规则          | 微信小程序 |
| [workers](https://developers.weixin.qq.com/miniprogram/dev/framework/workers.html) | String       | 否   | `Worker` 代码放置的目录 | 微信小程序 |

#### pages

`uni-app`通过pages节点配置应用由那些页面组成，pages节点接收一个数组，数组每项都是一个对象，属性值如下：

| 属性  | 类型   | 默认值 | 描述                                                         |
| :---- | :----- | :----- | :----------------------------------------------------------- |
| path  | String |        | 配置页面路径                                                 |
| style | Object |        | 配置页面窗口表现，配置项参考下方 [pageStyle](https://uniapp.dcloud.io/collocation/pages?id=style) |

**注意**：

- pages节点的第一项为应用入口页（也就是首页）
- 应用中增加/减少页面，都需要对pages数组进行修改
- 文件名不需要写后缀，框架会自动寻找路径下的页面资源

### `manifest`

`manifest`文件是应用的配置文件，用于指定应用的名称、图标、权限等。

**配置项列表**

| 属性           | 类型    | 默认值                                                       | 描述                                                         | 最低版本 |
| :------------- | :------ | :----------------------------------------------------------- | :----------------------------------------------------------- | :------- |
| name           | String  |                                                              | 应用名称                                                     |          |
| appid          | String  | 新建 uni-app 项目时，DCloud 云端分配。用途[详见](https://ask.dcloud.net.cn/article/35907) | 应用标识                                                     |          |
| description    | String  |                                                              | 应用描述                                                     |          |
| versionName    | String  |                                                              | 版本名称，例如：1.0.0。详见下方Tips说明                      |          |
| versionCode    | String  |                                                              | 版本号，例如：36                                             |          |
| transformPx    | Boolean | true                                                         | 是否转换项目的px，为true时将px转换为rpx，为false时，px为传统的实际像素 |          |
| networkTimeout | Object  |                                                              | 网络超时时间，[详见](https://uniapp.dcloud.io/collocation/manifest?id=networktimeout) |          |
| debug          | Boolean | false                                                        | 是否开启 debug 模式，开启后调试信息以 `info` 的形式给出，其信息有页面的注册，页面路由，数据更新，事件触发等 |          |
| uniStatistics  | Object  |                                                              | [是否开启 uni 统计，全局配置](https://uniapp.dcloud.io/collocation/manifest?id=unistatistics) | 2.2.3+   |
| app-plus       | Object  |                                                              | [App 特有配置](https://uniapp.dcloud.io/collocation/manifest?id=app-plus) |          |
| h5             | Object  |                                                              | [H5 特有配置](https://uniapp.dcloud.io/collocation/manifest?id=h5) |          |
| quickapp       | Object  |                                                              | 快应用特有配置，即将支持                                     |          |
| mp-weixin      | Object  |                                                              | [微信小程序特有配置](https://uniapp.dcloud.io/collocation/manifest?id=mp-weixin) |          |
| mp-alipay      | Object  |                                                              | [支付宝小程序未提供可配置项](https://uniapp.dcloud.io/collocation/manifest?id=mp-alipay) |          |
| mp-baidu       | Object  |                                                              | [百度小程序特有配置](https://uniapp.dcloud.io/collocation/manifest?id=mp-baidu) |          |
| mp-toutiao     | Object  |                                                              | [字节跳动小程序特有配置](https://uniapp.dcloud.io/collocation/manifest?id=mp-toutiao) | 1.6.0    |
| mp-qq          | Object  |                                                              | [qq 小程序特有配置](https://uniapp.dcloud.io/collocation/manifest?id=mp-qq) | 2.1.0    |

**注意**：

- uni-app 的 `appid` 由 DCloud 云端分配，主要用于 DCloud 相关的云服务，请勿自行修改。[详见](https://ask.dcloud.net.cn/article/35907)
- 注意区分 uni-app 的 `appid` 与微信小程序、iOS 等其它平台分配的 `appid`，以及第三方 SDK 的 `appid`。

#### networkTimeout

自`HBuilderX 2.5.10`起，默认超时时间由6秒改为60秒，对齐微信小程序平台。

| 属性          | 类型   | 必填 | 默认值 | 说明                                     |
| ------------- | ------ | ---- | ------ | ---------------------------------------- |
| request       | Number | 否   | 60000  | uni.request 的超时时间，单位毫秒。       |
| connectSocket | Number | 否   | 60000  | uni.connectSocket 的超时时间，单位毫秒。 |
| uploadFile    | Number | 否   | 60000  | uni.uploadFile 的超时时间，单位毫秒。    |
| downloadFile  | Number | 否   | 60000  | uni.downloadFile 的超时时间，单位毫秒。  |

#### 微信小程序配置

| 属性                           | 类型    | 说明                                                         |
| :----------------------------- | :------ | :----------------------------------------------------------- |
| appid                          | String  | 微信小程序的AppID，登录 [https://mp.weixin.qq.com](https://mp.weixin.qq.com/) 申请 |
| usingComponents                | Boolean | 是否启用自定义组件模式，`v1.8.0+`，默认为false，[编译模式区别详情](https://ask.dcloud.net.cn/article/35843) |
| setting                        | Object  | 微信小程序项目设置，参考[setting](https://uniapp.dcloud.io/collocation/manifest?id=setting) |
| functionalPages                | Boolean | 微信小程序是否启用插件功能页，默认关闭                       |
| requiredBackgroundModes        | Array   | 微信小程序需要在后台使用的能力,[详见](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html#requiredbackgroundmodes) |
| plugins                        | Object  | 使用到的插件，[详见](https://developers.weixin.qq.com/miniprogram/dev/framework/plugin/using.html) |
| resizable                      | Boolean | 在iPad上小程序是否支持屏幕旋转，默认关闭                     |
| navigateToMiniProgramAppIdList | Array   | 需要跳转的小程序列表，[详见](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/miniprogram-navigate/wx.navigateToMiniProgram.html) |
| permission                     | Object  | 微信小程序接口权限相关设置，比如申请位置权限必须填此处[详见](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html) |
| workers                        | String  | Worker 代码放置的目录。 [详见](https://developers.weixin.qq.com/miniprogram/dev/framework/workers.html) |
| optimization                   | Object  | 对微信小程序的优化配置                                       |
| cloudfunctionRoot              | String  | 配置云开发目录，参考[setting](https://uniapp.dcloud.io/collocation/manifest?id=cloudfunctionroot) |
| uniStatistics                  | Object  | [微信小程序是否开启 uni 统计，配置方法同全局配置](https://uniapp.dcloud.io/collocation/manifest?id=unistatistics) |

##### setting

编译到微信小程序平台下的项目设置。

| 属性     | 类型    | 说明                        |
| :------- | :------ | :-------------------------- |
| urlCheck | Boolean | 是否检查安全域名和 TLS 版本 |
| es6      | Boolean | ES6 转 ES5                  |
| postcss  | Boolean | 上传代码时样式是否自动补全  |
| minified | Boolean | 上传代码时是否自动压缩      |

##### usingComponents

`usingComponents`字段是否启用`自定义组件模式`，为true表示新的`自定义组件模式`，否则为`template模板模式`。 

`uni-app`自从`1.8版本`开始，同时支持两种编译模式：

- `template`模式，老框架模式，借鉴自`mpvue`，将用户编写的`Vue`组件，编译为`WXML`中的[模板（template)](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxml/template.html)，变相实现组件化开发，性能差，支持 Vue 语法少，比如不支持过滤器。
- `自定义组件模式`：新框架模式，DCloud自研，将用户编写的`Vue`组件，编译为微信小程序[自定义组件](https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/)，实现了**更高的性能**及**更完善的 Vue 语法支持**。同时在App端提供了独立的js引擎，**大幅提升性能**。

### `package.json`

`package.json`文件中增加了`uni-app`扩展节点的支持，可实现自定义条件编译平台。

```json
{
    /**
     package.json其它原有配置 
     */
    "uni-app": {// 扩展配置
        "scripts": {
            "custom-platform": { //自定义编译平台配置，可通过cli方式调用
                "title":"自定义扩展名称", // 在HBuilderX中会显示在 运行/发行 菜单中
                "BROWSER":"",  //运行到的目标浏览器，仅当UNI_PLATFORM为h5时有效
                "env": {//环境变量
                    "UNI_PLATFORM": ""  //基准平台 
                 },
                "define": { //自定义条件编译
                    "CUSTOM-CONST": true //自定义条件编译常量，建议为大写
                }
            }
        }    
    }
}
```

**注意**：

- `UNI_PLATFORM`仅支持填写`uni-app`默认支持的基准平台，目前仅限如下枚举值：`h5`、`mp-weixin`、`mp-alipay`、`mp-baidu`、`mp-toutiao`、`mp-qq`
- `BROWSER` 仅在`UNI_PLATFORM`为`h5`时有效,目前仅限如下枚举值：`Chrome`、`Firefox`、`IE`、`Edge`、`Safari`、`HBuilderX`
- `package.json`文件中不允许出现注释，否则扩展配置无效
- `vue-cli`需更新到最新版，`HBuilderX`需升级到 2.1.6+ 版本

### `vue.config.js`

vue.config.js 是一个可选的配置文件，如果项目的根目录中存在这个文件，那么它会被自动加载，一般用于配置 webpack 等编译选项，具体规范参考：[vue.config.js](https://cli.vuejs.org/zh/config/#vue-config-js)

但是有些配置项在`uni-app`中不支持，[详见](https://uniapp.dcloud.io/collocation/vue-config)

#### 发布时删除console

`HBuilderX 2.6.8+`支持

```js
module.exports = {
    chainWebpack: (config) => {
        // 发行或运行时启用了压缩时会生效
        config.optimization.minimizer('terser').tap((args) => {
            const compress = args[0].terserOptions.compress
            // 非 App 平台移除 console 代码(包含所有 console 方法，如 log,debug,info...)
            compress.drop_console = true
            compress.pure_funcs = [
                '__f__', // App 平台 vue 移除日志代码
                // 'console.debug' // 可移除指定的 console 方法
            ]
            return args
        })
    }
}
```

由于这个功能需要启用压缩才能生效，启用压缩的方法如下：

- HBuilderX创建的项目勾选运行-->运行到小程序模拟器-->运行时是否压缩代码
- cli创建的项目可以在`pacakge.json`中添加参数`--minimize`，示例：`"dev:mp-weixin": "cross-env NODE_ENV=development UNI_PLATFORM=mp-weixin vue-cli-service uni-build --watch --minimize"`

### `app.vue`

`app.vue`是`uni-app`的主组件，所有页面都是在`app.vue`下进行切换的，是页面的入口文件，但`app.vue`本身不是页面，不能编写视图元素。

这个文件的作用包括：调用应用生命周期函数、配置全局样式、配置全局的存储globalData。

**应用的生命周期只能在`app.vue`中监听，在页面监听无效**。

#### 应用生命周期

`uni-app`支持的生命周期函数如下：

| 函数名               | 说明                                                         | 平台兼容 |
| :------------------- | :----------------------------------------------------------- | :------- |
| onLaunch             | 当`uni-app` 初始化完成时触发（全局只触发一次）               |          |
| onShow               | 当 `uni-app` 启动，或从后台进入前台显示                      |          |
| onHide               | 当 `uni-app` 从前台进入后台                                  |          |
| onError              | 当 `uni-app` 报错时触发                                      |          |
| onUniNViewMessage    | 对 `nvue` 页面发送的数据进行监听，可参考 [nvue 向 vue 通讯](https://uniapp.dcloud.io/use-weex?id=nvue-向-vue-通讯) | App      |
| onUnhandledRejection | 对未处理的 Promise 拒绝事件监听函数（2.8.1+）                |          |
| onPageNotFound       | 页面不存在监听函数                                           |          |
| onThemeChange        | 监听系统主题变化                                             |          |

##### `onlaunch`跳转页面白屏

【问题分析】

如果在`onlaunch`里面进行页面跳转，会和`pages.json`中配置的第一个页面跳转时机冲突，可能会导致手机端页面白屏。

【解决方案】

需要添加延迟做跳转处理。

由于性能优化，`HBuilderX 1.9.8`和`HBuilderX1.9.4`的执行时机有所不同。一些在`HBuilderX 1.9.4`下无需延时的代码，在升级到`HBuilderX 1.9.8`报错。先请延迟处理。

在`HBuilderX 1.9.9+`版本，已在底层修复此问题，自动兼容冲突，无需开发者再写延时代码。

如果使用新版的开发者有类似问题，已经不是这个问题了。

##### globalData

globalData就是一种简单的全局变量机制。

```js
<script>  
    export default {  
        globalData: {  
            text: 'text'  
        }
    }  
</script>
```

js中操作globalData的方式如下：`getApp().globalData.text = 'text'`

在应用onLaunch时，getApp对象还未获取，暂时可以使用this.$scope.globalData获取globalData。

如果需要把globalData的数据绑定到页面上，可在页面的onShow页面生命周期里进行变量重赋值。

globalData是简单的全局变量，如果使用状态管理，请使用`vuex`（main.js中定义）

##### 全局样式

在`app.vue`中，可以定义全局通用样式，例如需要加一个通用的背景色，首屏页面渲染的动画等都可以卸载`app.vue`中。

### `main.js`

`main.js`是`uni-app`的入口文件，主要作用是初始化`vue`实例，定义全局组件，使用需要的插件，比如`vuex`。

首先引入了`vue`和`app.vue`，创建一个`vue`实例，并且挂载`vue`实例。

```js
import Vue from 'vue'
import App from './App'
import pageHead from './components/page-head.vue' //全局引用page-head组件

Vue.config.productionTip = false
Vue.component('page-head', pageHead) //全局注册page-head组件，每个页面将可以直接使用该组件
App.mpType = 'app'

const app = new Vue({
    ...App
})
app.$mount() //挂载Vue实例
```

使用`Vue.use`引用插件，使用`Vue.prototype`添加全局变量，使用`Vue.component`注册全局组件。

可以引用`vuex`，因涉及多个文件，此处没有提供示例，详见`hello uni-app`示例工程。

无法使用`vue-router`，路由须在`pages.json`中进行配置。如果开发者坚持使用`vue-router`，可以在[插件市场](https://ext.dcloud.net.cn/search?q=vue-router)找到转换插件。

## 框架接口

### 日志打印

通过`console`向控制台打印日志信息，但是由于不同平台对`console`方法的支持存在差异，因此建议开发过程中只适用以下方法：

- info
- log
- debug
- warn
- error

`HBuilderX`可以通过代码块快速输入。

- 输入`clog`，可以直接输出`console.log()`
- 输入`clogv`，可以直接输出`console.log(": " +)`，并出现双光标，方便把变量名称和值同时打印
- 输入`clogj`，可以直接输出`console.log(": " + JSON.stringify())`，并出现双光标，方便把json对象转为字符串打印出来

### 生命周期

#### 应用生命周期

详见[应用生命周期](#应用生命周期)

#### 页面生命周期

`uni-app`支持的页面生命周期函数如下：

| 函数名                              | 说明                                                         | 平台差异说明                                                 | 最低版本 |
| :---------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :------- |
| onLoad                              | 监听页面加载，其参数为上个页面传递的数据，参数类型为Object（用于页面传参），参考[示例](https://uniapp.dcloud.io/api/router?id=navigateto) |                                                              |          |
| onShow                              | 监听页面显示。页面每次出现在屏幕上都触发，包括从下级页面点返回露出当前页面 |                                                              |          |
| onReady                             | 监听页面初次渲染完成。注意如果渲染速度快，会在页面进入动画完成前触发 |                                                              |          |
| onHide                              | 监听页面隐藏                                                 |                                                              |          |
| onUnload                            | 监听页面卸载                                                 |                                                              |          |
| onResize                            | 监听窗口尺寸变化                                             | App、微信小程序                                              |          |
| onPullDownRefresh                   | 监听用户下拉动作，一般用于下拉刷新，参考[示例](https://uniapp.dcloud.io/api/ui/pulldown) |                                                              |          |
| onReachBottom                       | 页面滚动到底部的事件（不是scroll-view滚到底），常用于下拉下一页数据。具体见下方注意事项 |                                                              |          |
| onTabItemTap                        | 点击 tab 时触发，参数为Object，具体见下方注意事项            | 微信小程序、支付宝小程序、百度小程序、H5、App（自定义组件模式） |          |
| onShareAppMessage                   | 用户点击右上角分享                                           | 微信小程序、百度小程序、字节跳动小程序、支付宝小程序         |          |
| onPageScroll                        | 监听页面滚动，参数为Object                                   | nvue暂不支持                                                 |          |
| onNavigationBarButtonTap            | 监听原生标题栏按钮点击事件，参数为Object                     | App、H5                                                      |          |
| onBackPress                         | 监听页面返回，返回 event = {from:backbutton、 navigateBack} ，backbutton 表示来源是左上角返回按钮或 android 返回键；navigateBack表示来源是 uni.navigateBack ；详细说明及使用：[onBackPress 详解](http://ask.dcloud.net.cn/article/35120)。支付宝小程序只有真机能触发，只能监听非navigateBack引起的返回，不可阻止默认行为。 | app、H5、支付宝小程序                                        |          |
| onNavigationBarSearchInputChanged   | 监听原生标题栏搜索输入框输入内容变化事件                     | App、H5                                                      | 1.6.0    |
| onNavigationBarSearchInputConfirmed | 监听原生标题栏搜索输入框搜索事件，用户点击软键盘上的“搜索”按钮时触发。 | App、H5                                                      | 1.6.0    |
| onNavigationBarSearchInputClicked   | 监听原生标题栏搜索输入框点击事件                             | App、H5                                                      | 1.6.0    |
| onShareTimeline                     | 监听用户点击右上角转发到朋友圈                               | 微信小程序                                                   | 2.8.1+   |
| onAddToFavorites                    | 监听用户点击右上角收藏                                       | 微信小程序                                                   | 2.8.1+   |









# 问题

## 微信登录开发者工具显示“网络连接失败”

### 问题描述

安装微信开发者工具后，使用微信扫码登录开发者工具，显示“网络连接失败”

### 问题原因

猜测跟网络权限有关，经过询问刘工，得知需要提升外网权限，需要开通微信平台的外网权限。

### 解决方案

申请微信平台的外网权限即可。

## 无法运行在微信开发者工具

### 问题描述

在`HBuilderX`中点击【运行->运行到小程序模拟器->选择微信开发者工具】，发现编译报错：

```
10:31:09.691 正在启动微信开发者工具...
10:31:10.018 [微信小程序开发者工具] [error] IDE service port disabled. To use CLI Call, please enter y to confirm enabling CLI capability, or manually open IDE -> Settings -> Security Settings, and set Service Port On.
10:31:10.025 [微信小程序开发者工具] For more details see: https://developers.weixin.qq.com/miniprogram/en/dev/devtools/cli.html
10:31:10.026 [微信小程序开发者工具] [error] 工具的服务端口已关闭。要使用命令行调用工具，请在下方输入 y 以确认开启，或手动打开工具 -> 设置 -> 安全设置，将服务端口开启。
10:31:10.031 [微信小程序开发者工具] 详细信息: https://developers.weixin.qq.com/miniprogram/dev/devtools/cli.html
10:31:10.032 [微信小程序开发者工具]
10:31:10.040 [微信小程序开发者工具] ? Enable IDE Service (y/N) [27D[27C
10:31:10.046 [微信小程序开发者工具] - initialize
10:31:10.054 [微信小程序开发者工具]
10:31:10.057 [微信小程序开发者工具] × initialize
10:31:10.062 [微信小程序开发者工具]
10:31:10.068 [微信小程序开发者工具] Runtime error
10:31:10.076 [微信小程序开发者工具] Error: read EBADF
10:31:10.081 [微信小程序开发者工具]     at Pipe.onStreamRead (internal/stream_base_commons.js:183:27) {
10:31:10.090 [微信小程序开发者工具]   errno: 'EBADF',
10:31:10.091 [微信小程序开发者工具]   code: 'EBADF',
10:31:10.096 [微信小程序开发者工具]   syscall: 'read'
10:31:10.101 [微信小程序开发者工具] }
10:31:10.102 [微信小程序开发者工具]
10:31:10.108 [微信小程序开发者工具]
```

### 问题原因

微信小程序开发者工具的服务端口没有开启

### 解决方案

打开微信开发者工具，点击工具栏中的【设置->安全设置->打开服务端口】即可，验证有效。

## 



# 优化

- package.json中添加uni-app的扩展节点
- `Vue.prototype.$store = store;`在`main.js`中的必要性
- 

 
