[TOC]

## `transpileDependencies`属性不生效

**问题描述**

由于在项目中引入了`node-rsa`模块，发现在编译产物中包含`const`关键字，导致项目在IE10无法运行。

**问题原因**

默认情况下 `babel-loader` 会忽略所有 `node_modules` 中的文件。

**解决过程**

根据[vue cli官网](https://cli.vuejs.org/zh/config/#transpiledependencies)：

> 默认情况下 `babel-loader` 会忽略所有 `node_modules` 中的文件。如果你想要通过 Babel 显式转译一个依赖，可以在这个选项中列出来。

因此通过修改`vue.config.js`文件：

```
module.exports = {
    transpileDependencies: ['node-rsa']
    ...
};
```

发现编译产物中，依然存在`const`关键字。

我以为是路径的问题，可能路径写的不对，通过在网上查找，`transpileDependencies`写法千奇百怪。

```
const path = require('path');
function resolve(dir) {
    return path.join(__dirname, dir);
}

module.exports = {
    // transpileDependencies: ['./node_modules/node-rsa']
    // transpileDependencies: ['node_modules/node-rsa']
    // transpileDependencies: ['node-rsa']
    transpileDependencies: [resolve('node_modules/node-rsa')]
    ...
};
```

以上四种写法全是试了一遍，还是没有效果。。。

最后通过查看[vue-cli issues#1552](https://github.com/vuejs/vue-cli/issues/1552)，发现`transpileDependencies` 必须搭配`babel.config.js`文件使用才能生效（这个在文档里面是没有强调说明的，只能靠自己踩坑发现）。

在修改`.babelrc`为`babel.config.js`后，发现还是没有生效。。。

```
// babel.config.js
module.export = {
  "presets": [
    "@vue/babel-preset-app"
  ]
}
```

最后通过查看[vue cli issues#1568](https://github.com/vuejs/vue-cli/issues/1568)，发现`transpileDependencies `需要在`babel.config.js`中设置`process.env.VUE_CLI_BABEL_TRANSPILE_MODULES = true`，才能生效。

经过实践，发现依然无法生效，依然是[vue cli issues#1568](https://github.com/vuejs/vue-cli/issues/1568)，vue作者本人提到：

> Yeah, a warning block under `transpileDependencies` and in the polyfills section would help. Essentially, `transpileDependencies` would only work for deps with ES module formats.

**也就是说`transpileDependencies`只能对ES模块代码生效。**

**最终解决方案**

通过一番尝试babel转译node_modules下的代码无果后，想到另外一个解决方案：`patch-package`。

1. `npm install --save-dev patch-package postinstall-postinstall `，安装模块依赖

2. 通过修改`node-rsa`中源码，将`const`关键字通通修改为`var`关键字

3. 然后通过`yarn patch-package node-rsa`生成patches文件

   ```
   diff --git a/node_modules/node-rsa/src/formats/pkcs1.js b/node_modules/node-rsa/src/formats/pkcs1.js
   index 5fba246..b32db6a 100644
   --- a/node_modules/node-rsa/src/formats/pkcs1.js
   +++ b/node_modules/node-rsa/src/formats/pkcs1.js
   @@ -2,11 +2,11 @@ var ber = require('asn1').Ber;
    var _ = require('../utils')._;
    var utils = require('../utils');
    
   -const PRIVATE_OPENING_BOUNDARY = '-----BEGIN RSA PRIVATE KEY-----';
   -const PRIVATE_CLOSING_BOUNDARY = '-----END RSA PRIVATE KEY-----';
   +var PRIVATE_OPENING_BOUNDARY = '-----BEGIN RSA PRIVATE KEY-----';
   +var PRIVATE_CLOSING_BOUNDARY = '-----END RSA PRIVATE KEY-----';
    
   -const PUBLIC_OPENING_BOUNDARY = '-----BEGIN RSA PUBLIC KEY-----';
   -const PUBLIC_CLOSING_BOUNDARY = '-----END RSA PUBLIC KEY-----';
   +var PUBLIC_OPENING_BOUNDARY = '-----BEGIN RSA PUBLIC KEY-----';
   +var PUBLIC_CLOSING_BOUNDARY = '-----END RSA PUBLIC KEY-----';
    
    module.exports = {
        privateExport: function (key, options) {
   diff --git a/node_modules/node-rsa/src/formats/pkcs8.js b/node_modules/node-rsa/src/formats/pkcs8.js
   index 3dd1a3c..5f88081 100644
   --- a/node_modules/node-rsa/src/formats/pkcs8.js
   +++ b/node_modules/node-rsa/src/formats/pkcs8.js
   @@ -3,11 +3,11 @@ var _ = require('../utils')._;
    var PUBLIC_RSA_OID = '1.2.840.113549.1.1.1';
    var utils = require('../utils');
    
   -const PRIVATE_OPENING_BOUNDARY = '-----BEGIN PRIVATE KEY-----';
   -const PRIVATE_CLOSING_BOUNDARY = '-----END PRIVATE KEY-----';
   +var PRIVATE_OPENING_BOUNDARY = '-----BEGIN PRIVATE KEY-----';
   +var PRIVATE_CLOSING_BOUNDARY = '-----END PRIVATE KEY-----';
    
   -const PUBLIC_OPENING_BOUNDARY = '-----BEGIN PUBLIC KEY-----';
   -const PUBLIC_CLOSING_BOUNDARY = '-----END PUBLIC KEY-----';
   +var PUBLIC_OPENING_BOUNDARY = '-----BEGIN PUBLIC KEY-----';
   +var PUBLIC_CLOSING_BOUNDARY = '-----END PUBLIC KEY-----';
    
    module.exports = {
        privateExport: function (key, options) {
   ```

4. 在`package.json`的script脚本中添加`"postinstall": "patch-package"`后续所有下载代码，运行`npm install`后，都会运行`patch-package`将上一步生成的patches文件打补丁到`node_modules`中去

问题解决。

## TypeError: this.getOptions is not a function at runMicrotasks

**问题描述**

项目编译的时候碰到错误如下所示：

```
 error  in ./src/components/uni-forms-item/uni-forms-item.vue?vue&type=style&index=0&id=39373d84&lang=scss&scoped=true&

TypeError: this.getOptions is not a function
    at runMicrotasks (<anonymous>)


 @ ./src/components/uni-forms-item/uni-forms-item.vue?vue&type=style&index=0&id=39373d84&lang=scss&scoped=true& 1:0-872 1:888-891 1:893-1762 1:893-1762
 @ ./src/components/uni-forms-item/uni-forms-item.vue
 @ ./node_modules/@dcloudio/vue-cli-plugin-uni/packages/vue-loader/lib/loaders/templateLoader.js??vue-loader-options!./node_modules/@dcloudio/vue-cli-plugin-uni/packages/webpack-preprocess-loader??ref--14-0!./node_modules/@dcloudio/webpack-uni-mp-loader/lib/template.js!./node_modules/@dcloudio/vue-cli-plugin-uni/packages/webpack-uni-app-loader/page-meta.js!./node_modules/@dcloudio/vue-cli-plugin-uni/packages/vue-loader/lib??vue-loader-options!./node_modules/@dcloudio/webpack-uni-mp-loader/lib/style.js!./src/pages/extUI/forms/forms.nvue?vue&type=template&id=4eeabe92&
 @ ./src/pages/extUI/forms/forms.nvue?vue&type=template&id=4eeabe92&
 @ ./src/pages/extUI/forms/forms.nvue
 @ ./src/main.js?{"page":"pages%2FextUI%2Fforms%2Fforms"}
```

**问题原因**

```
    "sass-loader": "^11.1.0",
```

上网搜索，发现 `sass-loader@11.1.0` 版本需要 `webpack@5.0.0` ，而 `@vue/cli@4.5.0` 所用的是 `webpack@4`，所以需要将 `sass-loader`的版本降到11以下

**解决方案**

第一步： `yarn remove sass-loader`

第二步：`yarn add sass-loader@10.1.1 --dev`

