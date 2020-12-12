[TOC]

# 移动端兼容

## 小程序

### `scroll-view`在部分IOS手机上无法滑动

【问题描述】

在部分IOS手机上，scroll-view内容超出设置高度后无法滑动。

【问题原因】

经过跟踪发现scroll-view在部分IOS上无法兼容下拉刷新，和滑动事件。

【解决方案】

使用touchmove事件监听实现下拉刷新功能，并封装成组件，组件内部使用scroll-view实现内部滚动即可。

### input组件设置placeholder样式不能设置opacity

【问题描述】

```html
<input type='text' placeholder-style="color:#000;opacity:0.4;" style="color:#000;opacity:0.8">
```

手机上，input聚焦时，placeholder会显示为透明度为0.8，而不是0.4

【问题原因】

微信原生input组件设置placeholder的样式时，不能设置opacity属性，否则会在聚焦时产生穿透，导致placeholder的颜色发生错乱。

【解决方案】

设置placeholder样式时，不设置opacity属性

###  textarea组件无法被modal覆盖

【问题描述】

小程序使用了textarea组件后，弹出层，或者弹窗无法覆盖textarea的内容，会导致内容出现穿透

【问题原因】

微信小程序的textarea组件层级太高，导致其他元素无法覆盖，就算z-index设置为9999也没有用

【解决方案】

出现弹窗的时候，将textarea组件替换为view显示

## H5



## 小程序 && H5

### ios最后一个margin-bottom无效

【问题描述】

ios手机开发小程序，以及H5应用时，最后一个节点的`margin-bottom`无效。

【问题原因】

IOS手机兼容性问题。

【解决方案】

使用`padding-bottom`，或者设置高度为所需高度的分割条。

### 0.5px分割线在部分手机特殊情况下绘制不出来

【问题描述】

使用border或者height实现0.5px的分割线时，有的时候能显示，有的时候分割线突然消失。

【问题原因】

不清楚具体原因，不过根据调试发现，如果设置的高度不是整数的情况下，会导致分割线显示不完整。

【解决方案】

如果高度不是整数时，尽量设置整数的高度，凑整。

# PC浏览器兼容

## 通有问题

### enter按键会刷新当前页面

【问题描述】

填写表单时，按下enter按键，会导致当前页面被刷新

【问题原因】

浏览器默认行为。

如果form表单中只有一个input时，enter按键会自动触发submit行为提交表单，而form表单的action为空时，会刷新当前页面

【解决方案】

方案一：

给form表单绑定submit事件，return false；或者使用prevent阻止默认行为

方案二：

给input绑定keydown事件，监听keyCode为13时，阻止默认行为

## IE



## safari

### 使用blob下载文件时，低版本safari上无法下载

【问题描述】

使用blob接受二进制流文件，保存下载文件的操作，在低版本的safari上无法下载

【问题原因】

低版本的safari不兼容blob，导致无法下载文件

【解决方案】

后台提供要下载的文件url给前端，使用a标签下载即可
