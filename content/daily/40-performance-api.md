---
title: "Performance Api （下）"
date: 2021-01-30T20:17:53+08:00
lastmod: 2021-01-30T20:17:53+08:00
draft: false
toc: false
keywords: ["daily"]
description: "Performance Api 介绍"
tags: ["Performance"]
author: "youting"
---

[上篇指路](/daily/39-performance-api/)

## 其他类目分析

### Navigation

导航用于深入了解构建网页的关键步骤。 访问导航数据的方法是： `performance.getEntriesByType("navigation")`。

```js
performance.getEntriesByType("navigation")[0];
// connectEnd: 3.490000031888485
// connectStart: 3.490000031888485
// decodedBodySize: 7846
// domComplete: 180.9699999867007
// domContentLoadedEventEnd: 156.07000002637506
// domContentLoadedEventStart: 143.4649999719113
// domInteractive: 143.31499999389052
// domainLookupEnd: 3.490000031888485
// domainLookupStart: 3.490000031888485
// duration: 180.99000002257526
// encodedBodySize: 7846
// entryType: "navigation"
// fetchStart: 3.490000031888485
// initiatorType: "navigation"
// loadEventEnd: 180.99000002257526
// loadEventStart: 180.99000002257526
// name: "http://localhost:1313/daily/40-performance-api/"
// nextHopProtocol: "http/1.1"
// redirectCount: 0
// redirectEnd: 0
// redirectStart: 0
// requestStart: 11.200000066310167
// responseEnd: 13.340000063180923
// responseStart: 11.520000058226287
// secureConnectionStart: 0
// serverTiming: []
// startTime: 0
// transferSize: 8032
// type: "reload"
// unloadEventEnd: 17.865000059828162
// unloadEventStart: 15.914999996311963
// workerStart: 0
```

[这里](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming)可以看到这些数据的详细解释。 下面是导航时间轴的可视化：

![resource](../../static-img/40-performance-api/resource.png)

### Resource

页面加载资源情况可以通过 `performance.getEntriesByType("resource")` 获取。结果包括图片、脚本以及 css 文件。我们如果想单独获取图片资源可以使用以下方法：

```js
performance
  .getEntriesByType("resource")
  .filter((resource) => resource.initiatorType == "img");
```

静态资源会有很多属性值为 0，是因为我们被 CORS 策略限制了。（这个策略使 resourceTiming API 有很大的局限性），所以静态资源的以下`redirectStart`, `redirectEnd`, `domainLookupStart`, `domainLookupEnd`, `connectStart`, `connectEnd`, `secureConnectionStart`, `requestStart`, 和 `responseStart` 值通常都为 0。

### Paint

paint API 与在窗口上绘制像素的事件有关。例如：

```js
performance.getEntriesByType("paint");
// [{
//   duration: 0
//   entryType: "paint"
//   name: "first-paint"
//   startTime: 2189.879999961704
// },{
//   duration: 0
//   entryType: "paint"
//   name: "first-contentful-paint"
//   startTime: 2189.879999961704
// }]
```

从上面可以看出 paint API 主要有两种，`first-paint` 指的是用户屏幕上出现第一个像素。`first-contentful-paint` 指的是第一次渲染 DOM 中定义的元素。
