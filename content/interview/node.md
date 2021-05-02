---
title: "Node 面试知识点汇总"
date: 2021-04-17T12:09:10+08:00
lastmod: 2021-04-17T12:09:10+08:00
draft: false
toc: true
keywords: ["interview"]
description: "Node 面试知识点汇总"
tags: ["interview"]
author: "youting"
---

## 事件循环

node 的事件循环阶段顺序大致为：

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

输入数据(Incoming data) -> 轮询阶段(poll) -> 检查阶段(check) -> 关闭事件回调阶段(close callback) -> 定时器检测阶段(timers) -> I/O 事件回调阶段 -> 闲置阶段(Idle, prepare) -> 轮询阶段(poll) -> ...

- 定时器检测阶段(timers)：本阶段执行 timer 的回调，即 `setTimeout`、`setInterval` 里面的回调函数。
- I/O 事件回调阶段(I/O callbacks)：执行上一轮循环中未被执行的一些 I/O 回调。
- 闲置阶段(idle, prepare)：仅系统内部使用。
- 轮询阶段(poll)：检索新的 I/O 事件；执行与 I/O 相关的回调（几乎所有情况下，除了关闭的回调函数，那些由计时器和 `setImmediate` 调度的之外），其余情况 node 将在适当的时候在此阻塞。
- 检查阶段(check)：`setImmediate` 回调函数在这里执行
- 关闭事件回调阶段(close callback)：一些关闭的回调函数，如：`socket.on('close', ...)`。

### `process.nextTick`

`process.nextTick` 是一个独立于 eventLoop 的任务队列。在每一个 eventLoop 阶段完成后会去检查 nextTick 队列，如果里面有任务，会让这部分任务优先于微任务执行。

```js
setImmediate(() => {
  console.log("timeout1");
  Promise.resolve().then(() => console.log("promise resolve"));
  process.nextTick(() => console.log("next tick1"));
});
setImmediate(() => {
  console.log("timeout2");
  process.nextTick(() => console.log("next tick2"));
});
setImmediate(() => console.log("timeout3"));
setImmediate(() => console.log("timeout4"));
```

- 在 node11 之前，因为每一个 eventLoop 阶段完成后会去检查 nextTick 队列，如果里面有任务，会让这部分任务优先于微任务执行，因此上述代码是先进入 check 阶段，执行所有 setImmediate，完成之后执行 nextTick 队列，最后执行微任务队列，因此输出为 `timeout1=>timeout2=>timeout3=>timeout4=>next tick1=>next tick2=>promise resolve`
- 在 node11 之后，process.nextTick 是微任务的一种,因此上述代码是先进入 check 阶段，执行一个 setImmediate 宏任务，然后执行其微任务队列，再执行下一个宏任务及其微任务,因此输出为 `timeout1=>next tick1=>promise resolve=>timeout2=>next tick2=>timeout3=>timeout4`

node11 之后的版本一旦执行完一个宏任务就立即执行对应的微任务，向浏览器看齐。

## 参考资料

- [面试题：说说事件循环机制(满分答案来了)](https://juejin.cn/post/6844904079353708557)
