---
title: "TypeScript泛型"
date: 2020-06-14T21:07:54+08:00
lastmod: 2020-06-14T21:07:54+08:00
draft: false
toc: true
keywords: ["daily"]
description: "TypeScript泛型"
author: "youting"
tags: ["TypeScript"]
---

可以使用泛型来创建高可用的类、类型、接口和函数。

最经典的来自[官方文档](https://www.typescriptlang.org/docs/handbook/generics.html)的例子：

```typescript
function identity(argument: number): number {
  return argument;
}
```

上面的函数返回我们传递给它的参数。 不幸的是，这意味着我们需要为每种返回类型创建一个函数，这并不符合可重用性的标准。为了改进代码，我们可以引入**类型变量**，要声明一个类型变量，我们需要使用 `function identity<T>` 来命名函数：

```typescript
function identity<T>(argument: T): T {
  return argument;
}
```

上面的代码表示 `argument` 和 `identity` 函数返回的类型应该是相同的。

我们可以显式传入类型参数，也可以使用类型推断：

```typescript
identity<string>("Hello world!");
// 或是
identity("Hello world!");
```

## 箭头函数

也可以使用**箭头函数**重写一下：

```typescript
const identity = <T>(argument: T): T => argument;
```

在 `.tsx` 文件中会报错 `JSX 元素“T”没有相应的结束标记。`，要怎么解决这个问题呢，应该复写一下：

```
const identity = <T,>(argument: T): T => argument;
```

## 更常见的例子

基于 Promise 的[Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)功能强大且灵活，在定义类型时，泛型就格外适用。

```js
function request<T>(url: string, options?: RequestInit): Promise<T> {
  return fetch(url, options).then((response) => {
    if (!response.success) {
      return Promise.reject();
    }
    return response.json();
  });
}
```

上面的代码是个简略版，当返回数据没有 body 时[json()](https://developer.mozilla.org/en-US/docs/Web/API/Body/json) 方法可能会报错，需要做更多的一些判断来让上面这段代码更加可靠。

怎么使用呢，如下：

```js
interface User {
  id: number;
  name: string;
  email: string;
}

request<User[]>('https://jsonplaceholder.typicode.com/users')
  .then((users) => {
    console.log(`There are ${users.length} users`);
  }).catch(error) => {
    console.error(error)
  });
```

通过 `request<User[]>('https://jsonplaceholder.typicode.com/users')` 调用可以表明返回的数据是 `User` 数组。

## 泛型接口和类

除了用内置的例如 `Promise` 等接口，我们还可以创建我们自己的：

```js
interface KeyValuePair<K, V> {
  key: K;
  value: V;
}

const keyValuePair: KeyValuePair<string, number> = {
  key: "name",
  value: 15,
};
```

我们也可以创建通用类：

```js
class KeyValuePair <K, V> {
  public key: K;
  public value: V;
  constructor(key: K, value: V) {
    this.key = key;
    this.value = value;
  }
}
const keyValuePair = new KeyValuePair('name', 15);
```
