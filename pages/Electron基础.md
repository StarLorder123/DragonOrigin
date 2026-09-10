# 1.  Electron介绍
- Electron是一个使用 JavaScript、HTML 和 CSS 构建桌面应用程序的框架。 嵌入 Chromium 和 Node.js 到 二进制的 Electron 允许您保持一个 JavaScript 代码代码库并创建 在Windows上运行的跨平台应用 macOS和Linux——不需要本地开发经验。
- 文档网址：[https://www.electronjs.org/zh/docs/latest/](https://www.electronjs.org/zh/docs/latest/)
- # 2.  Electron 特性
- ## 2.1.  流程模型
- Electron 继承了来自 Chromium 的多进程架构，这使得此框架在架构上非常相似于一个现代的网页浏览器。
- 单进程模型意味着打开每个标签页的开销比较少，但同时，一个网站的崩溃或者无响应会影响到整个浏览器。
- 为了解决这个问题，Chrome 团队决定让每个标签页在自己的进程中渲染， 从而限制了一个网页上的有误或恶意代码可能导致的对整个应用程序造成的伤害。 然后用单个浏览器进程控制这些标签页进程，以及整个应用程序的生命周期。
- Electron 应用程序的结构非常相似。 作为应用开发者，你将控制两种类型的进程：主进程 和 渲染器进程。
- 每个 Electron 应用都有一个单一的主进程，作为应用程序的入口点。 主进程在 Node.js 环境中运行，这意味着它具有 require 模块和使用所有 Node.js API 的能力。
- 主进程的主要目的是使用 BrowserWindow 模块创建和管理应用程序窗口。
- BrowserWindow 类的每个实例创建一个应用程序窗口，且在单独的渲染器进程中加载一个网页。 您可从主进程用 window 的 webContent 对象与网页内容进行交互。
  
  ```
  const { BrowserWindow } = require('electron')
  
  const win = new BrowserWindow({ width: 800, height: 1500 })
  win.loadURL('https://github.com')
  
  const contents = win.webContents
  console.log(contents)
  ```
  
  注意：渲染器进程也是为 web embeds 而被创建的，例如 BrowserView 模块。 嵌入式网页内容也可访问 webContents 对象。
- 由于 BrowserWindow 模块是一个 EventEmitter， 所以也可以为各种用户事件 ( 例如，最小化 或 最大化您的窗口 ) 添加处理程序。
- 当一个 BrowserWindow 实例被销毁时，与其相应的渲染器进程也会被终止。
- 应用程序的生命周期
- 主进程还能通过 Electron 的 app 模块来控制您应用程序的生命周期。 该模块提供了一整套的事件和方法，可以让您用来添加自定义的应用程序行为 (例如：以编程方式退出您的应用程序、修改应用程序坞，或显示一个关于面板) 。
- 示例
  
  ```
  // quitting the app when no windows are open on non-macOS platforms
  app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit()
  })
  ```
- 原生的API
- 为了使 Electron 的功能不仅仅限于对网页内容的封装，主进程也添加了自定义的 API 来与用户的作业系统进行交互。 Electron 有着多种控制原生桌面功能的模块，例如菜单、对话框以及托盘图标。
- ### 2.1.1.  渲染器进程
- 每个 Electron 应用都会为每个打开的 BrowserWindow ( 与每个网页嵌入 ) 生成一个单独的渲染器进程。 洽如其名，渲染器负责 ***渲染*** 网页内容。 所以实际上，运行于渲染器进程中的代码是须遵照网页标准的 (至少就目前使用的 Chromium 而言是如此) 。
- 因此，一个浏览器窗口中的所有的用户界面和应用功能，都应与您在网页开发上使用相同的工具和规范来进行攥写。
	- 以一个 HTML 文件作为渲染器进程的入口点。
	- 使用层叠样式表 (Cascading Style Sheets, CSS) 对 UI 添加样式。
	- 通过 <script> 元素可添加可执行的 JavaScript 代码。
- 此外，这也意味着渲染器无权直接访问 require 或其他 Node.js API。 为了在渲染器中直接包含 NPM 模块，您必须使用与在 web 开发时相同的打包工具 (例如 webpack 或 parcel)
- ### 2.1.2.  Preload脚本
- 预加载（preload）脚本包含了那些执行于渲染器进程中，且先于网页内容开始加载的代码 。 这些脚本虽运行于渲染器的环境中，却因能访问 Node.js API 而拥有了更多的权限。
- 预加载脚本可以在 BrowserWindow 构造方法中的 webPreferences 选项里被附加到主进程。
  
  ```
  const { BrowserWindow } = require('electron')
  // ...
  const win = new BrowserWindow({
  webPreferences: {
    preload: 'path/to/preload.js'
  }
  })
  // ...
  ```
- 因为预加载脚本与浏览器共享同一个全局 Window 接口，并且可以访问 Node.js API，所以它通过在全局 window 中暴露任意 API 来增强渲染器，以便你的网页内容使用。
- 虽然预加载脚本与其所附着的渲染器在共享着一个全局 window 对象，但您并不能从中直接附加任何变动到 window 之上，因为 contextIsolation 是默认的。
  
  ```
  window.myAPI = {
  desktop: true
  }
  ```
  
  ```
  console.log(window.myAPI)
  // => undefined
  ```
- 语境隔离（Context Isolation）意味着预加载脚本与渲染器的主要运行环境是隔离开来的，以避免泄漏任何具特权的 API 到您的网页内容代码中。
- 取而代之，我们將使用 contextBridge 模块来安全地实现交互：
  
  ```
  const { contextBridge } = require('electron')
  
  contextBridge.exposeInMainWorld('myAPI', {
  desktop: true
  })
  ```
  
  ```
  console.log(window.myAPI)
  // => { desktop: true }
  ```
- 此功能对两个主要目的來說非常有用：
	- 通过暴露 ipcRenderer 帮手模块于渲染器中，您可以使用 进程间通讯 ( inter-process communication, IPC ) 来从渲染器触发主进程任务 ( 反之亦然 ) 。
	- 如果您正在为远程 URL 上托管的现有 web 应用开发 Electron 封裝，则您可在渲染器的 window 全局变量上添加自定义的属性，好在 web 客户端用上仅适用于桌面应用的设计逻辑 。
- ### 2.1.3.  效率进程
- 每个Electron应用程序都可以使用主进程生成多个子进程UtilityProcess API。 主进程在 Node.js 环境中运行，这意味着它具有 require 模块和使用所有 Node.js API 的能力。 效率进程可用于托管，例如：不受信任的服务， CPU 密集型任务或以前容易崩溃的组件 托管在主进程或使用Node.jschild_process.fork API 生成的进程中。 效率进程和 Node 生成的进程之间的主要区别.js child_process模块是实用程序进程可以建立通信 通道与使用MessagePort的渲染器进程。 当需要从主进程派生一个子进程时，Electron 应用程序可以总是优先使用 效率进程 API 而不是Node.js child_process.fork API。
- 特定于进程的模块别名(TypeScript)
- Electron的npm包还导出了包含Electron的TypeScript类型定义子集的子路径
	- Electron /main包括所有主要过程模块的类型。
	- Electron /renderer包括所有renderer进程模块的类型。
	- Electron /common包括可以在主进程和渲染进程中运行的模块类型。
- 这些别名对运行时没有影响，但可用于类型检查和自动完成
- ## 2.2.  上下文隔离
- 上下文隔离功能将确保您的 预加载脚本 和 Electron的内部逻辑 运行在所加载的 webcontent网页 之外的另一个独立的上下文环境里。 这对安全性很重要，因为它有助于阻止网站访问 Electron 的内部组件 和 您的预加载脚本可访问的高等级权限的API 。
- 这意味着，实际上，您的预加载脚本访问的 window 对象并不是网站所能访问的对象。 例如，如果您在预加载脚本中设置 window.hello = 'wave' 并且启用了上下文隔离，当网站尝试访问window.hello对象时将返回 undefined。
- 自 Electron 12 以来，默认情况下已启用上下文隔离，并且它是 所有应用程序推荐的安全设置。
- Electron 提供一种专门的模块来无阻地帮助您完成这项工作。 contextBridge 模块可以用来安全地从独立运行、上下文隔离的预加载脚本中暴露 API 给正在运行的渲染进程。 API 还可以像以前一样，从 window.myAPI 网站上访问。
  
  ```
  // 在上下文隔离启用的情况下使用预加载
  const { contextBridge } = require('electron')
  
  contextBridge.exposeInMainWorld('myAPI', {
  doAThing: () => {}
  })
  ```
  
  ```
  // 在渲染器进程使用导出的 API
  window.myAPI.doAThing()
  ```
- ## 2.3.  进程间通信
- 进程间通信 (IPC) 是在 Electron 中构建功能丰富的桌面应用程序的关键部分之一。 由于主进程和渲染器进程在 Electron 的进程模型具有不同的职责，因此 IPC 是执行许多常见任务的唯一方法。
- ### 2.3.1.  IPC通道
- 在 Electron 中，进程使用 ipcMain 和 ipcRenderer 模块，通过开发人员定义的“通道”传递消息来进行通信。 这些通道是 任意 （您可以随意命名它们）和 双向 （您可以在两个模块中使用相同的通道名称）的。
- ### 2.3.2.  渲染进程到主进程（单向）
- 要将单向 IPC 消息从渲染器进程发送到主进程，您可以使用 ipcRenderer.send API 发送消息，然后使用 ipcMain.on API 接收。
- 1. Listen for events with ipcMain.on
  
  ```
  const { app, BrowserWindow, ipcMain } = require('electron')
  const path = require('node:path')
  
  // ...
  
  function handleSetTitle (event, title) {
  const webContents = event.sender
  const win = BrowserWindow.fromWebContents(webContents)
  win.setTitle(title)
  }
  
  function createWindow () {
  const mainWindow = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })
  mainWindow.loadFile('index.html')
  }
  
  app.whenReady().then(() => {
  ipcMain.on('set-title', handleSetTitle)
  createWindow()
  })
  // ...
  ```
- 上面的 handleSetTitle 回调函数有两个参数：一个 IpcMainEvent 结构和一个 title 字符串。 每当消息通过 set-title 通道传入时，此函数找到附加到消息发送方的 BrowserWindow 实例，并在该实例上使用 win.setTitle API。
- 2. 通过预加载脚本暴露 ipcRenderer.send
- 要将消息发送到上面创建的监听器，您可以使用 ipcRenderer.send API。 默认情况下，渲染器进程没有权限访问 Node.js 和 Electron 模块。 作为应用开发者，您需要使用 contextBridge API 来选择要从预加载脚本中暴露哪些 API。
- 在您的预加载脚本中添加以下代码，向渲染器进程暴露一个全局的 window.electronAPI 变量。
  
  ```
  const { contextBridge, ipcRenderer } = require('electron')
  
  contextBridge.exposeInMainWorld('electronAPI', {
  setTitle: (title) => ipcRenderer.send('set-title', title)
  })
  ```
- 3. 构建渲染器进程 UI
- 在 BrowserWindow 加载的我们的 HTML 文件中，添加一个由文本输入框和按钮组成的基本用户界面：
  
  ```
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Hello World!</title>
  </head>
  <body>
    Title: <input id="title"/>
    <button id="btn" type="button">Set</button>
    <script src="./renderer.js"></script>
  </body>
  </html>
  ```
- 为了使这些元素具有交互性，我们将在导入的 renderer.js 文件中添加几行代码，以利用从预加载脚本中暴露的 window.electronAPI 功能：
  
  ```
  const setButton = document.getElementById('btn')
  const titleInput = document.getElementById('title')
  setButton.addEventListener('click', () => {
  const title = titleInput.value
  window.electronAPI.setTitle(title)
  })
  ```
- ### 2.3.3.  渲染进程到主进程（双向）
- 双向 IPC 的一个常见应用是从渲染器进程代码调用主进程模块并等待结果。 这可以通过将 ipcRenderer.invoke 与 ipcMain.handle 搭配使用来完成。
- 在下面的示例中，我们将从渲染器进程打开一个原生的文件对话框，并返回所选文件的路径。
- 对于此演示，您需要将代码添加到主进程、渲染器进程和预加载脚本。 完整代码如下，我们将在后续章节中对每个文件进行单独解释。
  
  ```
  const { app, BrowserWindow, ipcMain, dialog } = require('electron/main')
  const path = require('node:path')
  
  async function handleFileOpen () {
  const { canceled, filePaths } = await dialog.showOpenDialog()
  if (!canceled) {
    return filePaths[0]
  }
  }
  
  function createWindow () {
  const mainWindow = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })
  mainWindow.loadFile('index.html')
  }
  
  app.whenReady().then(() => {
  ipcMain.handle('dialog:openFile', handleFileOpen)
  createWindow()
  app.on('activate', function () {
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
  })
  
  app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
  })
  ```
- 使用 ipcMain.handle 监听事件
- 在主进程中，我们将创建一个 handleFileOpen() 函数，它调用 dialog.showOpenDialog 并返回用户选择的文件路径值。 每当渲染器进程通过 dialog:openFile 通道发送 ipcRender.invoke 消息时，此函数被用作一个回调。 然后，返回值将作为一个 Promise 返回到最初的 invoke 调用。
  
  ```
  const { app, BrowserWindow, dialog, ipcMain } = require('electron')
  const path = require('node:path')
  
  // ...
  
  async function handleFileOpen () {
  const { canceled, filePaths } = await dialog.showOpenDialog({})
  if (!canceled) {
    return filePaths[0]
  }
  }
  
  function createWindow () {
  const mainWindow = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })
  mainWindow.loadFile('index.html')
  }
  
  app.whenReady().then(() => {
  ipcMain.handle('dialog:openFile', handleFileOpen)
  createWindow()
  })
  // ...
  ```
- 通过预加载脚本暴露 ipcRenderer.invoke
- 在预加载脚本中，我们暴露了一个单行的 openFile 函数，它调用并返回 ipcRenderer.invoke('dialog:openFile') 的值。 我们将在下一步中使用此 API 从渲染器的用户界面调用原生对话框。
  
  ```
  const { contextBridge, ipcRenderer } = require('electron')
  
  contextBridge.exposeInMainWorld('electronAPI', {
  openFile: () => ipcRenderer.invoke('dialog:openFile')
  })
  ```
- 构建渲染器进程 UI
- 最后，让我们构建加载到 BrowserWindow 中的 HTML 文件。
  
  ```
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Dialog</title>
  </head>
  <body>
    <button type="button" id="btn">Open a File</button>
    File path: <strong id="filePath"></strong>
    <script src='./renderer.js'></script>
  </body>
  </html>
  ```
- 用户界面包含一个 #btn 按钮元素，将用于触发我们的预加载 API，以及一个 #filePath 元素，将用于显示所选文件的路径。 要使这些部分起作用，需要在渲染器进程脚本中编写几行代码：
  
  ```
  const btn = document.getElementById('btn')
  const filePathElement = document.getElementById('filePath')
  
  btn.addEventListener('click', async () => {
  const filePath = await window.electronAPI.openFile()
  filePathElement.innerText = filePath
  })
  ```
- 在上面的代码片段中，我们监听 #btn 按钮的点击，并调用 window.electronAPI.openFile() API 来激活原生的打开文件对话框。 然后我们在 #filePath 元素中显示选中文件的路径。
- ### 2.3.4.  主进程到渲染进程
- 将消息从主进程发送到渲染器进程时，需要指定是哪一个渲染器接收消息。 消息需要通过其 WebContents 实例发送到渲染器进程。 此 WebContents 实例包含一个 send 方法，其使用方式与 ipcRenderer.send 相同。
- 为了演示此模式，我们将构建一个由原生操作系统菜单控制的数字计数器。
- 对于此演示，您需要将代码添加到主进程、渲染器进程和预加载脚本。 完整代码如下，我们将在后续章节中对每个文件进行单独解释。
  
  ```
  const { app, BrowserWindow, Menu, ipcMain } = require('electron/main')
  const path = require('node:path')
  
  function createWindow () {
  const mainWindow = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })
  
  const menu = Menu.buildFromTemplate([
    {
      label: app.name,
      submenu: [
        {
          click: () => mainWindow.webContents.send('update-counter', 1),
          label: 'Increment'
        },
        {
          click: () => mainWindow.webContents.send('update-counter', -1),
          label: 'Decrement'
        }
      ]
    }
  
  ])
  
  Menu.setApplicationMenu(menu)
  mainWindow.loadFile('index.html')
  
  // Open the DevTools.
  mainWindow.webContents.openDevTools()
  }
  
  app.whenReady().then(() => {
  ipcMain.on('counter-value', (_event, value) => {
    console.log(value) // will print value to Node console
  })
  createWindow()
  
  app.on('activate', function () {
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
  })
  
  app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
  })
  ```
- 使用 webContents 模块发送消息
- 对于此演示，我们需要首先使用 Electron 的 Menu 模块在主进程中构建一个自定义菜单，该模块使用 webContents.send API 将 IPC 消息从主进程发送到目标渲染器。
  
  ```
  const { app, BrowserWindow, Menu, ipcMain } = require('electron')
  const path = require('node:path')
  
  function createWindow () {
  const mainWindow = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })
  
  const menu = Menu.buildFromTemplate([
    {
      label: app.name,
      submenu: [
        {
          click: () => mainWindow.webContents.send('update-counter', 1),
          label: 'Increment'
        },
        {
          click: () => mainWindow.webContents.send('update-counter', -1),
          label: 'Decrement'
        }
      ]
    }
  ])
  Menu.setApplicationMenu(menu)
  
  mainWindow.loadFile('index.html')
  }
  // ...
  ```
- 通过预加载脚本暴露 ipcRenderer.on
- 与前面的渲染器到主进程的示例一样，我们使用预加载脚本中的 contextBridge 和 ipcRenderer 模块向渲染器进程暴露 IPC 功能：
  
  ```
  const { contextBridge, ipcRenderer } = require('electron')
  
  contextBridge.exposeInMainWorld('electronAPI', {
  onUpdateCounter: (callback) => ipcRenderer.on('update-counter', (_event, value) => callback(value))
  })
  ```
- 构建渲染器进程 UI
- 为了将它们联系在一起，我们将在加载的 HTML 文件中创建一个接口，其中包含一个 #counter 元素，我们将使用该元素来显示值：
  
  ```
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Menu Counter</title>
  </head>
  <body>
    Current value: <strong id="counter">0</strong>
    <script src="./renderer.js"></script>
  </body>
  </html>
  ```
- 最后，为了更新 HTML 文档中的值，我们将添加几行 DOM 操作的代码，以便在每次触发 update-counter 事件时更新 #counter 元素的值。
  
  ```
  const counter = document.getElementById('counter')
  
  window.electronAPI.onUpdateCounter((value) => {
  const oldValue = Number(counter.innerText)
  const newValue = oldValue + value
  counter.innerText = newValue.toString()
  })
  ```
- 在上面的代码中，我们将回调传递给从预加载脚本中暴露的 window.electronAPI.onUpdateCounter 函数。 第二个 value 参数对应于我们传入 webContents.send 函数的 1 或 -1，该函数是从原生菜单调用的。
- ### 2.3.5.  渲染进程到渲染进程
- 没有直接的方法可以使用 ipcMain 和 ipcRenderer 模块在 Electron 中的渲染器进程之间发送消息。 为此，您有两种选择：
	- 将主进程作为渲染器之间的消息代理。 这需要将消息从一个渲染器发送到主进程，然后主进程将消息转发到另一个渲染器。
	- 从主进程将一个 MessagePort 传递到两个渲染器。 这将允许在初始设置后渲染器之间直接进行通信。
- ## 2.4.  进程沙盒化
- Chromium的一个关键安全特性是，进程可以在沙盒中执行。 沙盒通过限制对大多数系统资源的访问来减少恶意代码可能造成的伤害 — 沙盒化的进程只能自由使用CPU周期和内存。 为了执行需要额外权限的操作，沙盒处的进程通过专用通信渠道将任务下放给更大权限的进程。
- 当 Electron 中的渲染进程被沙盒化时，它们的行为与常规 Chrome 渲染器一样。 一个沙盒化的渲染器不会有一个 Node.js 环境。
- 因此，在沙盒中，渲染进程只能透过 进程间通讯 (inter-process communication, IPC) 委派任务给主进程的方式， 来执行需权限的任务 (例如：文件系统交互，对系统进行更改或生成子进程) 。
- 为了让渲染进程能与主进程通信，附属于沙盒化的渲染进程的 preload 脚本中仍可使用一部分以 Polyfill 形式实现的 Node.js API。 有一个与 Node 中类似的 require 函数提供了出来，但只能载入 Electron 和 Node 内置模块的一个子集：
	- electron (以下是渲染进程的模块: contextBridge, crashReporter, ipcRenderer, nativeImage, webFrame)
	- 事件
	- timers
	- url
- 对于大多数应用程序来说，沙盒是最佳选择。 在某些与沙盒不兼容的使用情况下（例如，在渲染器中使用原生的 Node.js 模块时），可以禁用特定进程的沙盒。 但这会带来安全风险，特别是当未受信任的代码或内容存在于未沙盒化的进程中时。
- 在 Electron 中，可通过在 BrowserWindow 构造函数中使用 sandbox: false选项来针对每个进程禁用渲染器沙盒。
  
  ```
  app.whenReady().then(() => {
  const win = new BrowserWindow({
    webPreferences: {
      sandbox: false
    }
  })
  win.loadURL('https://google.com')
  })
  ```
- 在渲染器中启用 nodeIntegration 时，沙盒也会被禁用。 可以通过在 BrowserWindow 构造函数中添加 nodeIntegration: true 标志的来实现。
  
  ```
  app.whenReady().then(() => {
  const win = new BrowserWindow({
    webPreferences: {
      nodeIntegration: true
    }
  })
  win.loadURL('https://google.com')
  })
  ```
- 也可以调用 app.enableSandbox API 来强制沙盒化所有渲染器。 注意，此 API 必须在应用的 ready 事件之前调用。
  
  ```
  app.enableSandbox()
  app.whenReady().then(() => }
  // 因为调用了app.enableSandbox()，所以任何sandbox:false的调用都会被覆盖。
  const win = new BrowserWindow()
  win.loadURL('https://google.com')
  })
  ```
- ## 2.5.  消息端口
- MessagePort是一个允许在不同上下文之间传递消息的Web功能。 就像 window.postMessage, 但是在不同的通道上。 此文档的目标是描述 Electron 如何扩展 Channel Messaging model ，并举例说明如何在应用中使用 MessagePorts
- 下面是 MessagePort 是什么和如何工作的一个非常简短的例子：
  
  ```
  // 消息端口是成对创建的。 连接的一对消息端口
  // 被称为通道。
  const channel = new MessageChannel()
  
  // port1 和 port2 之间唯一的不同是你如何使用它们。 消息
  // 发送到port1 将被port2 接收，反之亦然。
  const port1 = channel.port1
  const port2 = channel.port2
  
  // 允许在另一端还没有注册监听器的情况下就通过通道向其发送消息
  // 消息将排队等待，直到一个监听器注册为止。
  port2.postMessage({ answer: 42 })
  
  // 这次我们通过 ipc 向主进程发送 port1 对象。 类似的，
  // 我们也可以发送 MessagePorts 到其他 frames, 或发送到 Web Workers, 等.
  ipcRenderer.postMessage('port', null, [port1])
  ```
  
  ```
  // 在主进程中，我们接收端口对象。
  ipcMain.on('port', (event) => {
  // 当我们在主进程中接收到 MessagePort 对象, 它就成为了
  // MessagePortMain.
  const port = event.ports[0]
  
  // MessagePortMain 使用了 Node.js 风格的事件 API, 而不是
  // web 风格的事件 API. 因此使用 .on('message', ...) 而不是 .onmessage = ...
  port.on('message', (event) => {
    // 收到的数据是： { answer: 42 }
    const data = event.data
  })
  
  // MessagePortMain 阻塞消息直到 .start() 方法被调用
  port.start()
  })
  ```
- 主进程中的MessagePort
	- 在渲染器中， MessagePort 类的行为与它在 web 上的行为完全一样。 但是，主进程不是网页（它没有 Blink 集成），因此它没有 MessagePort 或 MessageChannel 类。 为了在主进程中处理 MessagePorts 并与之交互，Electron 添加了两个新类： MessagePortMain 和 MessageChannelMain。 这些行为 类似于渲染器中 analogous 类。
	- MessagePort 对象可以在渲染器或主 进程中创建，并使用 ipcRenderer.postMessage 和 WebContents.postMessage 方法互相传递。 请注意，通常的 IPC 方法，例如 send 和 invoke 不能用来传输 MessagePort, 只有 postMessage 方法可以传输 MessagePort。
	- 通过主进程传递 MessagePort，就可以连接两个可能无法通信的页面 (例如，由于同源限制) 。
- close事件
	- Electron在 MessagePort 添加了一个在Web上本不存在的功能，以使MessagePort更加好用。 这个功能就是 close 事件, 在通道的另一端关闭时会触发该事件。 端口也可以通过垃圾回收而隐式关闭。
	- 在渲染进程中，你可以通过将事件分配给port.onclose 或调用 port.addEventListener('close', ...) 来监听 close 事件。 在主进程中，你可以通过调用 port.on('close', ...) 来监听 close 事件。
- ### 2.5.1.  建立MessageChannel
- 在这个示例中，主进程设置了一个MessageChannel，然后将每个端口发送给不同的渲染进程。 这样可以让渲染进程彼此之间发送消息，而无需使用主进程作为中转。
  
  ```
  const { BrowserWindow, app, MessageChannelMain } = require('electron')
  
  app.whenReady().then(async () => {
  // 创建窗口
  const mainWindow = new BrowserWindow({
    show: false,
    webPreferences: {
      contextIsolation: false,
      preload: 'preloadMain.js'
    }
  })
  
  const secondaryWindow = new BrowserWindow({
    show: false,
    webPreferences: {
      contextIsolation: false,
      preload: 'preloadSecondary.js'
    }
  })
  
  // 建立通道
  const { port1, port2 } = new MessageChannelMain()
  
  // webContents准备就绪后，使用postMessage向每个webContents发送一个端口。
  mainWindow.once('ready-to-show', () => {
    mainWindow.webContents.postMessage('port', null, [port1])
  })
  
  secondaryWindow.once('ready-to-show', () => {
    secondaryWindow.webContents.postMessage('port', null, [port2])
  })
  })
  ```
- 接下来，在你的预加载脚本中通过IPC接收端口，并设置相应的监听器。
  
  ```
  const { ipcRenderer } = require('electron')
  
  ipcRenderer.on('port', e => {
  // 接收到端口，使其全局可用。
  window.electronMessagePort = e.ports[0]
  
  window.electronMessagePort.onmessage = messageEvent => {
    // 处理消息
  }
  })
  ```
- 在这个示例中，messagePort 直接绑定到了 window 对象上。 更好的方法是使用 contextIsolation，并为每个预期的消息设置特定的 contextBridge 调用， 但为了示例简洁，这里没有这样做。 你可以在本页面下方的 直接在上下文隔离页面的主进程和主世界之间进行通信部分找到一个上下文隔离的示例。
- 这意味着 window.electronMessagePort 在全局范围内可用，你可以在应用程序的任何地方调用postMessage 方法，以便向另一个渲染进程发送消息。
  
  ```
  // elsewhere in your code to send a message to the other renderers message handler
  window.electronMessagePort.postMessage('ping')
  ```
- ### 2.5.2.  Worker进程
- 在这个示例中，你的应用程序有一个作为隐藏窗口存在的 Worker 进程。 你希望应用程序页面能够直接与 Worker 进程通信，而不需要通过主进程进行中继，以避免性能开销。
  
  ```
  const { BrowserWindow, app, ipcMain, MessageChannelMain } = require('electron')
  
  app.whenReady().then(async () => {
  // Worker 进程是一个隐藏的 BrowserWindow
  // 它具有访问完整的Blink上下文（包括例如 canvas、音频、fetch()等）的权限
  const worker = new BrowserWindow({
    show: false,
    webPreferences: { nodeIntegration: true }
  })
  await worker.loadFile('worker.html')
  
  // main window 将发送内容给 worker process 同时通过 MessagePort 接收返回值
  const mainWindow = new BrowserWindow({
    webPreferences: { nodeIntegration: true }
  })
  mainWindow.loadFile('app.html')
  
  // 在这里我们不能使用 ipcMain.handle() , 因为回复需要传输
  // MessagePort.
  // 监听从顶级 frame 发来的消息
  mainWindow.webContents.mainFrame.ipc.on('request-worker-channel', (event) => {
    // 建立新通道  ...
    const { port1, port2 } = new MessageChannelMain()
    // ... 将其中一个端口发送给 Worker ...
    worker.webContents.postMessage('new-client', null, [port1])
    // ... 将另一个端口发送给主窗口
    event.senderFrame.postMessage('provide-worker-channel', null, [port2])
    // 现在主窗口和工作进程可以直接相互通信，无需经过主进程！
  })
  })
  ```
  
  ```
  <script>
  const { ipcRenderer } = require('electron')
  
  const doWork = (input) => {
    // 一些对CPU要求较高的任务
    return input * 2
  }
  
  // 我们可能会得到多个 clients, 比如有多个 windows,
  // 或者假如 main window 重新加载了.
  ipcRenderer.on('new-client', (event) => {
    const [ port ] = event.ports
    port.onmessage = (event) => {
      // 事件数据可以是任何可序列化的对象 (事件甚至可以
      // 携带其他 MessagePorts 对象!)
      const result = doWork(event.data)
      port.postMessage(result)
    }
  })
  </script>
  ```
  
  ```
  <script>
  const { ipcRenderer } = require('electron')
  
  // 我们请求主进程向我们发送一个通道
  // 以便我们可以用它与 Worker 进程建立通信
  ipcRenderer.send('request-worker-channel')
  
  ipcRenderer.once('provide-worker-channel', (event) => {
  // 一旦收到回复, 我们可以这样做...
  const [ port ] = event.ports
  // ... 注册一个接收结果处理器 ...
  port.onmessage = (event) => {
    console.log('received result:', event.data)
  }
  // ... 并开始发送消息给 work!
  port.postMessage(21)
  })
  </script>
  ```
- ### 2.5.3.  回复流
- Electron的内置IPC方法只支持两种模式：即发即弃(例如， send)，或请求-响应(例如， invoke)。 使用MessageChannels，你可以实现一个“响应流”，其中单个请求可以返回一串数据。
  
  ```
  const makeStreamingRequest = (element, callback) => {
  // MessageChannels 是轻量的
  // 为每个请求创建一个新的 MessageChannel 带来的开销并不大
  const { port1, port2 } = new MessageChannel()
  
  // 我们将端口的一端发送给主进程 ...
  ipcRenderer.postMessage(
    'give-me-a-stream',
    { element, count: 10 },
    [port2]
  )
  
  // ... 保留另一端。 主进程将向其端口发送消息
  // 并在完成后关闭它
  port1.onmessage = (event) => {
    callback(event.data)
  }
  port1.onclose = () => {
    console.log('stream ended')
  }
  }
  
  makeStreamingRequest(42, (data) => {
  console.log('got response data:', data)
  })
  // 我们会看到 "got response data: 42" 出现了10次
  ```
  
  ```
  ipcMain.on('give-me-a-stream', (event, msg) => {
  // 渲染进程向我们发送了一个 MessagePort
  // 并期望得到响应
  const [replyPort] = event.ports
  
  // 在这里，我们同步发送消息
  // 我们也可以将端口存储在某个地方，异步发送消息
  for (let i = 0; i < msg.count; i++) {
    replyPort.postMessage(msg.element)
  }
  
  // 当我们处理完成后，关闭端口以通知另一端
  // 我们不会再发送任何消息 这并不是严格要求的
  // 如果我们没有显式地关闭端口，它最终会被垃圾回收
  // 这也会触发渲染进程中的'close'事件
  replyPort.close()
  })
  ```
- ### 2.5.4.  直接在上下文隔离页面的主进程和主世界之间进行通信
- 当 [context isolation][] 已启用。 IPC 消息从主进程发送到渲染器是发送到隔离的世界，而不是发送到主世界。 有时候你希望不通过隔离的世界，直接向主世界发送消息。
  
  ```
  const { BrowserWindow, app, MessageChannelMain } = require('electron')
  const path = require('node:path')
  
  app.whenReady().then(async () => {
  // Create a BrowserWindow with contextIsolation enabled.
  const bw = new BrowserWindow({
    webPreferences: {
      contextIsolation: true,
      preload: path.join(__dirname, 'preload.js')
    }
  })
  bw.loadURL('index.html')
  
  // We'll be sending one end of this channel to the main world of the
  // context-isolated page.
  const { port1, port2 } = new MessageChannelMain()
  
  // 允许在另一端还没有注册监听器的情况下就通过通道向其发送消息 消息将排队等待，直到有一个监听器注册为止。
  port2.postMessage({ test: 21 })
  
  // 我们也可以接收来自渲染器主进程的消息。
  port2.on('message', (event) => {
    console.log('from renderer main world:', event.data)
  })
  port2.start()
  // 预加载脚本将接收此 IPC 消息并将端口
  // 传输到主进程。
  bw.webContents.postMessage('main-world-port', null, [port1])
  })
  ```
  
  ```
  const { ipcRenderer } = require('electron')
  
  // 在发送端口之前，我们需要等待主窗口准备好接收消息 我们在预加载时创建此 promise ，以此保证
  // 在触发 load 事件之前注册 onload 侦听器。
  const windowLoaded = new Promise(resolve => {
  window.onload = resolve
  })
  
  ipcRenderer.on('main-world-port', async (event) => {
  await windowLoaded
  // 我们使用 window.postMessage 将端口
  // 发送到主进程
  window.postMessage('main-world-port', '*', event.ports)
  })
  ```
  
  ```
  <script>
  window.onmessage = (event) => {
  // event.source === window 意味着消息来自预加载脚本
  // 而不是来自iframe或其他来源
  if (event.source === window && event.data === 'main-world-port') {
    const [ port ] = event.ports
    // 一旦我们有了这个端口，我们就可以直接与主进程通信
    port.onmessage = (event) => {
      console.log('from main process:', event.data)
      port.postMessage(event.data.test * 2)
    }
  }
  }
  </script>
  ```
- # 3.  Electron基础
- ## 3.1.  窗口的生命周期
- ready: app 初始化完成
- dom-ready: 一个窗口中的文本加载完成
- did-finish-load: 导航完成时触发
- closed: 当窗口关闭时触发，此时应删除窗口引用
- window-all-closed: 所有窗口都被关闭时触发
- before-quit: 在关闭窗口之前触发
- will-quit: 在窗口关闭并且应用退出时触发
- quit: 当所有窗口被关闭时触发
  
  ```
  const { app, BrowserWindow } = require('electron')
  
  const createWindow = () => {
    const win = new BrowserWindow({
        width: 800,
        height: 600,
    })
  
    win.loadFile("index.html")
  
    win.webContents.on("dom-ready" , ()=> {
        console.log("dom ready")
    })
  
    win.webContents.on("did-finish-load", ()=> {
        console.log("did finish load")
    })
  
    win.on('closed', ()=> {
        console.log("closed")
    })
  }
  
  app.whenReady().then(() => {
    console.log("ready finish")
    createWindow()
  })
  
  app.on("window-all-closed", () => {
    console.log("all window close")
    app.quit()
  })
  
  app.on("before-quit", () => {
    console.log("before quit")
  })
  
  app.on("will-quit", () => {
    console.log("will quit")
  })
  
  app.on("quit", () => {
    console.log("quit")
  })
  ```
- ## 3.2.  简单项目
- 新建一个html的页面
- 编写一个简单的main.js
  
  ```
  // 1、引入electron模块
  let electron = require('electron');
  
  // 2、创建electron引用
  let app = electron.app;
  
  // 3、创建窗口引用
  let BrowserWindow = electron.BrowserWindow;
  
  // 4、声明要打开的主窗口
  let mainWindow = null;
  
  app.on('ready',() => {
  
    // 设置窗口大小
    mainWindow = new BrowserWindow({width:300, height:200});
  
    // 加载哪个页面
    mainWindow.loadFile('helloworld.html');
  
    // 监听关闭事件,把主窗口设置为null，否则内存占用越来越多
    mainWindow.on('closed',() => {
        mainWindow = null;
    })
  })
  ```
- ## 3.3.  进阶项目
- process进程对象
- 编辑Index.html文件，如下：
  
  ```
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'">
    <link href="./styles.css" rel="stylesheet">
    <title>Process</title>
  </head>
  <body>
    <button id="buttonProcess" >查看Process信息</button>
    <script src="./renderer.js"></script>
  </body>
  </html>
  ```
- 编辑渲染进程renderer.js，定义查看进程信息
  
  ```
  var processBtn=document.getElementById('buttonProcess');
  
  processBtn.onclick=getProcessInfo;
  
  function getProcessInfo() {
  console.log("getCPUUsage=",process.getCPUUsage());
  console.log("arch=",process.arch);
  console.log("platform=",process.platform);
  console.log("env=",process.env);
  }
  ```
- File对象
- 在文件系统中，使用HTML5 File 原生API操作文件
- DOM文件接口为原生文件提供了抽象，以便让用户使用HTML5文件API直接处理原生文件对象。Electron已经向文件对象接口添加了一个path属性，在文件系统中暴露出文件的真实路径
- (1)、在index.html文件中添加文件拖动代码
  
  ```
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'">
    <link href="./styles.css" rel="stylesheet">
    <title>Process</title>
  </head>
  <body>
    <div class="for_file_drag" id="drag_test">
      <h2>File对象</h2>
      <span>往这里拖动文件</span>
    </div>
    <button id="buttonProcess" >查看Process信息</button>
    <script src="./renderer.js"></script>
  </body>
  </html>
  ```
  
  ```
  .for_file_drag {
  width: 100%;
  height: 200px;
  background: pink;
  }
  ```
  
  ```
  const electron = require("electron");
  const fs = require('fs');
  
  const dragWrapper = document.getElementById("drag_test");
  
  dragWrapper.addEventListener('drop',(e) => {
    e.preventDefault();
  
    const files = e.dataTransfer.files;
  
    if(files && files.length>0){
        // 获取文件路径
        const path = files[0].path;
        console.log('path',path);
  
        // 获取文件内容
        const content = fs.readFileSync(path);
        console.log(content.toString());
    }
  })
  
  dragWrapper.addEventListener('dragover',(e)=>{
    e.preventDefault();
  })
  ```
- ## 3.4.  实现主进程与渲染进程之间的通信
- 在主进程内实现一个窗口
  
  ```
  function createWindow() {
  win = new BrowserWindow({
  width: 524,
  height: 656,
  show: false, // 设置为false，不立即显示窗口
  // autoHideMenuBar: true, // 设置为 true 隐藏菜单栏
  frame: false, // 设置为 false 隐藏标题栏和边框
  resizable: false,// 用户是否可以调节窗口尺寸
  maximizable: false,// 窗口是否能手动最大化
  // transparent: true,
  webPreferences: {
  	// nodeIntegration: true
  	contextIsolation: true,
  	preload: path.join(__dirname, 'preload.js')
  }
  });
  // 等待窗口加载完毕后再显示并最大化窗口
  win.once('ready-to-show', () => {
  win.show();
  // win.maximize();
  });
  
  // 添加 CSS 类或 ID 到窗口
  win.webContents.on('did-finish-load', () => {
  win.webContents.insertCSS(`
    .rounded-corner-window {
  	border-radius: 4px; /* 设置圆角的大小 */
  	overflow: hidden; /* 确保内容在圆角边界内显示 */
    }
  `);
  });
  
  // // 设置缩放级别为实际大小 (100%)
  // webFrame.setZoomFactor(1);
  // webFrame.setZoomLevel(0);
  
  // 禁用用户调整缩放级别
  // win.webContents.setVisualZoomLevelLimits(1, 1);
  // win.webContents.setLayoutZoomLevelLimits(0, 0);
  
  // 监听渲染进程发送的消息
  ipcMain.on('login', (event, loginData) => {
  requestNCS(
  	loginConfig.accessToken,
  	loginData,
  	data => {
  		const responseData = JSON.parse(data);
  		console.log(data);
  		if (responseData.access_token) {
  			// 登录成功
  			/**
  			 * 登录成功的步骤：
  			 * 1. 保存用户名和密码
  			 * 2. 保存RefreshToken
  			 */
  			fs.writeFileSync(path.join(__dirname, LOGIN_INFORMATION), JSON.stringify({ "username": loginData.username, "password": loginData.password }), { encoding: 'utf-8' });
  			updateRefreshToken(responseData.refresh_token, responseData.refresh_expires_in);
  			win.webContents.send('login-success', 'success');
  			initTorWindow();
  			win.hide();
  			setTimeout(() => {
  				win.close();
  			}, 2000);
  			// ipcRenderer.send();
  		} else {
  			if (responseData.error === 'generic_authentication_error') {
  				win.webContents.send('login-reject', 'reject');
  			} else {
  				// 处理登录失败情况
  				win.webContents.send('login-failure', 'failure');
  			}
  		}
  	},
  	error => {
  		// 只能登录一个的情况
  		win.webContents.send('login-error', error);
  	}
  )
  })
  
  ipcMain.on('windowsClose', (event, args) => {
  win.close();
  })
  
  if (handle) {
  win.webContents.send('login-reject', 'reject');
  }
  
  // 加载登录界面的 HTML 文件
  win.loadFile(path.join(__dirname, 'index.html'));
  }
  ```
- 在上述的配置中，可以看到一个对应的preload文件，这个文件可以将nodejs内的一些函数直接带入到渲染进程中去。
  
  ```
  // preload.js
  const { contextBridge, ipcRenderer } = require('electron');
  
  // 在预加载脚本中向渲染进程暴露需要使用的 API
  contextBridge.exposeInMainWorld('TorAccessAPI', {
  LoginSuccess: (message) => {
  ipcRenderer.send('login-success', message);
  },
  LoginFailure: (callback) => {
  ipcRenderer.on('login-failure', (event, args) => {
  	callback();
  });
  },
  LoginError: (callback) => {
  ipcRenderer.on('login-error', (event, args) => {
  	callback();
  });
  },
  LoginReject: (callback) => {
  ipcRenderer.on('login-reject', (event, args) => {
  	callback();
  });
  },
  showLog: (message) => {
  ipcRenderer.send('showLog', message);
  },
  windowsClose: () => {
  ipcRenderer.send('windowsClose');
  },
  sendLoginMessageToNCS: (username, password) => {
  const loginMessage = {
  	"client_id": "robot",
  	"grant_type": "password",
  	"client_secret": "dS7RPbSTzqnsYHZ3s4XffdpKJ4TObc5V",
  	"username": username,
  	"password": password,
  }
  ipcRenderer.send('login', loginMessage);
  }
  });
  ```
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>登录</title>
  </head>
  
  <body>
  <form id="loginForm">
  <div class="container" style="-webkit-app-region: drag" id="drag-region">
  	<!-- <div class="error-user-text">错误的用户名 / 电子邮件</div> -->
  	<!-- <div class="error-password-text">密码不能为空</div> -->
  	<div style="position: absolute;width: 48px;height: 48px;top:0px;right:0;-webkit-app-region: no-drag"
  		id="window-close">
  		<img id="close-picture" src="./loginresouces/Union.png" sizes="10px" style="margin: 19px;" />
  	</div>
  	<img src="./loginresouces/Titletitle.png" alt="Logo" class="logo">
  	<div class="user-text">用户名 / 电子邮箱</div>
  	<input type="text" style="-webkit-app-region: no-drag" class="input-field" id="username" placeholder="">
  	<div class="password-text">密码</div>
  	<input type="password" style="-webkit-app-region: no-drag" class="input-field password-input" id="password"
  		placeholder="">
  
  	<!-- <input type="checkbox" class="remember-checkbox" id="rememberPassword"> 添加复选框 -->
  	<!-- <div class="remember-text">记住我</label> -->
  	<div class="remember-text">
  		<!-- <form id="remember"></form> -->
  		<p id="information" style="color: red;"></p>
  	</div>
  
  	<button type="submit" style="-webkit-app-region: no-drag">登录</button>
  
  	<div class="background-element"></div>
  	<!-- <div class="forget-password">忘记密码</div> -->
  	<div class="new-user">新用户？</div>
  	<div class="register">去注册</div>
  </div>
  </form>
  
  <script>
  function LoginFail() {
  	// document.getElementById('username').value = '';
  	document.getElementById('password').value = '';
  	document.getElementById('information').innerHTML = `登录失败，请重新输入用户名和密码`;
  }
  
  function LoginError() {
  	// document.getElementById('username').value = '';
  	document.getElementById('password').value = '';
  	document.getElementById('information').innerHTML = `服务异常，请联系管理员`;
  }
  
  function LoginReject() {
  	document.getElementById('username').value = '';
  	document.getElementById('password').value = '';
  	document.getElementById('information').innerHTML = `目前只支持登录一个Tor窗口`;
  	document.getElementById('loginBtn').disabled = true;
  }
  
  window.TorAccessAPI.LoginFailure(LoginFail);
  window.TorAccessAPI.LoginError(LoginError);
  window.TorAccessAPI.LoginReject(LoginReject);
  
  // 在这里编写处理登录的逻辑
  const loginForm = document.getElementById('loginForm');
  loginForm.addEventListener('submit', (event) => {
  	event.preventDefault();
  	const username = document.getElementById('username').value;
  	const password = document.getElementById('password').value;
  	window.TorAccessAPI.sendLoginMessageToNCS(username, password);
  });
  
  const closeBtn = document.getElementById('window-close');
  const closeBtnPic = document.getElementById('close-picture');
  
  closeBtn.addEventListener('click', (event) => {
  	window.TorAccessAPI.windowsClose();
  });
  
  closeBtn.addEventListener('mouseenter', (event) => {
  	closeBtn.classList.add('hovered');
  	closeBtnPic.src = "./loginresouces/Union_hover.png";
  });
  
  closeBtn.addEventListener('mouseleave', (event) => {
  	closeBtn.classList.remove('hovered');
  	closeBtnPic.src = './loginresouces/Union.png';
  });
  </script>
  </body>
  
  </html>
  ```
- 上述就是渲染进程中，需要渲染的内容。
- # 4.  Electron性能优化建议
- ## 4.1.  谨慎地加载模块
- 在向你的应用程序添加一个 Node.js 模块之前，请检查这个模块。 这个模块包含了多少依赖？ 简单的一个a require()声明中包含了什么种类的资源？
- 当考虑一个模块时，建议做以下检查：
- 包含的依赖项的大小
- 需要加载的(require()) 资源
- 你所加载的资源能够执行你关心的操作
- 可以使用命令行上的单个命令生成用于加载模块的 CPU 配置文件和堆内存配置文件 在下面的示例中，我们看一下受欢迎的模块 request。
  
  ```
  node --cpu-prof --heap-prof -e "require('request')"
  ```
- 执行此命令将在您执行的目录下生成一个.cpuprofile和一个.heapprofile 文件。 这两个文件都可以使用 Chrome 开发者工具进行分析，分别使用 Performance 和 Memory 标签 进行分析。
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1704329468219-41c198ca-2b6c-4c8d-8a82-b8c3e7e8138c.png)
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1704329483217-bc4b3eb4-1559-4b13-9568-02bf7804589d.png)
- 在这个例子里，我们看到在作者的机器上加载request 大概用了半秒钟，其中 node-fetch明显占用了极少的内存并且加载用时少于 50ms。
- ## 4.2.  过早地加载和执行代码
- 如果你有非常繁重的初始化操作，请考虑推迟进行。 程序启动立刻查看应用执行的全部工作。 考虑按照用户操作的顺序将它们错开执行，而不是立刻执行所有的操作。
- 在传统的Node.js开发中，我们习惯将所有的require()语句放在代码顶部。 如果你目前正在使用相同的策略and并且使用你不需要立即加载的大型模块编写你的 Electron 应用程序，使用相同的策略并推迟到更适当的时机加载。
- 加载模块是令人吃惊的繁重的操作，尤其是在Windows上。 当你的应用开始，不应该让用户等待当时不需要的操作。
- 这似乎是显而易见的， 但许多应用程序在程序启动后可能会马上完成大量的 工作 - 如检查更新，正在下载稍后流程中使用的内容，或执行大型的磁盘I/O 操作。
- 让我们把Visual Studio Code作为一个例子。 当你打开一个文件，它会立刻展示没有高亮任何代码的内容，优先实现和文本交互的功能。 一旦它完成了这项工作，它将继续让代码高亮。
- 让我们考虑一个示例，并假定您的应用程序正在以架空的.foo形式解析文件 。 为了做到这一点，它依赖同样架空的foo-parserver 模块。 在传统的 Node.js 开发中，你可以写代码热加载依赖：
  
  ```
  const fs = require('node:fs')
  const fooParser = require('foo-parser')
  
  class Parser {
  constructor () {
    this.files = fs.readdirSync('.')
  }
  
  getParsedFiles () {
    return fooParser.parse(this.files)
  }
  }
  
  const parser = new Parser()
  
  module.exports = { parser }
  ```
- 在上面的例子中，我们做了很多工作，一旦文件加载，我们就会立即执行。 我们需要立即获取解析的文件吗？ 或许我们可以晚一点再做这件事，当getParsedFiles() 真正的执行到的时候？
  
  ```
  // "fs" is likely already being loaded, so the `require()` call is cheap
  const fs = require('node:fs')
  
  class Parser {
  async getFiles () {
    // Touch the disk as soon as `getFiles` is called, not sooner.
    // 此外，通过使用异步方法
    // 确保我们不会阻塞其他操作
    this.files = this.files || await fs.promises.readdir('.')
  
    return this.files
  }
  
  async getParsedFiles () {
    // 我们假设 foo-parser 是一个庞大且耗费资源的模块
    // 因此将这个工作推迟到我们真正需要解析文件时再进行
    // 由于require()带有模块缓存
    // require()调用只会有一次开销
    // 后续对getParsedFiles()的调用将会更快
    const fooParser = require('foo-parser')
    const files = await this.getFiles()
  
    return fooParser.parse(files)
  }
  }
  
  // 现在此操作的开销比我们之前的示例要低得多
  const parser = new Parser()
  
  module.exports = { parser }
  ```
- 简而言之，只有当需要的时候才分配资源，而不是在你的应用启动时分配所有。
- ## 4.3.  阻塞主进程
- Electron的主要进程(有时称为“浏览器进程”) 非常特殊：它是与你应用的所有其他进程的父进程，也是和操作系统交互的关键进程。 它负责处理窗口、交互以及应用程序内各个组件之间的通信。 它还包含了UI线程。
- 在任何情况下你都不应阻塞此进程或者运行时间长的用户界面线程。 阻塞UI线程意味着您的整个应用程序将冻结直到主进程准备好继续处理。
- 主进程和其UI线程实质上是你的应用程序内重要操作的控制中心。 当操作系统向你的应用程序报告鼠标点击事件时，它会经过主进程，然后才到达你的窗口。 如果您的窗口呈现黄色平滑动画， 它需要和 GPU 进程进行通信——再次穿越主进程。
- Electron 和 Chromium 谨慎地将大型的磁盘I/O 和 CPU绑定的操作放入新线程，以避免阻塞UI 线程。 你也应该这样做。
- Electron强大的多进程架构随时准备帮助你完成你的长期任务，但其中也包含少量性能陷阱。
- 对于需要长期占用 CPU 的繁重任务，利用worker threads，请考虑将它们移动到 BrowserWindow，或（作为最后手段）生成一个专用进程。
- 尽可能避免使用同步 IPC 和 @electron/remote 模块。 虽然有合法的使用案例，但很容易不知情地阻塞 UI 线程。
- 避免在主进程中使用阻塞 I/O 操作。 简而言之，每当Node.js的核心模块 (如fs 或 child_process) 提供一个同步版本或 异步版本，你更应该使用异步和非阻塞式的变量。
- ## 4.4.  阻塞渲染进程
- 自从 Electron 使用了当前版本的 Chrome，你可以使用Web 平台提供的最新和最优秀的功能来推迟或卸载繁重的操作，以使你的应用保持流畅和迅速的反应。
- 你的应用可能有很多JavaScript在渲染过程中运行。 有个技巧是尽快执行操作，而不占用保持滚动平滑、响应用户输入或60帧/秒动画所需的资源。
- 一般来说，所有用于构建现代浏览器的性能网络应用程序的建议，对于Electron 的渲染器也同样适用。 现在处理你的应用的主要两个方法是对于小的操作使用requestIdleCallback() 而长时间运行的操作使用 Web Workers。
- requestIdleCallback()允许开发者将函数排队为在进程进入空闲期后立刻执行。 它使你能够在不影响用户体验的情况下执行低优先级或后台执行的工作。 想要了解如何使用它的更多信息，请查看MDN上的文档。
- Web Workers是在单独线程上运行代码的一个好方式。 有一些注意事项需要考虑 - 请查阅 Electron 的 多线程文档 和 MDN 的 Web Workers文档。 对于长时间并且大量使用CPU的操作来说它们是一个理想的解析器。
- ## 4.5.  不必要的polyfills
- Electron的一大好处是，你准确地知道哪个引擎将解析你的 JavaScript, HTML和CSS。 如果你重新设计的代码是为整个网页编写的，请确保不会polyfill包含在Electron 中的特性。
- 现在互联网构建网页应用程序时，最老的环境决定了你能够和不能使用的功能。 尽管Electron支持性能良好的 CSS 选择器和动画，但是较早的浏览器可能不支持。 在你可以使用WebGL的场合，你的开发者可能选择了一个资源更加匮乏的解决方案来支持旧机器。
- 当它遇到JavaScript时， 你可能已经包含了工具包库，如DOM选择器 jQuery 或是 如regenerator-runtime支持async/await 的polyfills。
- 基于 JavaScript 的polyfill速度比Electron 中的原生特征要快一些。 不要通过发布你自己的网络平台标准来减慢你的 Electron 应用速度。
- 假定当前版本的 Electron不需要使用polyfills。 如果你有所疑虑，检查 caniuse.com 以确认 是否在你的Electron版本中使用的Chromium版本 已经支持了你需要的特性.
- 此外，仔细检查您使用的三方库。 它们是否真的必要？ 例如，jQuery非常成功，它的许多功能现在都是 标准JavaScript功能设置的 的一部分。
- 如果您正在使用 TypeScript 这样的编译器，检查它的配置并确保你的目标是Electron 支持的最新 ECMAScript 版本。
- ## 4.6.  不必要的或者阻塞的网络请求
- 许多开始使用基于Web的应用程序的Electron用户后来都使用了桌面应用。 作为网页开发者，我们习惯了从各种内容交付网站加载资源。 现在你正在发布一个桌面应用程序，尽可能地尝试“切断连接”，避免让用户等待那些从不改变且可以轻松包含在应用程序中的资源。
- 一个典型的例子是谷歌字体。 许多开发者使用谷歌令人印象深刻的免费字体集，这些字体通过内容交付网络获取。 方法显而易见：包括几行CSS 和谷歌将处理其余部分。
- 构建Electron应用程序时，如果你下载字体并将其包含在应用包中，你的用户将会得到更好的服务。
- 在理想情况下，你的应用程序不需要网络就可以运行。 要达到这个目标，你必须了解你的应用正在下载哪些资源以及这些资源的大小。
- 要做到这一点，请打开开发者工具。 导航到 Network 选项卡，然后检查 Disable cache 选项。 然后重新加载你的页面。 除非你的应用禁止重新加载， 你通常可以在使用开发者工具时点击Cmd + R 或Ctrl + R触发重新加载。
- 开发者工具将仔细记录所有网络请求。 第一步，评估正在下载的所有资源，首先侧重于较大的文件。 其中是否有任何图像、字体或媒体文件不会改变并且可以包含在你的包中？ 如果可以，把它们打包。
- 下一步，启用 Network Throttling。 查找当前读取Online的下拉列表，并选择较慢的速度，例如Fast 3G。 重新加载你的页面并查看你的应用程序是否有等待任何不必要的资源。 在大多数情况下，尽管实际上不需要相关的资源，应用还是会等待网络请求完成。
- 作为一个提示, 从互联网上加载你可能想要更改的而不发送应用程序更新是一个强有力的策略。 为了进一步控制如何加载资源，请考虑使用Service Worker。
- ## 4.7.  打包你的代码
- 正如中已经指出的那样，"加载和运行代码太早", 调用 require() 是一项繁重的操作。 如果你能够这样做，将你的应用程序的代码打包到单个文件中。
- 现代JavaScript开发通常涉及许多文件和模块。 对于使用Electron开发的人来说这是非常好的事情，我们强烈建议你将你的代码打包到单个文件中以确保调用require() 时只在你的应用加载花费一次开销。
- 有许多JavaScript打包的方法可供使用，我们知道我们最好不要因为推荐某一种工具来使得社区不满。 然而，我们的确建议您使用一个能够处理Electron独特的环境的打包程序，它需要处理Node.js 和浏览器两种环境。
- ## 4.8.  当你不需要默认菜单时调用 Menu.setApplicationMenu(null)
- Electron在启动时将设置一个默认菜单，其中包含一些标准条目。 但是你的应用程序或许希望更改默认菜单，这么做有助于提高启动性能。
- 如果你打算构建自己的菜单或使用无帧窗口而不使用原生菜单，你应该尽早告诉 Electron 不要设置默认菜单。
- 在 app.on("ready") 之前调用 Menu.setApplicationMenu(null) 。 这将阻止Electron设置默认菜单。