---
title: "HTML面试知识点汇总"
date: 2021-04-17T12:09:18+08:00
lastmod: 2021-04-17T12:09:18+08:00
draft: false
toc: true
keywords: ["interview"]
description: "HTML面试知识点汇总"
tags: ["interview"]
author: "youting"
---

## 浏览器解析渲染机制

浏览器内核指的是浏览器运行的最核心的程序，分为两个部分，一个是渲染引擎，另一个是 JS 引擎。

- 解析 HTML，生成 DOM 树；
- 解析 CSS，生成 CSSOM 树；
- 将 DOM 树和 CSSOM 树结合，生成**渲染树**；
- Layout（回流）：根据生成的渲染树，进行回流，得到节点的几何信息（位置，大小）；
- Painting（重绘）：根据渲染树以及回流得到的几何信息，得到节点的绝对像素；
- Display：将像素发送给 GPU，展示在页面上。

为了构建渲染树，浏览器主要完成以下工作：

- 从 DOM 树的根节点开始遍历每个**可见节点**；
- 对于每个可见节点，找到 CSSOM 树中对应的规则并应用他们；
- 根据每个可见节点以及其对应的样式，组合生成渲染树

不可见节点：

- 一些不会渲染输出的节点，比如 script、meta、link 等；
- 一些通过 css 进行隐藏的节点，比如 `display: none`。（通过 `visibility` 和 `opacity` 隐藏的节点还是会显示在渲染树上的）

### 浏览器在渲染过程中遇到 JS 文件怎么处理

如果遇到 `<script>` 标签就停止渲染，执行 JS 代码。一因为浏览器的 GUI 渲染线程与 JS 引擎线程是互斥的关系，JavaScript 的加载、解析与执行会阻塞 DOM 的构建，也就是说，在构建 DOM 时，HTML 解析器若遇到了 JavaScript，那么它会暂停构建 DOM，将控制权移交给 JavaScript 引擎，等 JavaScript 引擎运行完毕，浏览器再从中断的地方恢复 DOM 构建。

而且一旦引入了 JavaScript，CSSOM 也会阻塞 DOM 的构建，因为 JavaScript 可以更改 CSSOM，如果浏览器尚未完成 CSSOM 的下载和构建，而我们却想在此时运行脚本，那么浏览器将延迟脚本执行和 DOM 构建，直至其完成 CSSOM 的下载和构建。也就是说，在这种情况下，浏览器会先下载和构建 CSSOM，然后再执行 JavaScript，最后再继续构建 DOM。

### defer 和 async 的作用

- 没有 `defer` 和 `async`，浏览器会立即加载并执行指定的脚本，不会等待后续载入的文档元素。
- `async` 属性表示异步执行引入的 JavaScript 文件，与 `defer` 的区别在于，如果已经加载好，就会立即开始执行--无论此刻是 HTML 解析阶段还是 `DOMContentLoaded` 触发之后。但是会阻塞 `load` 事件，即 `async-script` 可能在 `DOMContentLoaded` 触发之前或之后执行，但是一定会在 `load` 触发之前执行。
- `defer` 表示延迟执行引入的 JavaScript 文件，即这段 JavaScript 加载时 HTML 并未停止解析，这两个过程是并行的。整个 document 解析完毕且 `defer-script` 也加载完成之后（这两件事情的顺序无关），会执行所有由 `defer-script` 加载的 JavaScript 代码，然后触发 `DOMContentLoaded` 事件。

`defer` 与相比普通 script，有两点区别：载入 JavaScript 文件时不阻塞 HTML 的解析，执行阶段被放到 HTML 标签解析完成之后。
在加载多个 JS 脚本的时候，`async` 是无顺序的加载，而 `defer` 是有顺序的加载。

### 浏览器在渲染过程中遇到 CSS 文件怎么处理

CSS 文件是有单独的下载线程异步下载的，不会阻塞 DOM 树的解析，但是会阻塞 render 树的形成。也就是会阻塞页面的渲染。

### 为什么操作 DOM 慢

操作 DOM 时，JS 引擎需要通过“桥接接口”来使渲染引擎发生改变，这个操作会比较昂贵。

### 触发的事件

- `readystatechange`：`readyState` 改变时触发
  - `loading`：表示页面仍在加载
  - `interactive`：表示文档已加载和解析，但子资源仍在加载（CSS、image）
  - `complete`：文档和所有子资源都加载完成，等待 load 事件触发
- `DOMContentLoaded`：DOM 树渲染完成，不论子资源是否加载完成
- `load`：所有资源都已加载完成

{{% admonition info "顺序" %}}

readystatechange， loading 状态 -> readystatechange， interactive 状态 ->DOMContentLoaded 事件 -> readystatechange， complete 状态 -> window.onload

{{% /admonition %}}

## 回流和重绘

### 回流

布局引擎会根据各种样式计算每个盒子在页面上的大小和位置，这个计算的阶段就是回流。

回流这一阶段主要是计算节点的位置和几何信息，那么当页面布局和几何信息发生变化的时候，就会触发回流，如：

- 添加或删除可见的 DOM；
- 元素的位置发生变化；
- 元素的尺寸发生变化（包括外边距、内边距、边框大小、高度和宽度等）；
- 内容发生变化，比如文本变化或图片被另一个不同尺寸的图片所替代；
- 页面初次渲染
- 浏览器的窗口尺寸变化（回流是根据视口的大小来计算元素位置和大小的）

和获取一些特定属性的值：`offsetTop`、`offsetLeft`、`offsetWidth`、`offsetHeight`、`scrollTop`、`scrollLeft`、`scrollWidth`、`scrollHeight`、`clientTop`、`clientLeft`、`clientWidth`、`clientHeight`，和 `getComputedStyle` 方法，浏览器为了获取这些值，需要进行即时计算，也会进行回流。

[what forces layout/reflow](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)

### 重绘

当计算好盒模型的位置、大小及其他属性后，浏览器将渲染树上的每个节点转换为屏幕上的实际像素，这个阶段就叫做重绘。

触发回流一定会触发重绘，但是重绘不一定导致回流。还有一些其他行为也会引发重绘：

- 颜色修改
- 文本方向的修改
- 阴影的修改

### 如何减少

- 如果想设定元素的样式，避免逐条修改样式，尽量通过改变元素的 `class` 类名去合并样式；
- 修改 DOM 元素时让其「离线」（`display: none`），修改完后再添加到 DOM 树中；
- 动画尽量使用 `position: fixed/absolute` ，尽可能使元素脱离文档流，从而减少对其他元素的影响；
- 触发 CSS3 硬件加速，可以使用 `transform`、`opacity`、`filters`，这些效果不会引起回流重绘；
- 避免使用 `table` 布局，`table` 中每个元素的大小以及内容的改动，都会导致整个 `table` 的重新计算；

## 参考资料

- [深入浅出浏览器渲染原理](https://github.com/ljianshu/Blog/issues/51)
- [回流与重绘](https://juejin.cn/post/6844903942137053192)
- [你真的了解回流和重绘吗](https://segmentfault.com/a/1190000017329980)
