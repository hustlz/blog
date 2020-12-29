# 模块化发展历程

在分析webpack模块化之前，我们先来简单过一下JS模块化的发展历程。

- IIFE
- CommonJS
- AMD
- CMD
- UMD
- ES6 Modules
- webpack模块化

## IIFE

模块化的一大作用就是用来隔离作用域，避免变量冲突。而Javascript没有语言层面的命名空间概念，只能将代码暴露到全局作用域下。因此最开始使用自执行函数（IIFE)来执行代码，从而避免变量名泄漏到全局作用域中

```js
(function(window) {
	// 这里写代码
})(window);
```

**缺点**：

虽然IIFE可以有效解决命名冲突的问题，但是对于依赖管理，还是束手无策。由于浏览器是从上至下执行脚本，因此为了维持脚本间的依赖关系，就必须手动维护好script标签的相对顺序。 

## CommonJS

CommonJS是一个同步加载模块规范。

允许模块通过require方法来**同步加载**所要依赖的其他模块，然后通过exports或module.exports来导出需要暴露的接口。

```js
// 导入
require("moduleName");
require("../app.js");

// 导出
exports.getStoreInfo = function() {...};
module.exports = someValue;
```

**缺点**：

此规范主要是应用于Node.js，不适用于浏览器端，因为同步意味着阻塞加载，浏览器资源是异步加载的。

## AMD

AMD提供了异步加载的功能，采用异步方式加载模块，模块的加载不影响后面语句的运行。所有依赖模块的语句，都定义在一个回调函数中，等到加载完成之后，回调函数才执行。

```js
// 定义模块
define("moduleName", ["dependency1", "dependency2"], function(d1, d2) {
	// 模块内部代码
});

// 加载模块
require(["moduleName", "app.js"], function(moduleName, app) {
	// 内部代码
});
```

- require函数检查依赖的模块，根据配置文件，获取js文件的实际路径
- 根据js文件实际路径，在dom中插入script节点，并绑定onload事件来获取该模块加载完成的通知。
- 依赖script全部加载完成后，调用回调函数

## CMD

CMD和AMD类似，提供了异步加载模块的功能，只是CMD加载完模块后不执行，只是下载而已，在所有依赖模块加载完成后进入主逻辑，遇到require语句的时候才执行对应的模块，这样模块的执行顺序和书写顺序是完全一致的。

```js
// 定义模块
define(function(require, exports, module) {
	let a = require("./a");
	a.doSomething();
	
	// 依赖就近书写，什么时候用到什么时候引入
	let b = require("./b");
	b.doSomething();
});
```

CMD和AMD的区别：

- 对于依赖的模块，AMD 是提前执行，CMD 是延迟执行。不过 RequireJS 从2.0开始，也改成了可以延迟执行（根据写法不同，处理方式不同）。CMD 推崇 as lazy as possible.

- AMD推崇依赖前置；CMD推崇依赖就近，只有在用到某个模块的时候再去require。

## UMD

UMD是AMD和CommonJS的结合，会进行判断支持Node.js的模块（exports）是否存在，存在则使用Node.js模块模式；否则继续判断是否支持AMD（define是否存在），存在则使用AMD方式加载模块，最后如果都不支持，则将模块挂载到window变量下。

```js
(function(window, factory) {
	if (typeOf exports === "object") {
		module.exports = factory();
	} else if (typeOf define === "function" && define.amd) {
        define(factory);
    } else {
        window.eventUtil = factory();
    }
})(this, function() {
	// 模块代码
});
```

## ES6 Modules

对于ES6 Modules来说，从语法层面就提供了模块化的功能。然而受限于浏览器的实现程度，如果想要在浏览器中运行，还是需要通过Babel等转译工具进行编译。

ES6提供了`import`和`export`命令，分别对应模块的导入和导出：

```
// demo.js
let name = "lz";
let sayHello = (name) => {
	console.log("HI, " + name);
}
export {name, sayHello};

// main.js
import { sayHello } from "./demo.js";
sayHello("lz");
```

- ES6使用的是基于文件的模块。所以必须一个文件一个模块，不能将多个模块合并到单个文件中去。
- ES6模块API是静态的，一旦导入模块后，无法再在程序运行过程中增添方法。
- ES6模块采用引用绑定（可以理解为指针)。这点和CommonJS中的值绑定不同，如果你的模块在运行过程中修改了导出的变量值，就会反映到使用模块的代码中去。所以，不推荐在模块中修改导出值，导出的变量应该是静态的。
- ES6模块采用的是单例模式，每次对同一个模块的导入其实都指向同一个实例。

