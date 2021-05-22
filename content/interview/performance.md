---
title: "前端性能相关"
date: 2021-04-17T12:09:26+08:00
lastmod: 2021-04-17T12:09:26+08:00
draft: false
toc: false
keywords: ["interview"]
description: "前端性能相关"
tags: ["interview"]
author: "youting"
---

- [回流和重绘](../html/#如何减少)
- [css 优化](../css/#css-优化)

- 减少 http 请求、使用 http2
- 使用服务端渲染
- 静态资源使用 CDN

## 打包方向 tree shaking 和 minify

## 性能指标方向: fcp 和 fp

首先是一些性能指标：

- TTFB(Time To First Byte): 返回第一个字节数据所用的时间
- FP(First Paint): 首次任何像素对用户可见的时间 -- 首次绘制
- FCP(First Contentful Paint): 首次内容对用户可见的时间 -- 首次内容绘制
- TTI(Time To Interact): 页面变为可交互的时间 -- 可交互

服务端渲染的优势：

- 省略了客户端二次请求数据的网络传输开销
- 服务端的网络环境要优于客户端，内部服务器之间通信路径也更短
- fcp 更快、有利于 SEO

## 前端性能监控

- 白屏时间：输入网址，到页面开始显示内容的时间。
- 首屏时间：输入网址，到页面完全渲染的时间。
