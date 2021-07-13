[TOC]

# uniapp平台toast规范开发

## 需求

说明：一种轻量级反馈/提示，可以用来呈现不会打断用户操作的内容。“加载提示”一般位于页面中轴线上，停留短暂的时间后自动消失。多用于当用户的操作需要得到反馈或当前状态变更需告知/提醒用户。包含的视觉元素有文本和图标。

设计原则：一次只显示一个 toast，显示3s后自动消失；

纯文本 Toast，字数建议两行以内；

图标 Toast，分两种情况（常规、和文字较多时），图标大小为24px

背景颜色：0F1214 不透明度80%，圆角统一为8px

有图标时：图标位于文字上方，与文字垂直居中对齐，toast距离页面左右间距最小为40px，超出最大宽度内容折行显示

## 需求分析

一般我们在项目开发中，都会需要使用toast进行提示，比如全局接口请求库，业务模块等内容。我们期望直接调用toast方法即可，toast属于一种轻量级反馈，呈现不会打断用户操作的内容，如下所示。

```js
this.toast.show({
	title: 'toast显示'
})
```

但是，在微信小程序平台，和APP平台不支持render，只能使用uniapp提供的原生组件`uni.showToast`方法，这个方法的图标很大，而且很丑，自定义图标的显示会显示异常（图标和文字挤在一起，没有美感）

>`uni-app`只支持 vue单文件组件（.vue 组件）。其他的诸如：动态组件，自定义 `render` ，和 `<script type="text/x-template">` 字符串模版等，在非H5端不支持。

如果我们使用自定义组件的方式，那么每一个页面都需要在template中写一个`<toast title="toast显示"/>`来显示toast，这个是我们无法忍受的，而且会导致基础组件和业务严重耦合在一起。

## 方案设计

目前我们的uniapp开发会涉及到微信小程序、APP、H5三个平台。

由于微信小程序是单独运行的，无法支持直接js显示符合我们需求的toast，因为只能直接去除icon的显示；

APP小程序是内嵌在APP内部的，因此可以使用APP的能力支持显示对应的toast；

H5端可以使用render，因此可以自己实现符合需求的toast。

## 技术实现

封装后的toast出口统一从下述文件中导出，目前只暴露了最常见的几个参数，title、icon、duration、以及mask。

```js
// #ifdef H5
import toast from '../../components/u-toast/index.js';
// #endif

export default {
    // options: {
    //     title: 提示的内容
    //     icon： 显示的图标类型传递iconfont的名称即可，（只有APP和H5支持）
    //     duration： 提示的延迟时间
    //     mask： 是否显示透明蒙层，防止触摸穿透
    // }
    show(options) {
        // #ifdef APP-PLUS
        // TODO: APP需要使用APP的能力提供toast，暂时使用uni.showToast
        uni.showToast({
            title: options.title,
            icon: 'none',
            duration: options.duration || 3000,
            mask: options.mask || false
        });
        // #endif
        // #ifdef MP-WEIXIN
        // 微信小程序toast不需要使用图标
        uni.showToast({
            title: options.title,
            icon: 'none',
            duration: options.duration || 3000,
            mask: options.mask || false
        });
        // #endif
        // #ifdef H5
        // H5目前采用自定义的laoding组件即可。
        toast.show(options);
        // #endif
    },
    hide() {
        // #ifdef APP-PLUS
        // TODO: APP需要使用APP的能力提供toast，暂时使用uni.showToast
        uni.hideToast();
        // #endif
        // #ifdef MP-WEIXIN
        // 微信小程序toast不需要使用图标
        uni.hideToast();
        // #endif
        // #ifdef H5
        // H5目前采用自定义的laoding组件即可。
        toast.hide();
        // #endif
    }
}
```

### H5 toast组件封装

index.js

```js
import Vue from 'vue'
import loadingComponent from './index.vue'
const LoadingConstructor = Vue.extend(loadingComponent)
const instance = new LoadingConstructor({
	el: document.createElement('div')
});

instance.show = false // 默认隐藏
const loading = {
	show(options) { // 显示方法
		instance.showToast(options);
		document.body.appendChild(instance.$el);
	},
	hide() { // 隐藏方法
		instance.hideToast();
	}
}
export default loading;
```

index.vue

