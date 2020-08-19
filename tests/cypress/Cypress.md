| 版本      | 起止日期  | 责任人 | 备注     |
| :-------- | --------- | ------ | -------- |
| V0.1/草稿 | 2020/7/10 | 李政   | 创建文档 |

[TOC]

# 安装

## npm

```
npm install cypress --save-dev
```

## yarn

```
yarn add cypress --dev
```

## 网址下载

直接从[CDN下载](http://download.cypress.io/desktop)，会自动识别你的平台，并直接下载最新的可用版本。

# 使用

## 打开cypress

### npm安装

在项目根目录中打开：

```
./node_modules/.bin/cypress open
```

或者使用`npm bin`

```
$(npm bin)/cypress open
```

或者使用`yarn`

```
yarn run cypress open
```

## 切换浏览器

Cypress测试执行器会尝试在用户的机器上找到所有兼容的浏览器。你可以在程序的右上角下拉列表选择不同的浏览器。

## 添加npm脚本

如果使用npm安装的cypress，可以在package.json中添加script脚本，然后使用npm run 调用此命令。

```
{
  "scripts": {
    "cypress:open": "cypress open"
  }
}
```

# 目录结构

新建项目后，cypress会自动创建标准文件夹结构：





# 语法

## 查询元素

### 选择器查询

cypress查询元素和jquery一样，因为cypress捆绑了jquery并公开了其DOM遍历方法，我们可以很熟悉的使用jquery的api来查询元素。

```
// 每个方法都等同于它的jQuery对应方法。用你所知道的！
cy.get('#main-content')
  .find('.article')
  .children('img[src^="/static"]')
  .first()
```

Cypress利用jQuery强大的选择器引擎帮助现代Web开发人员熟悉和查找元素。

但是和jquery不一样的地方是：cypress返回DOM元素是异步的，而jquery是同步的。

```
// 很好，jQuery同步返回元素。
const $jqElement = $('.element')

// 不行！Cypress没有同步返回元素。
const $cyElement = cy.get('.element')
```

cypress如果无法从选择器找到到匹配的DOM时，cypress会自动启用重查机制，知道元素被找到；或者达到超时设置。

```
cy
  // cy.get() 查找'#element'元素,重复查询,直到...
  .get('#element')

  // ...它找到元素!
  // 您现在也可以通过使用.then方式使用它
  .then(($myElement) => {
    doSomething($myElement)
  })
  
  cy
  // cy.get() 查找'#my-nonexistent-selector'元素,重复查询直到...
  // ...达到超时它还没有找到元素.
  // Cypress 停止并且标记测试失败.
  .get('#element-does-not-exist')

  // ...这段代码不会运行...
  .then(($myElement) => {
    doSomething($myElement)
  })
```

在Cypress,当你想直接与DOM元素进行交互时，可调用回调函数`.then()`并接收元素作为其第一个参数进行使用。当你想完全跳出重试和超时功能，使用传统的同步方法时，请使用`Cypress.$`。

### 文本内容查询

有一种更方便的方式就是通过文本内容进行查找。`cy.contains()`命令，比如：

```
// 在文档里查找文本为'New Post'的元素
cy.contains('New Post')

// 查找'.main'的元素且文本内容为'New Post'
cy.get('.main').contains('New Post')
```

## 命令链接机制

## 命令自定义

Cypress可以对重写已有的命令，或者新增命令。需要在`cypress/support/commands.js`文件中进行定义，并在`cypress/support/index.js`中引入。

每一个测试文件运行之前，Cypress都会自动加载`cypress/support/index.js`。

### 语法

```js
Cypress.Commands.add(name, callbackFn)
Cypress.Commands.add(name, options, callbackFn)
Cypress.Commands.overwrite(name, callbackFn)
```

# 碰到的问题

## 无法进行设备添加

### 问题描述

在模拟用户进行设备发现，并添加设备的时候，发现测试用例报错，无法运行。

### 问题原因

经过跟踪发现，是因为无法获取`AUTH-KEY`导致添加设备接口无法进行RSA加密，从而报错。

### 解决方案

1. 本地开发环境中进行调试，添加debugger以及日志打印，发现设备添加接口无法正常发送和接收
2. 后来发现是因为设备添加接口带有用户密码，因此接口会被加密，但是本地开发没有auth-key，导致加密过程中报错，无法发送此接口
3. 针对第二点，手动从RMS启动器中的网页拷贝了一个auth-key设置在本地开发环境中，此时添加设备接口可以成功发送出去
4. 但是，接口发送出去却不能被正确响应，因为webpack配置的devServer环境中，会被接口的body进行解析，而加密接口的body是一串加密后的随机字符串，导致body-parse解析报错，从而响应400（bad request）
5. 因此express中post请求的body是不能直接读取的，需要经过第三方中间件body-parse解析后，才能读取，经过查看devServer中的配置，发现读取body的需求，只有nms系统的api接口才有，因此RMS系统库中将此代码注释掉即可
6. 现在可以正确返回加密接口请求了，但是加密接口响应是普通的json格式，而不是所期待的加密格式，所以axios还是无法正确解析到响应
7. 经过和SMB同事彭尔登沟通前端RSA加密方案，以及和付工沟通后台RSA加密方案，并获得一对公钥和私钥的字符串，用于本地开发模拟加密解密接口
8. 查询node-rsa的github查看api使用原理以及使用方式，完成rms系统添加rsa本地开发加密解密接口的调试

**子系统本地开发添加rsa步骤**

- yarn add node-rda -D
- dev/main.js中添加设置localStorage中AUTH-KEY的值为RSA的公钥
- 在devServer的钩子函数中，对需要加密的接口文件使用RSA私钥进行加密

```
const keyDataPri = `-----BEGIN PRIVATE KEY----- ${authKeyPri} -----END PRIVATE KEY-----`;
keyPri.importKey(keyDataPri, 'pkcs8-private');
let result = keyPri.encryptPrivate(data, 'base64', 'utf8');
res.send((result));
```

经过本地测试方案可行。

## localstorage被清空无法获取

### 问题描述

使用cypress编写测试用例进行设备发现以及添加时，发现无法进行设备添加。

### 问题原因

经过跟踪发现，是因为无法获取`AUTH-KEY`导致添加设备接口无法进行RSA加密，从而报错。

### 解决方案

在调试的时候发现，`main.js`中是有设置`AUTH-KEY`的`localStorage`的，但是不知道为什么Cypress运行的时候无法获取，因此猜测Cypress在某个步骤清除了所有的`localStorage`。

经过查看Cypress在github上的issue，发现存在同样的问题：

> https://github.com/cypress-io/cypress/issues/8037

在其中找到了答案：Cypress在每一个测试用例之间都会自动清空所有的`localStorage`、`sessionStorage`、以及`cookies`。

> Cypress automatically clears `localStorage`, `sessionStorage`, and `cookies` in between each test (every `it` function). 

因此我们需要在每个测试用例之前将localStorage恢复，因此需要使用`beforeEach`钩子函数，在其中设置`localStorage`即可。

```js
beforeEach(() => {
    cy.setLocalStorageCache()
})
```

其中`setLocalStorageCache`在support中定义了一个函数：

```js
Cypress.Commands.add("setLocalStorageCache", () => {
    localStorage.setItem('AUTH-KEY', 'MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCOVhYuiGmXwAZ4Jpeh5qy3sdO8zF/dmX47vv/F2P8oZ33moNIAWE7OGnbzZWP3qJBMFDbaBCuPgItZf1WIdkEkN1fAA0Rl4D2BQedl/dOtmw+A3svUBJBVZRXOS3UEs6TMxxroTDHqCN3bu1VO6NTEg8xzOKvCFqxY2YbZRiM45QIDAQAB');
})
```

经过测试方案可行。