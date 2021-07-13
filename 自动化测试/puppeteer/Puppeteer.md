

| 版本      | 起止日期  | 责任人 | 备注     |
| :-------- | --------- | ------ | -------- |
| V0.1/草稿 | 2020/8/13 | 李政   | 创建文档 |

[TOC]

# 安装

## puppeteer

在项目中使用`puppeteer`需要安装：

```js
yarn add puppeteer
// npm install puppeteer -D
```

但是安装`puppeteer`的时候会下载最新版本的`Chromium`，保证可以使用API。

如果想要跳过下载，需要配置环境变量`PUPPETEER_SKIP_CHROMIUM_DOWNLOAD`

```js
npm config set PUPPETEER_SKIP_CHROMIUM_DOWNLOAD 1
```

## puppeteer-core

`puppeteer-core`是一个库用来帮助驱动任何支持`DevTools`协议的东西。在安装时不会下载`Chromium`，而且会忽略所有`PUPPETEER_*`env变量。

```js
yarn add puppeteer-core
// npm install puppeteer-core -D
```

# 使用

## puppeteer

通过require引入`puppeteer`，然后调用`launch`打开浏览器。

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');
  await page.screenshot({path: 'example.png'});

  await browser.close();
})();
```

## puppeteer-core

`puppeteer-core`的使用和`puppeteer`区别不大，引入的时候修改如下即可;

```
const puppeteer = require('puppeteer-core');
```

## 指定chrome运行路径

由于之前下载跳过了`chromium`的下载，因此运行浏览器，需要手动指定浏览器的执行路径。

### PUPPETEER_EXECUTABLE_PATH

通过`PUPPETEER_EXECUTABLE_PATH`环境变量指定一个chrome的可执行路径，会被用于`puppeteer.launch`。

### executablePath

通过调用`puppeteer.launch`时传入options：`executablePath` ，并指定chrome的可执行路径即可。

## 封装

直接封装成一个ES6模块即可。比如：

点击操作

```js

const click = async function (page, sel, num = 0) {
  await page.waitFor(sel)
  await page.evaluate((sel, num) => {
    document.querySelectorAll(sel)[num].click()
  }, sel, num)
};
```

输入操作（最大的坑是如果原有输入框有内容，如何把他替换掉）

```js
const type = async function (page, sel, value, num = 0) {
  const edit = async () => {
    const elementHandle = (await page.$$(sel))[num]
    elementHandle.click({clickCount: 3})
    await elementHandle.press('Backspace')
    await elementHandle.type(value, {delay: 20})
    await page.waitFor(1000)
 
    return await page.evaluate((sel, num) => {
      return document.querySelectorAll(sel)[num].value
    }, sel, num)
  }
  let cname_value
  await page.waitFor(sel)
  let i = 0
  do {
    cname_value = await edit()
  } while (cname_value !== value && i++ < 20)
}
```

页面崩溃捕获打印

```js

    page.on('pageerror', function (err) {
      if (err.message.indexOf('BJ_REPORT is not defined') >= 0
        || err.message.indexOf('template is not defined') >= 0
        || err.message.indexOf('$ is not defined') >= 0
        || err.message.indexOf('request was interrupted by a call to pause') >= 0
        || err.message.indexOf('seajs is not defined')
      ) {
        return
      }
      throw err
    })
    page.on('error', err => {
      throw err
    })
    page.on('console', msg => {
      for (let i = 0; i < msg.args.length; ++i) {
        console.log(`${i}: ${msg.args[i]}\`)
      }
    })
```

# 遇到的问题

## yarn运行报错Found incompatible module.

### 问题描述

使用yarn安装puppeteer时，命令行报错：

```js
error puppeteer@5.2.1: The engine "node" is incompatible with this module. Expected version ">=10.18.1". Got "10.15.0"
error Found incompatible module.
```

### 问题原因

node版本过低导致

 ### 解决方案

```js
yarn install --ignore-engines
```

## before is not define

### 问题描述

在`puppeteer`中使用`beforeAll`钩子函数 ，运行的时候报错：

```js
ReferenceError: beforeAll is not defined
```

### 问题原因

`puppeteer`本身不具有测试功能（断言、生成测试报告），只能进行浏览器的自动化，需要配合`Jest`一起使用才行。

### 解决方案

安装`jest`配合使用即可。

## 默认打开的浏览器窗口大小只有屏幕一半

### 问题描述

使用`puppeteer.launch`打开默认浏览器窗口，发现浏览器窗口只有屏幕一半

```js
puppeteer.launch({
    executablePath: 'C:/Program Files (x86)/Google/Chrome/Application/chrome.exe',
    headless: false,
    defaultViewport: {
        width: 1920,
        height: 937
    }
});
```

而且使用`  await page.setViewport({width: 1920, height: 937});`也没有效果。

### 问题原因

未知

### 解决方案

在`puppeteer.launch`中添加args参数设置窗口大小。

```js
puppeteer.launch({
    executablePath: 'C:/Program Files (x86)/Google/Chrome/Application/chrome.exe',
    headless: false,
    defaultViewport: {
        width: 1920,
        height: 937
    },
    args: [`--window-size=${1920},${937}`]
});
```

## puppeteer查询元素需要添加延迟

### 问题描述

在cypress中点击添加按钮后，判断弹窗的标题内容，可以直接判断正确，而puppeteer中判断弹窗的内容发现不对，总是显示“设置密码”而不是“添加设备”。

### 问题原因

Cypress使用get方法获取弹窗标题的时候，只会获取显示的DOM内容，而puppeteer使用`$eval`查询元素会获取到隐藏的元素，而弹窗未显示的时候，弹窗标题初始化为“设置密码”。

### 解决方案

添加延迟解决

`waitForSelector`方法可以设置option获取显示的元素，但是这个方法不支持

