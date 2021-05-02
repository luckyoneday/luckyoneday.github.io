---
title: "Performance Api （上）"
date: 2021-01-27T10:17:53+08:00
lastmod: 2021-01-27T10:17:53+08:00
draft: false
toc: false
keywords: ["daily"]
description: "Performance Api 介绍"
tags: ["Performance"]
author: "youting"
---

Performance API 可以帮助我们全面了解我们的网页，我们可以通过 Performance API 站点获取不同的浏览器，网络中的运行情况数据。

Performance API 是 [High Resolution Time API(高分辨率时间)](https://www.w3.org/TR/hr-time/) 的一部分，结合 Performance Timeline API，Navigation Timing API，User Timing API 和 Resource Timing API 使获取页面性能的能力得到了提高。

> 注：Performance API 还在发展过程中，不停在更新新功能和废弃的功能。所以应该定期查看 MDN 或 W3C 的网站以获取最新版本的 API。

## High Resolution Time

高分辨率时间精度可以达到千分之一毫秒，比 `Date.now()` 的毫秒精度更高一些。高分辨率时间不会随系统时间的变化而改变，因为它是从 UA 创建的全局时钟中获取的。获取当前时间的最基本方法是使用 `performance.now()`。如果范围是当前 window，则返回创建浏览器上下文的时间；如果范围是 worker，则返回创建 worker 的时间。

## Performance Timeline API

Performance Timeline API 是 Performance API 的扩展。 该扩展程序提供了基于特定过滤条件检索性能指标的接口:

- `getEntries()` 返回所有页面可用的性能条目。当在某个页面运行 `performance.getEntries()` 时，会看到控制台返回了一个巨大的数组。大多数条目与页面加载的图像，脚本等资源有关。
- `getEntriesByType(entryType)` 和 `getEntries()` 类似，但是会根据传入的 `entryType` 过滤资源条目。

`entryType` 大致有以下 6 种：

- `frame` 一个实验性功能，可以使开发人员了解一个事件循环中，浏览器完成了多少工作。如果浏览器在一个循环中执行过多的任务，帧速率将下降并且用户体验将变差。
- `resource` 与站点下载的所有资源有关。
- `navigation` 与浏览器导航跳转有关。
- `mark` 自定义标记，可以用来计算代码运行速度。
- `measure` 可以通过 measure 来测量两个 mark 标记之间的差异。
- `paint` 与屏幕上显示的像素有关。

例如：

```js
performance.getEntriesByType("resource")[2];
// connectEnd: 56.274999980814755
// connectStart: 56.274999980814755
// decodedBodySize: 86659
// domainLookupEnd: 56.274999980814755
// domainLookupStart: 56.274999980814755
// duration: 65.14500000048429
// encodedBodySize: 30180
// entryType: "resource"
// fetchStart: 56.274999980814755
// initiatorType: "script"
// name: "https://cdn.jsdelivr.net/npm/jquery@3.2.1/dist/jquery.min.js"
// nextHopProtocol: "h2"
// redirectEnd: 0
// redirectStart: 0
// requestStart: 99.92499998770654
// responseEnd: 121.41999998129904
// responseStart: 119.24999998882413
// secureConnectionStart: 56.274999980814755
// serverTiming: []
// startTime: 56.274999980814755
// transferSize: 30788
// workerStart: 0
```

- `getEntriesByName(entryName)` 通过 `entryName` 过滤资源条目。在上面例子里就可以通过 `performance.getEntriesByName("https://cdn.jsdelivr.net/npm/jquery@3.2.1/dist/jquery.min.js")[0]` 获取同款。

## 手动计算程序时间

例如：

```js
const firstNow = performance.now();
// 模拟大量计算
for (let i = 0; i < 100000; i++) {
  var ii = Math.sqrt(i);
}
const secondNow = performance.now();

const howLongDidOurLoopTake = secondNow - firstNow;
```

上述例子就是通过 `performance.now()` 之间的差值来获取程序运行时间。但是这样写当需要计算的时间比较多时比较难管理，这个时候就可以使用 `mark` 来创建 `entryType` 是 `mark` 的条目，随后根据 `measure` 来创建 `entryType` 是 `measure` 的条目，就可以获取到两个 `mark` 之间的时间差。

例如：

```js
performance.mark("beginSquareRootLoop");
// 模拟大量计算
for (let i = 0; i < 1000000; i++) {
  var ii = Math.sqrt(i);
}
performance.mark("endSquareRootLoop");

// 通过两个 mark 创建新的条目叫做 measureSquareRootLoop
performance.measure(
  "measureSquareRootLoop",
  "beginSquareRootLoop",
  "endSquareRootLoop"
);

performance.getEntriesByName("beginSquareRootLoop")[0];
// detail: null
// duration: 0
// entryType: "mark"
// name: "beginSquareRootLoop"
// startTime: 4298507.545000059

performance.getEntriesByName("measureSquareRootLoop")[0];
// detail: null
// duration: 15.754999942146242
// entryType: "measure"
// name: "measureSquareRootLoop"
// startTime: 4298507.545000059
```

其他类目分析见[下篇](/daily/40-performance-api/)。