## webpack 模块化方案

- **一切皆模块**：css,html.js，静态资源文件等都可以视作模块

- **按需加载**：进行代码分割，实现按需加载

Webpack实质上就是通过自执行函数启动，然后通过webpack自定义的exports和require实现模块化。

接下来我们来仔细看一下webpack是怎么实现模块化的。

# webpack模块化原理解析

## demo

首先我们先来写个简单的示例：

目录结构：

```
├── index.js // 项目入口
├── src                                     
│   ├── a.js // a模块
│   └── b.js // b模块
├── webpack.config.js // webpack编译配置文件
```

其中`index.js`源码如下所示：

```
// index.js
import('./src/a.js').then(a => {
    console.log('成功加载a.js文件');
    a.default();
})
```

`a.js`源码如下所示：

```
// a.js
import b from './b.js';
function a() {
    console.log('this is a.js'); 
    b();
}
export default a;
```

`b.js`源码如下所示：

```
// b.js
function b() {
    console.log('this is b.js')
}
export default b;
```

`webpack.config.js`配置文件如下所示：

```
// webpack.config.js
const path = require('path');

module.exports = {
  // 打包入口文件
  entry: './index.js',
  // 打包输出文件配置
  output: {
    // 输出文件路径
    path: path.resolve(__dirname, 'dist'),
    // 输出主文件的名称
    filename: 'main.js',
    // 输出模块的名称
    chunkFilename: '[name].bundle.js'
  }
};
```

通过webpack打包后，项目多出dist文件目录，并生成了以下三个文件。

```
├── index.js // 项目入口
├── dist
│   ├── 0.bundle.js // 动态加载模块的文件
│   ├── index.html // 页面主入口
│   └── mian.js // 页面主入口中引入的主js文件
├── src                                     
│   ├── a.js // a模块
│   └── b.js // b模块
├── webpack.config.js // webpack编译配置文件
```

接着我们查看打包后的源码，看看webpack到底是怎么实现模块化的。

## webpack编译产物解析

### index.html

首先我们从`index.html`入口文件开始看：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./main.js"></script>
</head>
<body>
    
