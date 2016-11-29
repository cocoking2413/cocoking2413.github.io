## electron入门笔记
=====================================

#### 简介

简单来说，Electron为用纯JavaScript创建桌面应用提供了运行时。原理是，Electron调用你在package.json中定义的main文件并执行它。main文件（通常被命名为main.js）会创建一个内含渲染完的web页面的应用窗口，并添加与你操作系统的原生GUI（图形界面）交互的功能。
    
详细地说，当用Electron启动一个应用，会创建一个主进程。这个主进程负责与你系统原生的GUI进行交互并为你的应用创建GUI（在你的应用窗口）。
![](http://newsget-cache.stor.sinaapp.com/0b998dc2ebd3441852e5423fc8e723c1.png)

仅启动主进程并不能给你的应用用户创建应用窗口。窗口是通过main文件里的主进程调用叫BrowserWindow的模块创建的。每个浏览器窗口会运行自己的渲染进程。渲染进程会在窗口中渲染出web页面（引用了CSS，JavaScript，图片等的HTML文件）。web页面是Chromium渲染的，因为各系统下标准是统一的的，所以兼容性很好。

举例来说，如果你有一个计算器应用，主进程会初始化一个窗口来呈现实际的web页面（计算器）。

虽说只有主进程才和系统原生GUI交互，还是有技术可以把部分任务转到渲染进程中运行。

主进程通过一套可直接调用的Electron模块与原生GUI交互，桌面应用可以使用所有的Node模块，如用node-notifier模块来推送系统通知，request模块来发起HTTP请求等。

#### hello world

#### example



