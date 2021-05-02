---
title: "Electron Q&A"
date: 2021-02-04T15:24:54+08:00
lastmod: 2021-02-04T15:24:54+08:00
draft: false
toc: false
keywords: ["daily"]
description: "Electron问答"
tags: ["JavaScript", "Electron"]
author: "youting"
---

- `loadURL` 和 `loadFile` 的区别

`loadURL` 可以加载线上的资源地址和本地的资源，但是 `loadFile` 只能加载本地的文件地址。

- 怎么创建无边框窗口？

无边框窗口是不带外壳（包括窗口边框、工具栏等），只含有网页内容的窗口，可以通过创建 `BrowserWindow` 传入 `options` 来创建：

```js
const { BrowserWindow } = require("electron");
const win = new BrowserWindow({
  width: 800,
  height: 600,
  frame: false, // 无边框窗口
});
```

- 怎么拖拽无边框窗口？

可以给加载的网页中的 dom 结构传入 `-webkit-app-region: drag` 的 css 样式。比如一个弹窗的 `header`：

```css
header {
  -webkit-app-region: drag;
}

header > .close {
  -webkit-app-region: no-drag;
}
```

需要额外将有操作的元素设置为不可拖拽，否则点击不了~
