| 版本      | 起止日期  | 责任人 | 备注     |
| :-------- | --------- | ------ | -------- |
| V0.1/草稿 | 2020/8/19 | 李政   | 创建文档 |

[TOC]

# 简介

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

1. 点击工具栏中的文件->新建->项目：

![img](https://img-cdn-qiniu.dcloud.net.cn/uniapp/doc/create1.png)

2. 选择`uni-app`类型，输入工程名称，选择`uni-app项目`模板，点击创建按钮，即可成功创建。

![img](https://img.cdn.aliyun.dcloud.net.cn/uni-app/doc/create.png)

#### 运行`uni-app`

由于后续开发主要是微信小程序，因此运行`uni-app`也只总结到微信开发者工具即可。

- 浏览器运行：点击工具栏的【运行->运行到浏览器->选择浏览器】，即可在浏览器中体验H5版本。

![img](https://img-cdn-qiniu.dcloud.net.cn/uniapp/doc/run-chrome.png)

- 真机运行：连接手机，开启USB调试，进行项目点击工具栏【运行->运行到手机或模拟器->选择运行的设备】，即可在设备中体验`uni-app`

![img](https://img-cdn-qiniu.dcloud.net.cn/uniapp/doc/run-phone.png)

- 在微信开发者工具里运行：进入项目点击工具栏【运行->运行到小程序模拟器->微信开发者工具】，即可在微信开发者工具体验`uni-app`

  ![](https://img-cdn-qiniu.dcloud.net.cn/uniapp/doc/uni20190222-1.png)

  **注意**：如果是第一次使用，需要先配置小程序ide的相关路径，才能运行成功。如下图，需在输入框输入微信开发者工具的安装路径。 若HBuilderX不能正常启动微信开发者工具，需要开发者手动启动，然后将uni-app生成小程序工程的路径拷贝到微信开发者工具里面，在HBuilderX里面开发，在微信开发者工具里面就可看到实时的效果。

  uni-app默认把项目编译到根目录的unpackage目录。

  ![img](https://img-cdn-qiniu.dcloud.net.cn/uniapp/doc/weixin-setting.png)

#### 发布`uni-app`

**打包成原生App（云端）**

工具栏点击【】

**打包成原生App（离线）**



### `vue-cli`

# 问题





 