</body>
</html>
```

编译之后的index.html会默认加载main.js运行。

### main.js

#### 代码组织形式

接着我们看启动文件（也就是第一个运行的js文件，这里是main.js）：

接着我们来整体看一下main.js文件的整体结构，从代码的整体接口来看，这个文件是一个IIFE(立即执行函数)，会以自执行的方式启动。

注：这里我先注释掉了函数内部的实现，先看整体。

```
(function(modules) { // webpack启动文件
	// 在window对象上定义一个webpackJsonp函数，此函数是一个回调函数，将会在js文件（chunk）加载成功之后执行
	var parentJsonpFunction = window["webpackJsonp"];
	window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) { ... };

	// module 缓存
	var installedModules = {};

	// 存储已经加载或者正在加载的chunk
	var installedChunks = {
		1: 0
	};

	// require函数用于加载模块，参数为模块的id
	function __webpack_require__(moduleId) { ... }

	// 由于本文件只包含入口的chunk，所以代码分割之后，其他的chunk都是由下面这个函数进行异步加载
	__webpack_require__.e = function requireEnsure(chunkId) { ... };

	// 暴露modules对象
	__webpack_require__.m = modules;

	// 暴露module缓存
	__webpack_require__.c = installedModules;

	// 提供Getter给导出的方法、变量
	__webpack_require__.d = function(exports, name, getter) { ... };

	// 用于与非协调模块兼容的getdefaultexport函数，将module.default或非module声明成getter函数的a属性上
	__webpack_require__.n = function(module) { ... };

	// Object.prototype.hasOwnProperty.call
	__webpack_require__.o = function(object, property) {...};

	// webpack公共路径
	__webpack_require__.p = "";

	// 异步加载的错误处理函数
	__webpack_require__.oe = function(err) { console.error(err); throw err; };

	// 加载入口的module，并且返回exports
	return __webpack_require__(__webpack_require__.s = 0);
})
([
/* 0 */
(function(module, exports, __webpack_require__) {
	...
})
]);
```

#### 关键变量和函数解析

详细解读代码之前，先解释一下其中几个核心的变量和函数：

- installedModules：此变量用于缓存已经require过的模块，用于多次require同一个模块时，直接返回exports

其中installedModules变量的结构如下所示：

![1609157251146](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1609157251146.png)

key为模块Id，value为模块对象。

模块对象有三个属性：

1. i表示模块ID
2. l:表示是否加载
3. exports表示模块内容

- installedChunks：此变量用于缓存已经加载或者正在加载的文件，主要针对于需要异步加载的文件

![1609157601092](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1609157601092.png)

其中key为chunkId，标识文件的id，value为加载文件操作的promise的resolve（成功回调），reject（失败回调）

- modules：此变量存储着所有被webpack包装加载过的模块函数。

![1609157732392](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1609157732392.png)

- `__webpack_require__(moduleId)`：require函数用于加载主文件（也就是mian.js）的模块，参数为模块的id

```
// require函数用于加载模块，参数为模块的id
	function __webpack_require__(moduleId) {
		// 如果所需加载模块存在缓存中，则直接从缓存取用
		if(installedModules[moduleId]) {
			return installedModules[moduleId].exports;
		}
		// 创建一个新的模块（并且将之push到缓存中）
		var module = installedModules[moduleId] = {
			i: moduleId,
			l: false,
			exports: {}
		};

		// 执行模块函数，注意这里做了一个动态绑定，将模块函数的调用对象绑定为module.exports，这是为了保证在模块中的this指向当前模块。
		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

		// 标记此模块为已加载
		module.l = true;

		// 返回模块的exports对象
		return module.exports;
}
```

![1609158036326](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1609158036326.png)

- `__webpack_require__.e (moduleId)`：此函数主要用于加载异步模块文件

```
// 由于本文件只包含入口的chunk，所以代码分割之后，其他的chunk都是由下面这个函数进行异步加载
	__webpack_require__.e = function requireEnsure(chunkId) {
		// 缓存查找
		var installedChunkData = installedChunks[chunkId];
		if(installedChunkData === 0) {
			return new Promise(function(resolve) { resolve(); });
		}

		// promise存在，意味着当前模块正在加载中
		if(installedChunkData) {
			return installedChunkData[2];
		}

		// 初始化缓存模块: [resolve，reject，promise]
		var promise = new Promise(function(resolve, reject) {
			installedChunkData = installedChunks[chunkId] = [resolve, reject];
		});
		installedChunkData[2] = promise;

		// 开始模块加载，创建script标签，append到head标签中，src指向加载的模块脚本资源，实现动态加载js脚本
		var head = document.getElementsByTagName('head')[0];
		var script = document.createElement('script');
		script.type = 'text/javascript';
		script.charset = 'utf-8';
		script.async = true;
		script.timeout = 120000;

       // HTMLElement 接口的 nonce 属性返回只使用一次的加密数字，被内容安全政策用来决定这次请求是否被允许处理。
		if (__webpack_require__.nc) {
			script.setAttribute("nonce", __webpack_require__.nc);
		}
       // 文件的路径由配置的publicPath、chunkid拼接而成
		script.src = __webpack_require__.p + "" + ({}[chunkId]||chunkId) + ".bundle.js";

		// 添加script标签定时器、onload、onerror 事件，如果超时或者模块加载失败，加载成功，都会调用onScriptComplete
		var timeout = setTimeout(onScriptComplete, 120000);
		script.onerror = script.onload = onScriptComplete;
		function onScriptComplete() {
			// 避免IE浏览器中内存泄露
			script.onerror = script.onload = null;
			clearTimeout(timeout);
			var chunk = installedChunks[chunkId];
			if(chunk !== 0) {
				if(chunk) {
					chunk[1](new Error('Loading chunk ' + chunkId + ' failed.'));
				}
				installedChunks[chunkId] = undefined;
			}
		};
		head.appendChild(script);

		return promise;
};
```



![1609158173342](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1609158173342.png)

- `webpackJsonpCallback(chunkIds, moreModules, executeModules)`：此函数是一个回调函数，将会在js文件（chunk）加载成功之后执行

```
var parentJsonpFunction = window["webpackJsonp"];
window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {
		// add "moreModules" to the modules object,
		// then flag all "chunkIds" as loaded and fire callback
		// 收集加载模块的resolve
		var moduleId, chunkId, i = 0, resolves = [], result;
		for(;i < chunkIds.length; i++) {
			chunkId = chunkIds[i];
			if(installedChunks[chunkId]) {
				resolves.push(installedChunks[chunkId][0]);
			}
			installedChunks[chunkId] = 0;
		}
       // 拷贝模块到modules
		for(moduleId in moreModules) {
			if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
				modules[moduleId] = moreModules[moduleId];
			}
		}

       // parentJsonpFunction作用：使异步加载的模块在多个不同的bundle内同步，多入口时，parentJsonpFunction执行的是上一个bundle的webpackJsonp函数
		if(parentJsonpFunction) parentJsonpFunction(chunkIds, moreModules, executeModules);
		// 直接调用resolve，完成整个异步加载
       while(resolves.length) {
			resolves.shift()();
		}

};
```

![1609158363915](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1609158363915.png)

### 整体流程

了解了几个关键的变量和函数之后，我们接着把webpack的整体流程捋一遍。

1. webpack主文件函数，都是在进行变量和函数的声明，只有最后使用`return __webpack_require__(__webpack_require__.s = 0);`返回了一个moduleId为0的模块对象；

2. 由于`__webpack_require__`函数会从modules参数中获取模块ID为0的模块，从IIFE中可看到，初始化的modules是一个数组，moduleId也对应着数组值的索引，也就是可以从modules获取第一个模块，并使用call将第一个模块的运行绑定到当前模块中，保证其内部this指向正确，并将变量`installedModules`对应moduleId的key的值设置已加载；

   ```js
   /* 0 */
   (function(module, exports, __webpack_require__) {
       __webpack_require__.e/* import() */(0).then(__webpack_require__.bind(null, 1)).then(a => {
           console.log('成功加载a.js文件');
           a.default();
       })
   })
   ```

3. 接着我们看第一个模块中的代码和我们`index.js`的代码一致，在运行代码之前，依赖a模块，由于a模块被打包为动态加载，因为需要使用`__webpack_require__.e`加载chunkId为0的异步文件，异步加载文件成功后，会根据`installedChunks`中对应chunkId对应key的值是否已加载；

   ```js
   // index.js
   import('./src/a.js').then(a => {
       console.log('成功加载a.js文件');
       a.default();
   })
   ```

4. 上一步在chunkId为0的异步文件加载完成之后，会调用`webpackJsonp`函数，在函数中，会设置`installedChunks`变量中对应chunkId的key对应的值为0(表示加载成功)，并将加载成功的模块拷贝到`modules`变量中，这样后续可以通过`__webpack_require__`获取加载后的模块，以及收集`installedChunks`中正在加载的chunkId的resolve，并一一调用，这样才会进入在后续的then回调中；

   ```js
   // 0.bundle.js
   webpackJsonp([0],[
   /* 0 */,
   /* 1 */
   /***/ (function(module, __webpack_exports__, __webpack_require__) {
   
   "use strict";
   Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
   /* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__b_js__ = __webpack_require__(2);
   
   function a() {
       console.log('this is a.js'); 
       __WEBPACK_IMPORTED_MODULE_0__b_js__["a" /* default */]();
   }
   /* harmony default export */ __webpack_exports__["default"] = (a);
   
   /***/ }),
   /* 2 */
   /***/ (function(module, __webpack_exports__, __webpack_require__) {
   
   "use strict";
   function b() {
       console.log('this is b.js')
   }
   /* harmony default export */ __webpack_exports__["a"] = (b);
   
   /***/ })
   ]);
   ```

5. 异步文件加载完成后，then回调中通过`__webpack_require__.bind(null, 1)`获取moduleId为1的a模块，从上述bundle文件中可以看出，和a.js以及b.js的代码是一致的，这里就不贴代码比较了；

6. 在a模块的代码中，设置了自己的`__esModule`属性为true，这个属性主要是为了防止出现非ES6模块，和ES6模块混合时，能够根据这个属性能够正确获取到模块导出所使用的；

7. 由于b模块的代码和a模块的代码打包在同一个chunk文件里面，因此会一直被加载到`modules`中去，所以这里直接通过`__webpack_require__`加载b模块代码即可。

## 总结

**一句话概括，webpack通过自执行函数启动，然后通过webpack自定义的exports和require实现模块化。**

