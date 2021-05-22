---
title: "first"
date: 2021-05-19T15:23:27+08:00
lastmod: 2021-05-19T15:23:27+08:00
draft: false
toc: true
keywords: ["interview"]
description: "伴鱼一面"
tags: ["interview"]
author: "youting"
---

### 1. 输出什么

```js
console.log("1");
async function async1() {
  console.log("2");
  await console.log("3");
  console.log("4");
}

setTimeout(function () {
  console.log("5");
}, 0);

async1();

new Promise(function (resolve) {
  console.log("6");
  resolve();
}).then(function () {
  console.log("7");
});

console.log("8");
```

### 2. 输出什么

```js
let x = 5;
function fn(x) {
  return function (y) {
    console.log(y + ++x);
  };
}

let f = fn(6);
f(7);
console.log(x);
```

### 3.输出什么

```js
var name = "window";
var obj = {
  name: "obj",
  normal() {
    return () => {
      console.log(this.name);
    };
  },
  arrow: () => {
    return function () {
      console.log(this.name);
    };
  },
};

const obj1 = { name: "obj1" };
obj.normal.call(obj1)();
obj.arrow.call(obj1)();
```

### 4.输出什么

```js
obj = { a: 0 };
function fun(obj) {
  obj.a = 222;
  obj = { a: 2 };
  obj.b = 2222222;
}
fun(obj);
console.log(obj);
```

顺便问了下深浅拷贝。

### 5.编程题三选一

1. 奇偶排序：给定⼀个⽆序 Number 数组 ，将数组中奇数升序排序、偶数降序排序输出。注意保持奇数和偶数的位置不变<br />
   例如 输入 [9, 4, 7, 6, 10] 输出 [7, 10, 9, 6, 4]

2. [版本号比较](https://leetcode-cn.com/problems/compare-version-numbers/)
3. [有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

### 6. 手写 new Object.create bind

顺便问了原型链，`bind` 方法为什么绑到 `prototype` 上面。

### 7. 手写发布订阅模式

```js
class Listen {
  ctor() {
    this.handlers = {};
  }
  on() {
    const [key, handle] = Array.from(arguments);
    if (key in this.handlers) {
      this.handlers[key] = [...this.handlers[key], handle];
    } else {
      this.handlers[key] = [handle];
    }
  }
  emit() {
    const [key, ...param] = Array.from(arguments);
    const handlerArr = this.handlers[key];
    handleArr.forEach((item) => item(...param));
  }
}

const instance = new Listen();
instance.on("touch", (e1, e2) => {});
instance.on("touch", (e1, e2, e3) => {});
instance.emit("touch", e1, e2, e3);
```

### 8. react hooks 怎么做到触发页面重新渲染

### 9. setState 做了什么

### 10. setState 是同步的还是异步的

### 11. react 调度是怎么做的

### 12. 介绍项目中的亮点

还问了个 [翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)
