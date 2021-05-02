---
title: "这是一篇翻译"
date: 2020-01-04T11:33:06+08:00
lastmod: 2020-01-04T11:33:06+08:00
draft: false
toc: false
keywords: ["daily"]
description: "是一篇翻译"
author: "youting"
tags: ["React", "翻译"]
---

是因为我在数组的 map 中使用了 hooks 触发了 eslint 检查，然后蓝教授扔了这篇文章给我。

太长不看版：有 hooks 的函数组件使用 JSX 或者`React.createElement`来渲染组件，而不是直接函数调用。

> 标题： Don't call a React function component
>
> 原文链接： https://kentcdodds.com/blog/dont-call-a-react-function-component

作者说他看到 [Taranveer Bains](https://github.com/kentcdodds/ama/issues/763) 在他的 [AMA](https://github.com/kentcdodds/ama/issues/763) 提出的很棒的 issue：

> 我遇到了个问题，当我在函数组件中使用了 hooks，并且将 JSX 传递给`Array.prototype.map`的回调函数并返回，触发了这样一个报错：`React Error: Rendered fewer hooks than expected`

大概这样写能复现：

```js
import React from "react";
function Counter() {
  const [count, setCount] = React.useState(0);
  const increment = () => setCount((c) => c + 1);
  return <button onClick={increment}>{count}</button>;
}
function App() {
  const [items, setItems] = React.useState([]);
  const addItem = () => setItems((i) => [...i, { id: i.length }]);
  return (
    <div>
      <button onClick={addItem}>Add Item</button>
      <div>{items.map(Counter)}</div>
    </div>
  );
}
```

这是怎么肥事呢，首先，解决方法是这样写：

```
- <div>{items.map(Counter)}</div>
+ <div>{items.map(i => <Counter key={i.id} />)}</div>
```

> 如果你认为这个跟`key` props 有关的话，那先告诉你是没有的。不过`key`还是很重要的，可以通过作者的这篇文章了解一下`key`的作用：[Understanding React's key prop](https://kentcdodds.com/blog/understanding-reacts-key-prop)

下面这样的写法也会触发这个错误：

```js
function Example() {
  const [count, setCount] = React.useState(0);
  let otherState;
  if (count > 0) {
    React.useEffect(() => {
      console.log("count", count);
    });
  }
  const increment = () => setCount((c) => c + 1);
  return <button onClick={increment}>{count}</button>;
}
```

关键是因为`Example`组件在条件语句中调用了 hook ，违反了 [hooks 规则](https://reactjs.org/docs/hooks-rules.html)，这也是 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) 包有 `rules-of-hooks` 规则的原因。也可以在 [React 文档](https://reactjs.org/docs/hooks-rules.html#explanation)中知道更多关于这个规则的内容，但是其实一句话就可以解决这个问题：保证对于给定的组件，hooks 的调用次数始终保持一致。

但是在刚开始的例子里，我们并没有在条件语句中调用 hooks 不是吗？那为什么这样调用会有问题呢？

让我们稍微重写下我们的例子：

```js
function Counter() {
  const [count, setCount] = React.useState(0);
  const increment = () => setCount((c) => c + 1);
  return <button onClick={increment}>{count}</button>;
}
function App() {
  const [items, setItems] = React.useState([]);
  const addItem = () => setItems((i) => [...i, { id: i.length }]);
  return (
    <div>
      <button onClick={addItem}>Add Item</button>
      <div>
        {/** 就是这里跟上面不一样 */}
        {items.map(() => {
          return Counter();
        })}
      </div>
    </div>
  );
}
```

你会注意到其实我们就是在一个函数中调用了另一个函数，所以可以直接改写成内联的：

```js
function App() {
  const [items, setItems] = React.useState([]);
  const addItem = () => setItems((i) => [...i, { id: i.length }]);
  return (
    <div>
      <button onClick={addItem}>Add Item</button>
      <div>
        {/** 还是这里跟上面不一样 */}
        {items.map(() => {
          const [count, setCount] = React.useState(0);
          const increment = () => setCount((c) => c + 1);
          return <button onClick={increment}>{count}</button>;
        })}
      </div>
    </div>
  );
}
```

我们只是简单的重构了下，并没有改变任何逻辑，注意到问题在哪里了吗？让我们复习一下上面说过的：我们需要保证对于**一个给定的组件**，hooks 调用的次数总是一致的。

基于我们的重构，这里调用了`useState`这个 hook 的“给定的组件”不是`App`和`Counter`，而是仅有`App`。这依赖我们调用`Counter`函数组件的方式。当下这种写法它只是个函数，而不是个组件。React 并不知道我们在 JSX 中调用函数和直接内联的区别，因此它无法将任何内容与`Counter`函数关联，因为它并未像一个组件一样被调用。

这就是为什么我们有的时候需要用 JSX (或者`React.createElement`)而不是简单调用函数来渲染组件，因为通过 JSX (或者`React.createElement`)可以使所有的 hooks 注册到 React 创建的实例上。

所以在这种情况下**不要直接调用函数，而是应该使用 JSX (或者`React.createElement`)渲染组件**。

顺便需要注意一下，有的时候直接调用函数也是阔以的：

```js
function Counter() {
  const [count, setCount] = React.useState(0);
  const increment = () => setCount((c) => c + 1);
  return <button onClick={increment}>{count}</button>;
}
function App() {
  return (
    <div>
      <div>Here is a counter:</div>
      {Counter()}
    </div>
  );
}
```

但是这个时候`Counter`内的 hooks 是与`App`组件实例相关联的，因为并没有`Counter`的组件实例。所以虽然这样写可以 work，但是不是按照我们期待的方式 work 的。所以还是正常走“渲染”流程吧。
