## electron入门笔记
=====================================

#### 简介

简单来说，Electron为用纯JavaScript创建桌面应用提供了运行时。原理是，Electron调用你在package.json中定义的main文件并执行它。main文件（通常被命名为main.js）会创建一个内含渲染完的web页面的应用窗口，并添加与你操作系统的原生GUI（图形界面）交互的功能。
    
详细地说，当用Electron启动一个应用，会创建一个主进程。这个主进程负责与你系统原生的GUI进行交互并为你的应用创建GUI（在你的应用窗口）。
![img](http://newsget-cache.stor.sinaapp.com/0b998dc2ebd3441852e5423fc8e723c1.png)

仅启动主进程并不能给你的应用用户创建应用窗口。窗口是通过main文件里的主进程调用叫BrowserWindow的模块创建的。每个浏览器窗口会运行自己的渲染进程。渲染进程会在窗口中渲染出web页面（引用了CSS，JavaScript，图片等的HTML文件）。web页面是Chromium渲染的，因为各系统下标准是统一的的，所以兼容性很好。

举例来说，如果你有一个计算器应用，主进程会初始化一个窗口来呈现实际的web页面（计算器）。

虽说只有主进程才和系统原生GUI交互，还是有技术可以把部分任务转到渲染进程中运行。

主进程通过一套可直接调用的Electron模块与原生GUI交互，桌面应用可以使用所有的Node模块，如用node-notifier模块来推送系统通知，request模块来发起HTTP请求等。

#### 环境配置

1. 安装nodejs，npm,vscode
2. npm下载慢，安装cnpm
    ```bash
    cnpm npm install -g cnpm
    ```
3. 安装electron
    ```bash
    cnpm  install -g electron
    ```
4. 打包工具electron-packager
    ```bash
    cnpm install -g electron-packager
    ```


#### hello world

1. 需要版本控制先```git init```
2. 构建node配置```npm init```
3. 写入依赖
    ```bash
    cnpm install electron --save

    cnpm install electron-packager --save-dev 
    ```
4. 创建main.js
    ```js
    const {app, BrowserWindow} = require('electron')

    let win

    function createWindow () {
    
    win = new BrowserWindow({width: 800, height: 600})

    win.loadURL(`file://${__dirname}/index.html`)

    win.webContents.openDevTools()

    // 处理窗口关闭
    win.on('closed', () => {
        win = null
    })
    }

    // Electron初始化完成
    app.on('ready', createWindow)

    // 处理退出
    app.on('window-all-closed', () => {
    if (process.platform !== 'darwin') {
        app.quit()
    }
    })

    app.on('activate', () => {
    if (win === null) {
        createWindow()
    }
    })
    ```
5. 创建index.html    
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>Hello World!</title>
    </head>
    <body>
        <h1>Hello World!</h1>
        node 当前使用的node为<script>document.write(process.versions.node)</script>,
        Chrome 为<script>document.write(process.versions.chrome)</script>,
        和 Electron 为<script>document.write(process.versions.electron)</script>.
    </body>
    </html>
    ```
6. 修改package.json
    ```json
    {
        "name": "maintest",
        "version": "1.0.0",
        "description": "hello electron program",
        "main": "main.js",
        "scripts": {
            "test": "echo \"Error: no test specified\" && exit 1",
            "start":"electron .",
            "win32pack":"electron-packager . --platform=win32 --overwrite"
        },
        "author": "coco king",
        "license": "MIT",
        "devDependencies": {
            "electron-packager": "^5.0.1",
            "electron-prebuilt": "^1.4.10"
        },
        "dependencies": {
            "nconf": "^0.7.2",
            "gulp": "^3.0.0"
        }
    }
    ```
7. 测试执行  
    直接执行
    ```bash
    electron .
    ```
    或通过npm(注意package.json的配置项)
    ```bash
    npm start
    ```
8. 打包输出
    ```bash
    electron-packager . --platform=win32
    ```
    或通过npm(注意package.json的配置项)
    ```bash
    npm run win32pack
    ```




