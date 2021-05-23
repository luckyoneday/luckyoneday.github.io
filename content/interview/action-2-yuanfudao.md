---
title: "second"
date: 2021-05-23T08:24:12+08:00
lastmod: 2021-05-23T08:24:12+08:00
draft: false
toc: true
keywords: ["interview"]
description: "猿辅导一面"
tags: ["interview"]
author: "youting"
---

### 1. 介绍项目

讲一下项目难点，以及怎么解决。

### 2. setState 是同步还是异步

### 3. 讲一下 React Hooks

### 4. 看一下输出

```js
var a = {
  f1: function () {
    console.log(this);
  },
  f2: () => console.log(this),
};
a.f1();
a.f2();
var c1 = new a.f1();
var c2 = new a.f2();
// 怎么判断 c1 是 a.f1 的实例 instanceof
// instanceof c1 做了什么
// for in 能不能遍历到原型上的属性，缺少什么属性
```

### 5. 依次放开 Error，看一下输出

```js
console.log(1);

// throw new Error('e1')

setTimeout(() => {
  console.log(2);
  // throw new Error('e2')
}, 500);

new Promise((resolve, reject) => {
  console.log(3);
  // throw new Error('e3')
  resolve();
})
  .then((res) => {
    console.log(4);
    // throw new Error("e4");
  })
  .catch((err) => {
    console.log(5);
  });

new Promise((resolve, reject) => {
  resolve();
})
  .then((res) => {
    console.log(8);
  })
  .catch((err) => {
    console.log(9);
  });

console.log(11);
```

### 6. CORS

跨域怎么发送 cookie

https://a.b.com 和 https://c.b.com 怎么共享 cookie

### 7. 二叉树深度遍历

```js
        1

2                3

            4
// 输出 ['1->2', '1->3->4']

// 构造函数
function Node(val) {
  this.val = val;
  this.left = null;
  this.right = null;
}
```
