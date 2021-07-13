| 版本      | 起止日期  | 责任人 | 备注     |
| :-------- | --------- | ------ | -------- |
| V0.1/草稿 | 2020/8/17 | 李政   | 创建文档 |

[TOC]

# 安装

**第一步：需要安装nightwatch**

```js
npm install nightwatch -D
// yarn add nightwatch
```

**第二步：需要下载selenium**

> selenium是一个java应用，因此需要依赖于JDK，版本至少要大于等于7。如果你不满足这个条件，你可以点击 [这里](https://www.oracle.com/java/technologies/javase-downloads.html)。

```js
npm install selenium-server -D
```

除此之外，可以在[selenium downloads page](https://selenium-release.storage.googleapis.com/index.html)下载最近的稳定和alpha版本。

下载`selenium-server-standalone-{VERSION}.jar`文件并放置在项目中。比较好的方式是创建一个单独的文件夹放置（比如`bin`），并且和其他浏览器驱动放置一起。

**第三步：安装webdriver**

想要安装的webDriver取决于目标浏览器：

| WebDriver Binary                                             |                           Browser                            | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------: | :----------------------------------------------------------- |
| [GeckoDriver](https://nightwatchjs.org/gettingstarted/installation/#install-geckodriver) | ![Mozilla Firefox](https://nightwatchjs.org/img/logos/Firefox_Logo_2017.png) | Standalone server which implements the [W3C WebDriver protocol](https://w3c.github.io/webdriver/#protocol) to communicate with Gecko browsers, such as Firefox. |
| [ChromeDriver](https://nightwatchjs.org/gettingstarted/installation/#install-chromedriver) | ![Google Chrome](https://nightwatchjs.org/img/logos/1200px-Google_Chrome_icon.svg.png) | Standalone server which implements the [JSON Wire Protocol](https://github.com/SeleniumHQ/selenium/wiki/JsonWireProtocol) for Chromium, however it is currently in the process of transitioning to the W3C WebDriver spec.  Available for Chrome on Android and Chrome on Desktop (Mac, Linux, Windows and ChromeOS). |
| [Microsoft WebDriver](https://nightwatchjs.org/gettingstarted/installation/#install-microsoftedge) | ![Microsoft Edge](https://nightwatchjs.org/img/logos/Microsoft_Edge_logo.svg.png) | Windows executable which supports both the W3C WebDriver spec and JSON Wire Protocol for running tests against Microsoft Edge. |
| [SafariDriver](https://nightwatchjs.org/gettingstarted/installation/#install-safaridriver) | ![Microsoft Edge](https://nightwatchjs.org/img/logos/safari_icon_large_2x.png) | The `/usr/bin/safaridriver` binary comes pre-installed with recent versions of Mac OS and it's available to use following the instructions on [Apple Developer website](https://developer.apple.com/documentation/webkit/testing_with_webdriver_in_safari).  More information is available on [About WebDriver for Safari](https://developer.apple.com/documentation/webkit/about_webdriver_for_safari) page. |

**GeckoDriver：**

```js
npm install geckodriver -D
```

除了使用npm下载之外，也可以从github上release下载（https://github.com/mozilla/geckodriver/releases）

**ChromeDriver：**

```js
npm install chromedriver -D
```

也可以从[chromeDriver downloads page](https://chromedriver.chromium.org/downloads)下载。

**Microsoft WebDriver：**

```bat
DISM.exe /Online /Add-Capability /CapabilityName:Microsoft.WebDriver~~~~0.0.1.0
```

更多关于安装和使用的说明见[官网](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/)

**SafariDriver:**

safariDriver在mac电脑上已经被安装过了，但是需要一些手动配置：

```bat
safaridriver --enable
```

# 使用

## 配置文件讲解

`nightwatch`的配置文件分为：基本配置、selenium配置和测试配置三个部分。

```js
{
    // 基础配置
    // 测试用例源文件路径
	"src_folders" : ["tests"], 
    // 测试报告输出路径
    "output_folder" : "reports",
    // 自定义扩展命令路径
    "custom_commands_path" : "",
    // 自定义扩展断言路径
    "custom_assertions_path" : "",
    // 页面对象路径，主要抽象页面操作的片段（sections），以及想要选择的元素（elements）
    "page_objects_path" : "",
    // 全局变量所在文件夹，可以通过browser.global.xx来获取
    "globals_path" : "",

    // selenium配置
    "selenium" : {
        // 是否开启selenium
        "start_process" : true,
        // selenium服务所在地址，一般是个jar包
        "server_path" : require('selenium-server').path,
        "log_path" : "",
        // 端口
        "port" : 4444,
        // 指定要运行的webdriver
        "cli_args" : {
            // 谷歌浏览器的drvier地址，在windows下是个exe文件
          	'webdriver.chrome.driver': require('chromedriver').path
        }
     },
         
     // 测试配置，制定测试时各个环境的设置，默认是default，通过--env加环境名指定配置的任意环境
     "test_settings": {
        "default": {
            "selenium_port": 5555,
            "selenium_host": 'localhost',
            "silent": true,
            "globals": {
            	"devServerURL": 'http://localhost:' + (process.env.PORT || 1111)
            }
        },

        "chrome": {
            "desiredCapabilities": {
                "browserName": 'chrome',
                "javascriptEnabled": true,
                "acceptSslCerts": true,
                "chromeOptions": {
                    "w3c": false 
                }
            }
        },

        "firefox": {
            "desiredCapabilities": {
                "browserName": 'firefox',
                "javascriptEnabled": true,
                "acceptSslCerts": true
            }
        }
    }
}
```

## API





# 碰到的问题

## 安装chromedriver失败

### 问题描述

使用npm安装chromedriver时报错

```js
npm install nightwatch chromedriver selenium-server -D
```

```js
53756 warn optional SKIPPING OPTIONAL DEPENDENCY: fsevents@2.1.3 (node_modules\watchpack\node_modules\fsevents):
53757 warn notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.1.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})
53758 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Valid OS:    darwin
53758 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Valid Arch:  any
53758 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Actual OS:   win32
53758 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Actual Arch: x64
53759 warn optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.13 (node_modules\fsevents):
53760 warn notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.13: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})
53761 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Valid OS:    darwin
53761 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Valid Arch:  any
53761 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Actual OS:   win32
53761 verbose notsup SKIPPING OPTIONAL DEPENDENCY: Actual Arch: x64
53762 verbose stack Error: chromedriver@84.0.1 install: `node install.js`
53762 verbose stack Exit status 1
53762 verbose stack     at EventEmitter.<anonymous> (C:\Program Files\nodejs\node_modules\npm\node_modules\npm-lifecycle\index.js:301:16)
53762 verbose stack     at EventEmitter.emit (events.js:182:13)
53762 verbose stack     at ChildProcess.<anonymous> (C:\Program Files\nodejs\node_modules\npm\node_modules\npm-lifecycle\lib\spawn.js:55:14)
53762 verbose stack     at ChildProcess.emit (events.js:182:13)
53762 verbose stack     at maybeClose (internal/child_process.js:962:16)
53762 verbose stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:251:5)
53763 verbose pkgid chromedriver@84.0.1
53764 verbose cwd G:\RMS\TUMS-git-commit\webui\rms
53765 verbose Windows_NT 10.0.18363
53766 verbose argv "C:\\Program Files\\nodejs\\node.exe" "C:\\Program Files\\nodejs\\node_modules\\npm\\bin\\npm-cli.js" "install" "nightwatch" "chromedriver" "selenium-server" "-D"
53767 verbose node v10.15.0
53768 verbose npm  v6.4.1
53769 error code ELIFECYCLE
53770 error errno 1
53771 error chromedriver@84.0.1 install: `node install.js`
53771 error Exit status 1
53772 error Failed at the chromedriver@84.0.1 install script.
53772 error This is probably not a problem with npm. There is likely additional logging output above.
53773 verbose exit [ 1, true ]
```

### 问题原因

环境和包版本不匹配。

### 解决方案

表示npm将不会运行在package.json中指定的scripts脚本

```
npm install chromedriver -D --ignore-scripts
```

 ## Unable to create new service: GeckoDriverService

### 问题描述

运行` ./node_modules/.bin/nightwatch  nightwatch/deviceDiscover.spec.js`报错如下：

```js
 × deviceDiscover.spec
   An error occurred while retrieving a new session: "Unable to create new service: GeckoDriverService"
Error: An error occurred while retrieving a new session: "Unable to create new service: GeckoDriverService"
    at endReadableNT (_stream_readable.js:1094:12)

  Error: An error occurred while retrieving a new session: "Unable to create new service: GeckoDriverService"
       at endReadableNT (_stream_readable.js:1094:12)

   SKIPPED:
   - search nightwatch on baidu
```

### 问题原因

这是因为我们没有指定`env`,nightwatch会自动的找`default`的`env`，接着它会找默认的driver，是firefox的，我们前面只安装了chrome-driver，所以这时候肯定是会报错的。

### 解决方案

在运行的时候添加一个参数指定chrome即可。

```js
./node_modules/.bin/nightwatch examples/01-hello-nightwatch.js --env chrome
```

## Unable to create new service: ChromeDriverService

### 问题描述

运行nightwatch进行测试报错如下：

```js
[Device Discover Spec] Test Suite
=================================
- Connecting to 127.0.0.1 on port 5555...
   Response 500 POST /wd/hub/session (70ms)
   { value:
      { error:
         [ 'Build info: version: \'3.141.59\', revision: \'e82be7d358\', time: \'2018-11-14T08:25:53\'',
           'System info: host: \'W9379\', ip: \'192.168.1.101\', os.name: \'Windows 10\', os.arch: \'amd64\', os.version: \'10.0\', java.version: \'1.8.0_181\'',
           'Driver info: driver.version: unknown' ],
        message: 'Unable to create new service: ChromeDriverService' },
‼ Error connecting to 127.0.0.1 on port 5555.
_________________________________________________

TEST FAILURE: 1 error during execution; 0 tests failed, 0 passed (908ms)

 × deviceDiscover.spec
   An error occurred while retrieving a new session: "Unable to create new service: ChromeDriverService"
Error: An error occurred while retrieving a new session: "Unable to create new service: ChromeDriverService"
    at endReadableNT (_stream_readable.js:1094:12)

  Error: An error occurred while retrieving a new session: "Unable to create new service: ChromeDriverService"
       at endReadableNT (_stream_readable.js:1094:12)

   SKIPPED:
   - search nightwatch on baidu

```

### 问题原因

最开始猜测是电脑没有安装jdk环境，导致selenium无法运行。

但是后续安装了java环境后，还是存在一样的问题。最后发现是chromedriver在下载的时候没有完全，由于跳过了script脚本的运行，导致没有下载

### 解决方案

手机开启USB共享网络下载chromedriver，成功下载，并运行nightwatch成功。

## TypeError [ERR_UNESCAPED_CHARACTERS]

### 问题描述

在运行nightwatch测试用例的时候，报错如下：

```js
 Error while running .isElementDisplayed() protocol action: TypeError [ERR_UNESCAPED_CHARACTERS]: Error while trying to create HTTP request for "/wd/hub/session/5001c8b007da1e5c7aad584d65ed5b24/element/[object Object]/displayed": Request path contains unescaped characters
    at new ClientRequest (_http_client.js:115:13)
    at Object.request (http.js:42:10)
    at HttpRequest.createHttpRequest (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\http\request.js:138:55)
    at HttpRequest.send (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\http\request.js:217:29)
    at Promise (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\transport.js:193:15)
    at new Promise (<anonymous>)
    at Selenium2Protocol.sendProtocolAction (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\transport.js:191:12)
    at Selenium2Protocol.runProtocolAction (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\jsonwire.js:61:17)
    at Object.isElementDisplayed (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\actions.js:54:10)
    at Selenium2Protocol.executeProtocolAction (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\transport.js:239:48)

 Error while running .isElementDisplayed() protocol action: TypeError [ERR_UNESCAPED_CHARACTERS]: Error while trying to create HTTP request for "/wd/hub/session/5001c8b007da1e5c7aad584d65ed5b24/element/[object Object]/displayed": Request path contains unescaped characters
    at new ClientRequest (_http_client.js:115:13)
    at Object.request (http.js:42:10)
    at HttpRequest.createHttpRequest (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\http\request.js:138:55)
    at HttpRequest.send (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\http\request.js:217:29)
    at Promise (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\transport.js:193:15)
    at new Promise (<anonymous>)
    at Selenium2Protocol.sendProtocolAction (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\transport.js:191:12)
    at Selenium2Protocol.runProtocolAction (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\jsonwire.js:61:17)
    at Object.isElementDisplayed (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\actions.js:54:10)
    at Selenium2Protocol.executeProtocolAction (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\transport.js:239:48)

 Error while running .isElementDisplayed() protocol action: TypeError [ERR_UNESCAPED_CHARACTERS]: Error while trying to create HTTP request for "/wd/hub/session/5001c8b007da1e5c7aad584d65ed5b24/element/[object Object]/displayed": Request path contains unescaped characters
    at new ClientRequest (_http_client.js:115:13)
    at Object.request (http.js:42:10)
    at HttpRequest.createHttpRequest (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\http\request.js:138:55)
    at HttpRequest.send (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\http\request.js:217:29)
    at Promise (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\transport.js:193:15)
    at new Promise (<anonymous>)
    at Selenium2Protocol.sendProtocolAction (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\transport.js:191:12)
    at Selenium2Protocol.runProtocolAction (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\jsonwire.js:61:17)
    at Object.isElementDisplayed (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\actions.js:54:10)
    at Selenium2Protocol.executeProtocolAction (G:\RMS\TUMS-git-commit\webui\rms\node_modules\nightwatch\lib\transport\transport.js:239:48)

```

### 问题原因

`chromedriver`新版本以W3C标准兼容模式运行，会导致报错（具体原因未知）。

### 解决方案

在nightwatch的配置文件中，添加配置项chromeOptions设置w3c模式为false即可，验证可行。

```js
        chrome: {
            desiredCapabilities: {
                browserName: 'chrome',
                javascriptEnabled: true,
                acceptSslCerts: true,
                + chromeOptions: {
                +    w3c: false 
                + }
            }
        },
```

##   Assertion class must implement method/property "expected"

### 问题描述

由于nightwatch没有方法访问网页的localStorage，因此封装了一个assertion用于判断指定localStorage是否不为null。

但是运行的时候报错：

```js
TEST FAILURE: 1 error during execution; 0 tests failed, 1 passed (14.506s)

 × deviceDiscover.spec
 – 设备发现 (4.629s)

  Error: Error while running "storageNotNull" command: Assertion class must implement method/property "expected"
       at Array.forEach (<anonymous>)
```

### 问题原因

根据报错原因，发现assertion必须实现expected属性。

### 解决方案

在assertion中添加`expected`属性即可。

```js
exports.assertion = function (key) {
    this.message = 'Testing if localStorage <' + key + '> is not null.'
+   this.expected = null;
    this.value = function (res) {
      return res.value
    }
    this.pass = function (val) {
+     return val !== this.expected;
    }
    this.command = function (cb) {
        var self = this
        return this.api.execute(function (key) {
            return localStorage.getItem(key);
        }, [key], function (res) {
            cb.call(self, res);
        })
    }
}
```



