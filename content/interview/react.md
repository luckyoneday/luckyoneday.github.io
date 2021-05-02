---
title: "React 面试知识点汇总"
date: 2021-04-17T11:53:24+08:00
lastmod: 2021-04-17T11:53:24+08:00
draft: false
toc: true
keywords: ["interview"]
description: "React 面试知识点汇总"
author: "youting"
tags: ["interview"]
---

## react fiber 解决了什么问题

React15 以递归方式执行更新，`stack Reconciler`，同步更新无法阻断，无法处理高优先级的任务，会造成卡顿。

React16 将同步更新变为`异步可中断更新`，架构大致分为三层：

- Scheduler（调度器）-- 调度任务优先级
- Reconciler（协调器）-- 找出有变化的组件（虚拟 dom）
- Renderer（渲染器）-- 将变化的组件渲染到页面上

其中 Scheduler 和 Reconciler 会因为有更高优任务需要先更新、或当前帧没有剩余时间等原因被中断。

React Fiber 即 `React` 内部实现的一套`状态更新机制`，`Fiber Reconciler`，支持任务不同`优先级`，可中断与恢复，并且恢复之后可以复用之前的`中间状态`。

## Fiber

Fiber 中包含了三个部分内容：

- 作为静态数据结构的属性
- 链接其他 Fiber 形成 Fiber 树
- 作为动态工作单位的属性（删除/更新等）。

使用双缓存 Fiber 树，在 React 中最多会同时存在两颗 Fiber 树，当前屏幕上显示的 Fiber 树称为 `current Fiber` 树，正在构建的 Fiber 树称为`workInProgress Fiber` 树，并通过 `alternate` 属性链接。

根节点 `fiberRoot` 的 current 指针指向的切换来完成 `current Fiber` 树的切换。

## ReactDOM.render 步骤

- 创建 `fiber`： 创建 `fiberRoot` 和 `rootFiber`，并初始化 `updateQueue`。`fiberRoot` 是整个应用的根节点，`rootFiber` 是要渲染组件所在组件树的根节点。
- 等待创建 `update` 来开启一次更新

## setState 步骤

## diff 算法

为了降低算法复杂度，react 的 diff 会预设 3 个限制：

- 只对同级元素进行 diff。如果一个 DOM 节点在前后两次更新中跨越了层级，那么 React 不会尝试复用他。
- 两个不同类型的元素会产生出不同的树。如果元素由 div 变为 p，React 会销毁 div 及其子孙节点，并新建 p 及其子孙节点。
- 开发者可以通过 key prop 来暗示哪些子元素在不同的渲染下能保持稳定。

通过 `child` 类型（ `object` 、`number` 、`string` 还是 `array` ）来区分子节点是单节点还是数组，分别进行 diff。

### 单节点

- 当 `child !== null` 且 `key` 相同且 `type` 不同时执行 `deleteRemainingChildren` 将 `child` 及其兄弟 `fiber` 都标记删除。

- 当 `child !== null` 且 `key` 不同时仅执行 `deleteChild` 将 `child` 标记删除（后面还有兄弟节点没有被遍历到，可能还可以复用）。

### 多节点

会经历两轮遍历：第一轮处理`更新`的节点，第二轮不属于`更新`的节点。

第一轮：

- `i = 0`， 比较 `child[i]` 和 `oldFiber`，可复用则 `i++`，比较 `child[i]` 和 `oldFiber.sibling`。
- 如果不可复用分为两种：

  - `key` 不同导致不可复用，直接退出第一轮遍历；
  - `key` 相同但 `type` 不同导致不可复用，将 `oldFiber` 标记为 `deletion`，并继续遍历。

- 如果 `child` 遍历完（即 `i === child.length - 1`）或者 `oldFiber` 遍历完（即 `oldFiber.sibling === null`），跳出遍历，第一轮遍历结束。

第二轮根据第一轮遍历结果分多种情况讨论：

- `child` 与 `oldFiber` 同时遍历完，则只需要在第一轮遍历进行组件更新（或标记删除）；
- `child` 没遍历完，`oldFiber` 遍历完，意味着 `child` 的剩余节点都是新增节点，只需要依次标记为 `placement`；
- `child` 遍历完，`oldFiber` 没有遍历完，意味着 `oldFiber` 的剩余节点被删除，只需要依次标记为 `deletion`；
- `child` 与 `oldFiber` 都没有遍历完，意味着有节点在这次更新中改变了位置：

  - 存一个 `oldFiber` 的 `key` 为 key，`oldFiber` 为 value 的 `Map` ；
  - 以最后一个可复用的节点在 `oldFiber` 中的位置 `lastPlacedIndex` 为参考；
  - 当前 `child` 节点的上一次更新的位置位于 `lastPlacedIndex` 左边 ( `child[i].alternate.index < lastPlacedIndex` )，则说明需要移动；

## hooks
