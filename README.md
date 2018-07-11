## 先决条件 —— 安装electron
npm install -g electron-prebuilt  

## 运行
electron .

## 发布
发布32位exe应用程序：npm run package32
发布64位exe应用程序：npm run package




## 使用Electron Hybird架构

使用Electron Hybird架构嵌入本Web App的时候，需要对项目下/src/index.html文件进行少许改进。要在页面head节内增加：  

<!--这里用来适配Electron : begin  -->
    <script>
        window.nodeRequire = require;
        delete window.require;
        delete window.exports;
        delete window.module;
    </script>
<!--这里用来适配Electron : end  -->




## Electron基础 - 解决无法使用jQuery/RequireJS/Meteor/AngularJS 的问题


jQuery等新版本的框架，在Electron中使用普通的引入的办法会引发异常，原因始Electron默认启用了Node.js的require模块，而这些框架为了支持commondJS标准，当Window中存在require时，会启用模块引入的方式。以下是几种解决办法：
1. 去掉框架中的模块引入判断代码，比如jQuery中的第一行代码中的：
```
!function(a,b)
{
    "object"==typeof module&&"object"==typeof module.exports?       module.exports=a.document?b(a,!0):function(a)
    {
        if(!a.document)
        throw new Error("jQuery requires a window with a document");
        return b(a)}:b(a)
    }
}

```


改成：


```
!function(a,b){b(a)}
```
以上方法只适用于jQuery框架，其它框架需要自行研究其模块结构，删除相应代码即可。

2. 使用Electron官方论坛提供的方法，需要改变require的写法，此方法各个框架通用:
```
<head>
<script>
window.nodeRequire = require;
delete window.require;
delete window.exports;
delete window.module;
</script>
<script type="text/javascript" src="jquery.js"></script>
</head>
```

如果你是刚开始开发Electron桌面端程序，建议使用第二种方法，注意引入习惯即可。如果已经开发了一定量的，不推荐使用第二种用法。如果你不想使用Node.js模块，大可以去掉require模块化引入，直接使用以下方法禁用Node.js的require模块化引入，即可正常使用任何框架

```
// In the main process.
let win = new BrowserWindow({
  webPreferences: {
    nodeIntegration: false
  }
});
Electron
```

# Quick Start

Electron enables you to create desktop applications with pure JavaScript by providing a runtime with rich native (operating system) APIs. You could see it as a variant of the Node.js runtime that is focused on desktop applications instead of web servers.

This doesn’t mean Electron is a JavaScript binding to graphical user interface (GUI) libraries. Instead, Electron uses web pages as its GUI, so you could also see it as a minimal Chromium browser, controlled by JavaScript.

### Main Process
In Electron, the process that runs package.json’s main script is called the main process. The script that runs in the main process can display a GUI by creating web pages.

### Renderer Process
Since Electron uses Chromium for displaying web pages, Chromium’s multi-process architecture is also used. Each web page in Electron runs in its own process, which is called the renderer process.

In normal browsers, web pages usually run in a sandboxed environment and are not allowed access to native resources. Electron users, however, have the power to use Node.js APIs in web pages allowing lower level operating system interactions.

### Differences Between Main Process and Renderer Process
The main process creates web pages by creating BrowserWindow instances. Each BrowserWindow instance runs the web page in its own renderer process. When a BrowserWindow instance is destroyed, the corresponding renderer process is also terminated.

The main process manages all web pages and their corresponding renderer processes. Each renderer process is isolated and only cares about the web page running in it.

In web pages, calling native GUI related APIs is not allowed because managing native GUI resources in web pages is very dangerous and it is easy to leak resources. If you want to perform GUI operations in a web page, the renderer process of the web page must communicate with the main process to request that the main process perform those operations.

In Electron, we have several ways to communicate between the main process and renderer processes. Like ipcRenderer and ipcMain modules for sending messages, and the remote module for RPC style communication. There is also an FAQ entry on how to share data between web pages.

## Write your First Electron App

Generally, an Electron app is structured like this:
```
your-app/
├── package.json
├── main.js
└── index.html
```
The format of package.json is exactly the same as that of Node’s modules, and the script specified by the main field is the startup script of your app, which will run the main process. An example of your package.json might look like this:
```
{
  "name"    : "your-app",
  "version" : "0.1.0",
  "main"    : "main.js"
}
```
Note: If the main field is not present in package.json, Electron will attempt to load an index.js.

The main.js should create windows and handle system events, a typical example being:
```
const {app, BrowserWindow} = require('electron')
const path = require('path')
const url = require('url')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let win

function createWindow () {
  // Create the browser window.
  win = new BrowserWindow({width: 800, height: 600})

  // and load the index.html of the app.
  win.loadURL(url.format({
    pathname: path.join(__dirname, 'index.html'),
    protocol: 'file:',
    slashes: true
  }))

  // Open the DevTools.
  win.webContents.openDevTools()

  // Emitted when the window is closed.
  win.on('closed', () => {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    win = null
  })
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.on('ready', createWindow)

// Quit when all windows are closed.
app.on('window-all-closed', () => {
  // On macOS it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // On macOS it's common to re-create a window in the app when the
  // dock icon is clicked and there are no other windows open.
  if (win === null) {
    createWindow()
  }
})

// In this file you can include the rest of your app's specific main process
// code. You can also put them in separate files and require them here.
```
Finally the index.html is the web page you want to show:
```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using node <script>document.write(process.versions.node)</script>,
    Chrome <script>document.write(process.versions.chrome)</script>,
    and Electron <script>document.write(process.versions.electron)</script>.
  </body>
</html>
```
Run your app

Once you’ve created your initial main.js, index.html, and package.json files, you’ll probably want to try running your app locally to test it and make sure it’s working as expected.

electron
electron is an npm module that contains pre-compiled versions of Electron.

If you’ve installed it globally with npm, then you will only need to run the following in your app’s source directory:

```
electron .
```
If you’ve installed it locally, then run:

macOS / Linux
```
$ ./node_modules/.bin/electron .
```
Windows
```
$ .\node_modules\.bin\electron .
```
Manually Downloaded Electron Binary
If you downloaded Electron manually, you can also use the included binary to execute your app directly.

Windows
```
$ .\electron\electron.exe your-app\
```
Linux
```
$ ./electron/electron your-app/
```
macOS
```
$ ./Electron.app/Contents/MacOS/Electron your-app/
```
Electron.app here is part of the Electron’s release package, you can download it from here.

Run as a distribution
After you’re done writing your app, you can create a distribution by following the Application Distribution guide and then executing the packaged app.

Try this Example
Clone and run the code in this tutorial by using the electron/electron-quick-start repository.

Note: Running this requires Git and Node.js (which includes npm) on your system.
```
# Clone the repository
$ git clone https://github.com/electron/electron-quick-start
# Go into the repository
$ cd electron-quick-start
# Install dependencies
$ npm install
# Run the app
$ npm start
```
For more example apps, see the list of [boilerplates](https://electron.atom.io/community/#boilerplates) created by the awesome electron community.

