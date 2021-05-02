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

- 解析 HTML，生成 DOM 树，解析 CSS，生成 CSSOM 树；
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

- [回流与重绘](https://juejin.cn/post/6844903942137053192)
- [你真的了解回流和重绘吗](https://segmentfault.com/a/1190000017329980)
