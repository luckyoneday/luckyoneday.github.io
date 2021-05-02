---
title: "useEffect 使用指南"
date: 2021-04-03T21:03:31+08:00
lastmod: 2021-04-03T21:03:31+08:00
draft: true
keywords: ["React"]
description: "A Complete Guide to useEffect 翻译"
tags: ["React", "翻译"]
author: "youting"
---

> 原文标题: A Complete Guide to useEffect
>
> 原文链接: https://overreacted.io/a-complete-guide-to-useeffect/

<!--more-->

只有踩过一些坑之后才会发现，[Dan Abramov](https://mobile.twitter.com/dan_abramov) 写的文章是真的很优秀，不过之前理解的不深刻，所以还是需要重新阅读一下经典文章~

从几个问题开始：

- 怎么使用 `useEffect` 代替 `componentDidMount`？

  虽然可以使用 `useEffect(fn, [])` 作为替代，但是其实二者并不一样，`useEffect` 会「捕获」 props 和 state

- 怎么正确在 `useEffect` 中请求数据？什么是 `[]`？
- 需要将函数作为 `useEffect` 依赖项吗？具体如何操作呢？
- 为什么有的时候会触发死循环？
- 为什么有时会在 `useEffect` 内部拿到旧的 `state` 或者 `props`？
