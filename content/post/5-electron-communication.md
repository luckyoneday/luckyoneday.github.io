---
title: "Electron 通信"
date: 2021-02-04T14:52:18+08:00
lastmod: 2021-02-04T14:52:18+08:00
draft: false
toc: true
keywords: ["Electron"]
description: "Electron 通信"
tags: ["JavaScript", "Electron"]
category: ["JavaScript", "Electron"]
author: "youting"
---

Electron 中有两种进程：主进程和渲染进程。

## 主进程

一个 electron 应用只有一个主进程，主进程指的是 `package.json` 文件中 `main` 字段指定的文件。主进程通过 `BrowserWindow` 实例来创建窗口，例如：

```js
const { BrowserWindow } = require("electron");
const win = new BrowserWindow({
  width: 800,
  height: 600,
  // ...
});

win.loadUrl("https://baidu.com");
win.loadUrl(`file://${__dirname}/app/index.html`);
// 或者
win.loadFile(`file://${__dirname}/app/index.html`);
```

主进程管理所有窗口极其渲染进程。

## 渲染进程

electron 的每一个 `BrowserWindow` 都在一个独立的渲染进程中渲染页面，当一个 `BrowserWindow` 实例被删除，它对应的渲染进程也会终止运行。每个渲染进程都是相互独立的，只需要关注自己对应的页面。

## 如何通信

那主进程和渲染进程是怎么通信的呢？那就要用到新的模块：`ipcMain` 和 `ipcRenderer`。

### 主进程给渲染进程发送消息

主进程中通过 `BrowserWindow` 对象的 `webContents` 的 [`send`](https://www.electronjs.org/docs/api/web-contents#contentssendchannel-args) 方法来向渲染进程发送消息：

```js
// 主进程中
const { app, BrowserWindow } = require("electron");
let win = null;

app.on("ready", () => {
  win = new BrowserWindow({ width: 800, height: 600 });
  win.loadURL(`file://${__dirname}/index.html`);
  win.webContents.on("did-finish-load", () => {
    win.webContents.send("ping", "whoooooooh!");
  });
});
```

```html
<!-- 在渲染进程中 -->
<html>
  <body>
    <script>
      require("electron").ipcRenderer.on("ping", (event, message) => {
        console.log(message); // 'whoooooooh!'
      });
    </script>
  </body>
</html>
;
```

### 渲染进程给主进程发消息

主进程通过 [`ipcMain`](https://www.electronjs.org/docs/api/ipc-main) 模块的 `on` 方法监听消息。第一个参数是消息标识，第二个参数是回调函数。如 `ipcMain.on(channel, (event, ...args) => {})`，可以通过 `event.sender.send` 向发送对象回传消息。

```js
// 主进程
const { ipcMain } = require("electron");

// 同步消息
ipcMain.on("synchronous-message", (event, arg) => {
  console.log(arg); // "ping"
  event.returnValue = "pong";
});

// 异步消息
ipcMain.on("asynchronous-message", (event, arg) => {
  console.log(arg); // "ping"
  event.sender.send("asynchronous-reply", "pong");
});
```

渲染进程通过 [`ipcRenderer`](https://www.electronjs.org/docs/api/ipc-renderer) 模块的 `on` 方法监听消息。第一个参数是消息标识，第二个参数是回调函数。如 `ipcRenderer.on(channel, (event, ...arg) => {})` 。通过 `send` 或 `sendSync` 方法发送消息，如 `ipcRenderer.send(channel, ...args)`

```js
// 渲染进程的页面
const { ipcRenderer } = require("electron");

// 同步消息会收到 returnValue
console.log(ipcRenderer.sendSync("synchronous-message", "ping")); // "pong"

ipcRenderer.on("asynchronous-reply", (event, arg) => {
  console.log(arg); // "pong"
});

ipcRenderer.send("asynchronous-message", "ping");
```

### 渲染进程给渲染进程发消息

渲染进程也可以通过 [`ipcRenderer`](https://www.electronjs.org/docs/api/ipc-renderer) 模块的 `sendTo` 方法给别的渲染进程发送消息，如 `ipcRenderer.sendTo(webContentsId, channel, ...args)` ，这里的 `webContentsId` 对应的就是另一个渲染进程的 `webContentsId`。

### 其他方法

看两个模块的 api 时会发现还有一些额外的方法，也可以支持进程间的通信。

> ipcRenderer

- `ipcRenderer.once(channel, (event, ...arg) => {})` 监听到消息之后只执行一次，随后就移除掉了。
- `ipcRenderer.invoke(channel, ...args)` 调用后会返回 Promise，针对这个方法，主进程应该使用 `ipcMain.handle()` 方法来响应，如下：

```js
// 渲染进程
ipcRenderer.invoke("some-name", someArgument).then((result) => {
  // ...
});

// 主进程
ipcMain.handle("some-name", async (event, someArgument) => {
  const result = await doSomeWork(someArgument);
  return result;
});
```

`invoke` 和 `send` 区别在哪里呢，如果不需要响应消息，可以使用 `send`。

- `ipcRenderer.removeListener(channel, (event, ...arg) => {})` 从监听器数组中移除监听 `channel` 的指定 `listener`。
- `ipcRenderer.removeAllListeners(channel)` 从监听器数组中移除监听 `channel` 的所有监听器。

> ipcMain

- `ipcMain.once(channel, (event, ...arg) => {})` 监听到消息之后只执行一次，随后就移除掉了。
- `ipcMain.handle(channel, listener)` 响应 `ipcRenderer.invoke(channel, ...args)` 。
- `ipcMain.handleOnce(channel, listener)` 响应 `ipcRenderer.invoke(channel, ...args)` ，也是只执行一次。

- `ipcMain.removeListener(channel, (event, ...arg) => {})` 从监听器数组中移除监听 `channel` 的指定 `listener`。
- `ipcMain.removeAllListeners(channel)` 从监听器数组中移除监听 `channel` 的所有监听器。
- `ipcMain.removeHandler(channel)` 移除名称为 `channel` 的 `handler` 。

## 参考资料

- [Electron 进程通信](https://www.imweb.io/topic/5b13a663d4c96b9b1b4c4e9c)
- [主进程和渲染器进程](https://www.electronjs.org/docs/tutorial/quick-start#main-and-renderer-processes)
- [ipcMain](https://www.electronjs.org/docs/api/ipc-main)
- [ipcRenderer](https://www.electronjs.org/docs/api/ipc-renderer)
