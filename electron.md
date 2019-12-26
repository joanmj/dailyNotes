# electron
可以用于跨平台app开发; ui可用react实现, ui+后台均可使用node的方法;
## 主要概念
  * 主进程:
    * 运行 package.json 里 main对应的js的进程被称为主进程
  * 渲染进程
  由于 Electron 使用 Chromium 来展示页面，所以 Chromium 的多进程结构也被充分利用。每个 Electron 的页面都在运行着自己的进程，这样的进程我们称之为渲染进程。
  * 主进程与渲染进程的区别
    * 主进程使用 BrowserWindow 实例创建网页
      * 每个 BrowserWindow 实例都在自己的渲染进程里运行着一个网页。当一个 BrowserWindow 实例被销毁后，相应的渲染进程也会被终止。
    * 主进程管理所有页面和与之对应的渲染进程。每个渲染进程都是相互独立的，并且只关心他们自己的网页。
  * 在主进程与渲染进程之间通讯
    *  ipc 模块
    *  remote 模块
  * 接口
    * electron的api
      * 指派给一种进程类型
        * 少数两种均可使用
      * 进程间通过remote模块暴露通常只能在主进程中获取的api
    * node.js的api
      * 所有都可用
      * 先npm install --save name
      * require;
      * 原生模块需要编译
        * TODO: 怎么编译;
    *
## 应用功能
  * Notification
    * Win
      * 高级通知
      * 免打扰模式/演示模式
    * Mac
      * 高级通知
      * 勿扰/会话状态
    * Linux
  * Recent documents
  * progress-bar
    * 任务栏进度条
  * macOS dock
  * windows taskbar
  * linux desktop ations
  * key shortcuts
  * online-offline event/state
  * native file drag to
  * offscreen rendering
## 辅助功能
  * spectron
    * 测试election app的框架
    * 可以审查window和webview标签
  * Devtron
    * Developer Tools中有一个Devtron的标签页
    * Accessibility
      * 可对页面进行审核/排序/筛选
  * 启用
## debugging
可以用chrome的developer tools;
编程方式打开:
集成測試:
  可自動化, 使用Travis/Jenkins等, 需要配置Xvbf,
  * [Xvbf](https://en.wikipedia.org/wiki/Xvfb):
  * Jenkins 有Xvfb插件:https://wiki.jenkins-ci.org/display/JENKINS/Xvfb+Plugin

* DevTools, 需要配置
  * 经过测试, 可用的有:
    * Electron 只支持有限的chrome.* API，所以，一些扩展程序如果使用了不支持的chrome.* API，它可能会无法正常工作。 以下 DevTools 扩展程序已经通过测试，可以在Electron中正常工作：
      Ember Inspector
      React Developer Tools
      Backbone Debugger
      jQuery Debugger
      AngularJS Batarang
      Vue.js devtools
      Cerebral Debugger
      Redux DevTools Extension
      MobX Developer Tools
* 自动化测试:
TODO: node's builtin IPC-over-STDIO
    link: https://electronjs.org/docs/tutorial/automated-testing-with-a-custom-driver

```js
const { BrowserWindow } = require('electron')
let win = new BrowserWindow()
win.webContents.openDevTools()
```
### 主进程调试:
1. 命令行开关
  * --inspect=[port]
  * --inspect-brk=[port]
    会在JavaScript 脚本的第一行暂停运行。
electron启动时打开开关:
```bash
electron --inspect=5858 your/app
```
当这个开关用于 Electron 时，它将会监听 V8 引擎中有关 port 的调试器协议信息。 默认的port 是 5858
2. 外部调试器
  * vscode
  * 访问 chrome://inspect 选择electron应用程序;
## ui中调用后台模块
  后台设置nodeIntegration为true;
  ```js
  const {
    app,
    BrowserWindow,
    ipcMain
  } = require('electron');
  app.on('ready', () => {
    onlineStatusWindow = new BrowserWindow({
      width: 100,
      height: 100,
      show: true,
      webPreferences: {
        nodeIntegration: true
      }
    });
    onlineStatusWindow.loadURL(`file://${__dirname}/online-status.html`);
  });
  ```
  UI中直接require:
  ```html
  <!DOCTYPE html>
  <html>
  <body>
  <script>
    const { ipcRenderer } = require('electron')
    const updateOnlineStatus = () => {
      ipcRenderer.send('online-status-changed', navigator.onLine ? 'online' : 'offline');
    }

    window.addEventListener('online',  updateOnlineStatus)
    window.addEventListener('offline',  updateOnlineStatus)

    updateOnlineStatus()
  </script>
  </body>
  </html>
  ```
## selenium
### 安装webdrivier
  按官网提示如下:
  ```bash
  $ npm install electron-chromedriver
  $ ./node_modules/.bin/chromedriver
  Starting ChromeDriver (v2.10.291558) on port 9515
  Only local connections are allowed.
  ```

  但是安装后运行提示错误:

  需要以管理员身份启动powershell后运行:
  ```bash
  set-ExecutionPolicy RemoteSigned：

  执行策略更改
  执行策略可以防止您执行不信任的脚本。更改执行策略可能会使您面临 about_Execution_Policies
  帮助主题中所述的安全风险。是否要更改执行策略?
  [Y] 是(Y)  [N] 否(N)  [S] 挂起(S)  [?] 帮助 (默认值为“Y”):
  ```
  输入Y,即可修改;
  再次运行就可以了;
## projects
### 浏览器:
https://blog.jscrambler.com/building-a-web-browser-using-electron/
### react项目模板;
electron-react-boilerplate
