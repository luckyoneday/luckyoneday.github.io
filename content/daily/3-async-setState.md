---
title: "为什么 setState 是异步的"
date: 2019-11-30T21:46:01+08:00
lastmod: 2019-11-30T21:46:01+08:00
draft: false
keywords: ["daily"]
description: "关于为什么setState是异步的"
author: "youting"
tags: [“React"]
---

> 原 issue 地址: https://github.com/facebook/react/issues/11527#issuecomment-360199710

首先需要承认延迟 reconciliation 来批量更新对性能优化是有益的，从这点来看，`setState`同步更新在很多 case 下效率不高，批量更新是更好的选择。

比如在一个`click`事件中，`Child`和`Parent`组件都调用了`setState`，我们并不希望`Child`组件重渲染两次，相反的希望先对他们进行标记，在退出事件前一次渲染搞定。

那你可能就要问了，为什么我们不能在`setState`调用时立即批量更新`this.state`，而非要等到 reconciliation 结束，这一点并没有特别明晰的答案，但是可以从几个方面来解释一下：

## 保持内部的一致性

即使`state`是同步更新的，但是`props`不会，（在重新渲染父组件前并不能获取到 props）。

React 内部提供的一些对象（`state`、`props`和`refs`）内部都是保持一致的。这代表当使用这些对象时，都需要确保他们引用到的是一个完整的 reconciled tree。

如果`state`按照建议的同步渲染，那么下面的代码是成立的：

```js
console.log(this.state.value); // 0
this.setState({ value: this.state.value + 1 });
console.log(this.state.value); // 1
this.setState({ value: this.state.value + 1 });
console.log(this.state.value); // 2
```

然而如果此时需要将这个状态提升到父组件以保证多个兄弟节点之间可以共享：

```js
-this.setState({ value: this.state.value + 1 });
+this.props.onIncrement(); // 父组件来进行更新
```

需要强调的是，这是 React 应用中经常会发生的一种重构。然而下面的代码将不能正常 work：

```js
console.log(this.props.value); // 0
this.props.onIncrement();
console.log(this.props.value); // 0
this.props.onIncrement();
console.log(this.props.value); // 0
```

这是因为同步模型中，`this.state`会同步更新，`this.props`却不会，在父组件没有重新渲染的情况下，`this.props`是不会变化的。如果想要及时更新`this.props`，那么就要放弃更新的批处理（性能可能会根据情况的不同受到影响）。

so，React 中`this.state`和`this.props`都是在 reconciliation 后更新的，所以上面的场景中都会打印 0，这会使状态提升更安全。

React 模型更愿意保证内部的一致性和状态提升的安全性，而不总是追求代码的简洁性。

## 支持 Concurrent

概念上来说，React 中每个组件都有一个更新队列（update queue）。然后我们就有了讨论是否立即执行更新的可能性，因为我们本应按照顺序更新，但事实并非如此。

最近我们一直在讨论“异步渲染”（async rendering）。那就是 React 会依据不同的调用源，给 `setState()` 分配不同的优先级。调用源包括事件处理、网络请求、动画等。

例如当我们聊天窗口中输入消息，则需要立即更新`TextBox`中的`setState()`，如果在打字过程中接收到一个消息，可能延迟渲染`MessageBubble`保持输入流畅是更好的选择，因为立即渲染`MessageBubble`组件可能阻塞线程，导致输入抖动和延迟。

如果给某些更新分配“低优先级”，那么把它们的渲染分拆为几个毫秒的块，用户也不会注意到。

异步更新并不只关于性能优化，而是 React 组件模型还能做什么的一个根本性转变（fundamental shift）。

假设我们从一个页面导航到到另一个页面，通常我们需要展示一个加载动画。如果导航非常快（1s 之内啥的），闪烁一下加载动画会降低用户体验。更惨的是，如果很多组件都有各种依赖接口，那么会出现很多的在转圈的加载动画，由于 DOM 重排，这在视觉上来看都很不美观，而且会让用户觉得我们的应用程序很慢。

但是这样会不会好点，我们只需要简单的调用`setState()`去渲染一个新的页面，React “在幕后”开始渲染这个新的页面。想象一下，不需要写任何的协调代码，如果这个更新花了比较长的时间，就展示一个加载动画，否则在新页面准备好后，让 React 执行一个无缝的切换。此外，在等待过程中，旧的页面依然可以交互，但是如果花费的时间比较长，就必须展示一个加载动画。

需要注意的是，异步更新`state`是有可能实现这种设想的前提。如果同步更新`state`就没有办法在幕后渲染新的页面，还保持旧的页面可以交互。它们之间独立的状态更新会冲突。