```vue
<template>
    <view
        class="u-toast" 
        :class="[
            isShow ? 'u-show' : '',
            'u-type-' + tmpConfig.type,
            'u-position-' + tmpConfig.position]"
        :style="{
            zIndex: uZIndex,
            width: toastWidth
        }"
    >
        <view class="u-icon-wrap" >
            <u-icon class="u-icon" v-if="tmpConfig.iconName" :name="tmpConfig.iconName" :size="48" :color="tmpConfig.type"></u-icon>
            <text class="u-text" :class="[tmpConfig.iconName?'':'iconHeight']">{{tmpConfig.title}}</text>
        </view>
        <view class="hide-view" ref="hideView">{{tmpConfig.title}}</view>
    </view>
</template>
<script>
export default {
    name: 'custom-toast',
    data() {
        return {
            isShow: false,
            timer: null, // 定时器
            config: {
                iconName: '', //icon名称
                params: {}, // URL跳转的参数，对象
                title: '', // 显示文本
                type: '', // 主题类型，primary，success，error，warning，black
                duration: 3000, // 显示的时间，毫秒
                isTab: false, // 是否跳转tab页面
                url: '', // toast消失后是否跳转页面，有则跳转，优先级高于back参数
                position: 'center', // toast出现的位置
                callback: null, // 执行完后的回调函数
                back: false, // 结束toast是否自动返回上一页
            },
            tmpConfig: {}, // 将用户配置和内置配置合并后的临时配置变量
			zIndex: '', // z-index值
            toastWidth: undefined
        };
    },
    computed: {
        uZIndex() {
            // 显示toast时候，如果用户有传递z-index值，有限使用
            return this.isShow ? (this.zIndex ? this.zIndex : this.$u.zIndex.toast) : '999999';
        }
    },
    methods: {
        // 显示toast组件，由父组件通过this.$refs.xxx.show(options)形式调用
        showToast(options) {
            // 不降结果合并到this.config变量，避免多次条用u-toast，前后的配置造成混论
            this.tmpConfig = this.$u.deepMerge(this.config, options);
            if (this.timer) {
                // 清除定时器
                clearTimeout(this.timer);
                this.timer = null;
            }
            this.isShow = true;
            this.$nextTick(() => {
                let textRect = this.$refs.hideView.$el.getBoundingClientRect();
                let toastWrapWidth = textRect.width + 40;
                this.toastWidth = toastWrapWidth * 2 + 'rpx';
            })
            
            this.timer = setTimeout(() => {
                // 倒计时结束，清除定时器，隐藏toast组件
                this.isShow = false;
                clearTimeout(this.timer);
                this.timer = null;
                // 判断是否存在callback方法，如果存在就执行
                typeof(this.tmpConfig.callback) === 'function' && this.tmpConfig.callback();
                this.timeEnd();
            }, this.tmpConfig.duration);
        },
        // 隐藏toast组件，由父组件通过this.$refs.xxx.hide()形式调用
        hideToast() {
            this.isShow = false;
            if (this.timer) {
                // 清除定时器
                clearTimeout(this.timer);
                this.timer = null;
            }
        },
        // 倒计时结束之后，进行的一些操作
        timeEnd() {
            // 如果带有url值，根据isTab为true或者false进行跳转
            if (this.tmpConfig.url) {
                // 如果url没有"/"开头，添加上，因为uni的路由跳转需要"/"开头
                if (this.tmpConfig.url[0] != '/') this.tmpConfig.url = '/' + this.tmpConfig.url;
                // 判断是否有传递显式的参数
                if (Object.keys(this.tmpConfig.params).length) {
                    // 判断用户传递的url中，是否带有参数
                    // 使用正则匹配，主要依据是判断是否有"/","?","="等，如“/page/index/index?name=mary"
                    // 如果有params参数，转换后无需带上"?"
                    let query = '';
                    if (/.*\/.*\?.*=.*/.test(this.tmpConfig.url)) {
                        // object对象转为get类型的参数
                        query = this.$u.queryParams(this.tmpConfig.params, false);
                        this.tmpConfig.url = this.tmpConfig.url + "&" + query;
                    } else {
                        query = this.$u.queryParams(this.tmpConfig.params);
                        this.tmpConfig.url += query;
                    }
                }
                // 如果是跳转tab页面，就使用uni.switchTab
                if (this.tmpConfig.isTab) {
                    uni.switchTab({
                        url: this.tmpConfig.url
                    });
                } else {
                    uni.navigateTo({
                        url: this.tmpConfig.url
                    });
                }
            } else if(this.tmpConfig.back) {
                // 回退到上一页
                this.$u.route({
                    type: 'back'
                })
            }
        }
    }
}
</script>

<style lang="scss" scoped>
	@import "../../libs/css/style.components.scss";
	.u-toast {
		position: fixed;
		z-index: -1;
		transition: opacity 0.3s;
		text-align: center;
		color: #fff;
		border-radius: 16rpx;
		background: $header-text-color;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		font-size: 34rpx;
		opacity: 0;
		pointer-events: none;
		padding: 40rpx;
        min-width: 192rpx;
        max-width: calc(100% - 160rpx);
	}

    .u-toast.u-show {
		opacity: 0.87;
	}

	.u-icon {
		@include vue-flex;
		align-items: center;
		line-height: normal;
		margin-bottom: 28rpx;
	}

	.u-icon-wrap {
		display: flex;
		flex-direction: column;
		justify-content: center;
	}
	
	.u-text{
		font-size: 34rpx;
		text-align: center;
		line-height: 48rpx;
        word-break: break-all;
	}

	.u-position-center {
		left: 50%;
		top: 50%;
		transform: translate(-50%,-50%);
	}

	.u-position-top {
		left: 50%;
		top: 20%;
		transform: translate(-50%,-50%);
	}

	.u-position-bottom {
		left: 50%;
		bottom: 20%;
		transform: translate(-50%,-50%);
	}

	.u-type-primary {
		color: $u-type-primary;
		background-color: $u-type-primary-light;
		border: 1px solid rgb(215, 234, 254);
	}

	.u-type-success {
		color: $u-type-success;
		background-color: $u-type-success-light;
		border: 1px solid #BEF5C8;
	}

	.u-type-error {
		color: $u-type-error;
		background-color: $u-type-error-light;
		border: 1px solid #fde2e2;
	}

	.u-type-warning {
		color: $u-type-warning;
		background-color: $u-type-warning-light;
		border: 1px solid #faecd8;
	}

	.u-type-info {
		color: $u-type-info;
		background-color: $u-type-info-light;
		border: 1px solid #ebeef5;
	}

	.u-type-default {
		color: #fff;
		background-color: #585858;
	}

    .hide-view {
        opacity: 0;
        font-size: 34rpx;
        line-height: 48rpx;
        position: fixed;
        top: -9999rpx;
        left: -9999rpx;
        white-space: nowrap;
    }
</style>

```





