---
title: "关于监听事件"
date: 2020-03-29T14:53:32+08:00
lastmod: 2020-03-29T14:53:32+08:00
draft: false
toc: false
keywords: ["daily"]
description: "关于事件监听的小注意点"
author: "youting"
tags: ["React", "Event", "JavaScript"]
---

今天解决了一个画板独立的 bug，具体表现为当切换了画板的 tab 时，触发 wheel 事件会导致所有画布一起缩放。

后来查看 `console` 中的 eventListener 发现切换一个 tab 就会在 window 上添加一个事件，原因归结于我在 `window.addEventListener` 的事件上添加了属性：

```js
window.addEventListener("wheel", handleSize, {
  passive: false,
  capture: true,
});
```

但是 `removeEventListener` 上并没有添加相应的属性：

```js
window.removeEventListener("wheel", handleSize);
```

所以就根本没有 remove 掉...

那为什么没有在 remove 的时候加上相应的属性呢，因为起初是这么加的：

```js
window.removeEventListener("wheel", handleSize, {
  passive: false,
  capture: true,
});

// error: 类型“{ passive: boolean; capture: true; }”的参数不能赋给类型“boolean | EventListenerOptions | undefined”的参数。
// 对象文字可以只指定已知属性，并且“passive”不在类型“EventListenerOptions”中。
// 类型“{ passive: boolean; capture: true; }”的参数不能赋给类型“boolean | EventListenerOptions | undefined”的参数。
```

对，没看完报错信息就以为 `{ capture: true }` 加上也没用了...
实际上应该这么写：

```js
window.removeEventListener("wheel", handleSize, {
  capture: true,
});
```

这件事情告诉我们什么呢，不要想当然。官网文档在这里：[`EventTarget.removeEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener)
