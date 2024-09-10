# electron-quick start遇到的一些问题

## volta

volta安装过程比较简单就跳过了

**问题**：volta安装node时失败，Could not download Node version registry；Please verify your internet connection

在volta安装路径比如```C:\Users\ ... \AppData\Local\Volta```新建hooks.json填入

```json
{
    "node": {
        "index": {
            "template": "https://mirrors.tuna.tsinghua.edu.cn/nodejs-release/index.json"
        },
        "distro": {
            "template": "https://mirrors.tuna.tsinghua.edu.cn/nodejs-release/v{{version}}/{{filename}}"
        }
    }
}
```

## 正式创建之前

安装volta之后，在工作目录下```volta pin node@xx.xx.xx```
**问题**：```npm install --save-dev electron```遇到unable to verify the first certificate
网上一般的解决方法如下，可以简单尝试

```
npm config set strict-ssl false
npm config set registry https://registry.npmmirror.com
```

个人尝试以上步骤无效，换用另一个方法

```
npm install -g cnpm --registry=https://registry.npmmirror.com   // 这条如果不成功，用管理员权限打开命令行执行
cnpm install --save-dev electron
```

## quick start

package.json结构参考

```json
{
  "name": "first_electron_app",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "electron ."
  },
  "author": "Flartiny",
  "license": "ISC",
  "volta": {
    "node": "18.20.4"
  },
  "devDependencies": {
    "electron": "^32.0.2"
  }
}
```

1. 创建main.js

```JavaScript
// main.js

// Modules to control application life and create native browser window
const { app, BrowserWindow } = require('electron')
const path = require('node:path')

const createWindow = () => {
  // Create the browser window.
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  // 加载 index.html
  mainWindow.loadFile('index.html')

  // 打开开发工具
  // mainWindow.webContents.openDevTools()
}

// 这段程序将会在 Electron 结束初始化
// 和创建浏览器窗口的时候调用
// 部分 API 在 ready 事件触发后才能使用。
app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    // 在 macOS 系统内, 如果没有已开启的应用窗口
    // 点击托盘图标时通常会重新创建一个新窗口
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// 除了 macOS 外，当所有窗口都被关闭的时候退出程序。 因此, 通常
// 对应用程序和它们的菜单栏来说应该时刻保持激活状态, 
// 直到用户使用 Cmd + Q 明确退出
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
})

// 在当前文件中你可以引入所有的主进程代码
// 也可以拆分成几个文件，然后用 require 导入。
```

2. 创建index.html

```html
<!--index.html-->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.

    <!-- 您也可以此进程中运行其他文件 -->
    <script src="./renderer.js"></script>
  </body>
</html>
```

3. 创建preload.js

```JavaScript
// preload.js

// 所有的 Node.js API接口 都可以在 preload 进程中被调用.
// 它拥有与Chrome扩展一样的沙盒。
window.addEventListener('DOMContentLoaded', () => {
    const replaceText = (selector, text) => {
      const element = document.getElementById(selector)
      if (element) element.innerText = text
    }
  
    for (const dependency of ['chrome', 'node', 'electron']) {
      replaceText(`${dependency}-version`, process.versions[dependency])
    }
  })
```

---
参考：
知乎《解决npm安装electron总失败的问题》 https://zhuanlan.zhihu.com/p/562217675
使用 Volta 管理Node https://blog.langpz.com/%E4%BD%BF%E7%94%A8-Volta-%E7%AE%A1%E7%90%86Node.html
electron官方文档 https://www.electronjs.org/zh/docs/latest/tutorial/quick-start
Electron打包配置 + package.json的常用配置项及其用法解析 https://blog.csdn.net/qq_41980754/article/details/119902572
使用npm init初始化项目 https://www.cnblogs.com/WD-NewDemo/p/11141384.html
