---
title: "Typescript 和 Redux"
date: 2021-05-20T15:07:04+08:00
lastmod: 2021-05-20T15:07:04+08:00
draft: false
toc: true
keywords: ["interview"]
description: "Typescript 和 Redux 的面试题"
tags: ["interview"]
author: "youting"
---

# typescript

## interface 和 type 的区别

{{% admonition "info" "1.都可以用来描述对象或函数的类型，但语法不同" %}}

```ts
interface SetPoint {
  (x: number, y: number): void;
}

type SetPoint = (x: number, y: number) => void;
```

{{% /admonition %}}

{{% admonition "info" "2.type 类型别名还可以用于其他类型" %}}

```ts
//number
type MyNumber = number;

//dom
let div = document.createElement("div");
type MyDiv = typeof div;
```

{{% /admonition %}}

{{% admonition "info" "3.extends 语法不同" %}}

- interface extends interface
  ```ts
  interface PointX {
    x: number;
  }
  interface Point extends PointX {
    y: number;
  }
  ```
- interface extends type
  ```ts
  type PointX = { x: number };
  interface Point extends PointX {
    y: number;
  }
  ```
- type extends type
  ```ts
  type PointX = { x: number };
  type PointY = { y: number };
  type Point = PointX & PointY;
  ```
- type extends interface
  ```ts
  type PointX = { x: number };
  interface PointY {
    x: number;
  }
  type Point = PointX & PointY;
  ```

{{% /admonition %}}

{{% admonition "info" "4.interface 可以定义多次，并会合并多次，但 type 不可以" %}}

```ts
interface User {
  name: string;
}
interface User {
  password: string;
}

//此时User -> {name: string; password: string}
```

{{% /admonition %}}

{{% admonition "info" "5.type 能使用 in 关键字生成映射类型，但 interface 不行" %}}

```ts
type Status = 200 | 500;

type StatusMap = {
  [key in Status]: string;
};

const test: StatusMap = {
  200: "ok",
  500: "server error",
};

/**
报错 
接口中的计算属性名称必须引用必须引用类型为文本类型或 "unique symbol" 的表达式。
计算属性名的类型必须为 "string"、"number"、"symbol" 或 "any"。
“Status”仅表示类型，但在此处却作为值使用。
**/
//interface StatusMap2 {
//  [key in Status]: string
//}
```

{{% /admonition %}}

{{% admonition "info" "6.默认导出方式不同" %}}

```ts
export default interface Person {
  name: string;
}

//会报错
// export default type Person = {
//   name: string
// }

type Person1 = {
  name: string;
};
export default Person1;
```

{{% /admonition %}}

# Redux

```js
const store = createStore(reducer);
// store 有三个方法： getState、subscribe、dispatch
// 发布订阅机制 -- subscribe订阅一个事件，dispatch派发。
```
