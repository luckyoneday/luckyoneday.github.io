---
title: "JavaScript 面试知识点汇总"
date: 2021-04-17T11:49:26+08:00
lastmod: 2021-04-17T11:49:26+08:00
draft: false
toc: true
keywords: ["interview"]
description: "JavaScript 面试知识点汇总"
author: "youting"
tags: ["interview"]
---

## 执行上下文

执行上下文就是当前 JavaScript 代码被解析和执行时所处的环境，也叫做执行环境。

当函数被调用时会创建执行上下文，执行上下文创建阶段变量对象、作用域链和 this 指向会被分别指定。

- 创建变量对象：首先初始化函数的参数 `arguments`，提升函数声明和变量声明（变量的声明提前有赖于 `var` 关键字）；
- 创建作用域链：在执行期上下文的创建阶段，作用域链是在变量对象之后创建的。作用域链本身包含变量对象。作用域链用于解析变量。当被要求解析变量时，JavaScript 始终从代码嵌套的最内层开始，如果最内层没有找到变量，就会跳转到上一层父作用域中查找，直到找到该变量。
- 确定 `this` 指向。

变量对象创建：

- 确定函数的 `arguments` 对象；
- 函数提升，`function` 关键字声明的函数；
- `var` 等别的变量声明

执行上下文执行阶段变量对象（VO）会变为活动对象（AO）。随后会完成变量赋值、函数引用以及执行其他代码。

多个作用域的变量对象串联起来组成的链表就是**作用域链**，如果在当前的变量对象里面找不到目标变量/函数，就在上一级作用域的变量对象里面查找。若这时找到了目标变量/函数，则停止查找；若找不到，一直回溯到全局作用域的变量对象里查找，若仍找不到目标变量/函数，停止查找。

## this

指向当前代码运行时所处的上下文环境。**函数执行时确定**。

- 默认指向：严格模式下指向 `undefined`；非严格模式下指向 `window`；
- 隐式指向：调用位置的调用者决定；
- 显式指向：`call` 、 `apply` 和 `bind`
- `new` 操作符绑定

优先级依次升高。

一些特殊情况

- `null` 或者 `undefined` 作为 `this` 指向的对象传入 `call`、`apply` 或者 `bind` 时，会采用默认指向规则。
- 箭头函数没有 `this`，`this` 指向定义时的作用域。

## new、call、apply、bind、Object.create

在 JavaScript 中，构造函数只是一些使用 `new` 操作符时被调用的普通函数。`new` 操作符调用时，会执行下面的操作：

- 创建一个新对象
- `this` 指向这个新对象
- 为这个对象添加属性、方法等
- 判断返回类型，如果不是返回了对象，则返回 this

{{% admonition info "new" %}}

```js
function examNew(fn, ...args) {
  const obj = {};
  obj.__proto__ = fn.prototype;

  const result = fn.call(obj, ...args);
  return result instanceof Object ? result : obj;
}
```

{{% /admonition %}}

{{% admonition info "call" %}}

```js
Function.prototype.wcall = function (thisArg, ...args) {
  const handledThis = thisArg == null ? window : Object(thisArg);

  // 在传入的 thisArg 上添加一个方法并调用，将 this 隐式绑定。
  const tempMethod = "tempMethod";
  handledThis[tempMethod] = this;

  const result = handledThis[tempMethod](...args);

  delete handledThis[tempMethod];
  return result;
};
```

{{% /admonition %}}

{{% admonition info "apply" %}}

区别就是第二个参数是数组，调用时展开

```js
Function.prototype.wapply = function (thisArg, args) {
  const handledThis = thisArg == null ? window : Object(thisArg);
  const tempMethod = "tempMethod";

  handledThis[tempMethod] = this;
  const result = handledThis[tempMethod](...args);

  delete handledThis[tempMethod];
  return result;
};
```

{{% /admonition %}}

{{% admonition info "bind" %}}

```js
Function.prototype.wbind = function (thisArg, ...args) {
  const handledThis = thisArg == null ? window : Object(thisArg);

  const context = this;

  const funcForBind = function () {
    const isNew = this instanceof funcForBind;
    const newThis = isNew ? this : handledThis;
    return context.call(newThis, ...args, ...arguments);
  };

  funcForBind.__proto__ = context.prototype;

  return funcForBind;
};
```

{{% /admonition %}}

{{% admonition info "Object.create" %}}

```js
Object.wcreate = function (obj) {
  const tempFunc = function () {};

  tempFunc.prototype = obj;
  return new tempFunc();
};
```

{{% /admonition %}}

{{% admonition info "Object.create(null) 和 {} 的区别"  %}}

`Object.create(null)` 虽然返回 `{}`，但是不存在任何 `__proto__` 属性。

`Object.create(Object.prototype)` 和 `{}` 类似。

{{% /admonition %}}

[示例](https://codesandbox.io/s/new-call-apply-95nz4)

## 闭包

内层作用域访问它外层函数作用域里的参数/变量/函数时，闭包就产生了。

闭包可以有效的封装内部属性，对功能进行模块化封装。

但是闭包会引用它外部函数作用于中的变量，当外层函数执行完毕退出函数调用栈的时候，函数中的变量可能并不会被 js 引擎的垃圾回收器回收，因而会引发内存泄露。

## 原型链和继承

- 原型是一个对象；
- `prototype` 是函数上面存在的属性，非函数不存在这个属性；
- 一个对象（实例）都有一个属性 `__proto__`，指向它的构造函数的 `prototype` 属性；
- 对象的 `__proto__` 也有自己的 `__proto__`，直到 `__proto__` 为 `null`，即原型链；

{{% admonition info tips %}}

`__proto__` 并不是标准属性，可以使用 `Object.create` 来为新建的对象设置原型。

{{% /admonition %}}

## 事件循环

JavaScript 是单线程、非阻塞的，需要操作 DOM 导致其无法多线程（一个线程添加属性，一个线程删除节点就会起冲突）；非阻塞则基于事件循环实现，可以协调各种事件、用户交互、脚本执行、UI 渲染、网络请求。

主线程遇到一个同步任务时直接添加到执行栈中执行，当遇到一个异步事件时，会将这个异步事件挂起在任务队列（task queue）中。

当主线程执行完当前执行栈中的任务后，会去查看任务队列是否有任务，若有则取出并执行，这个过程就是**事件循环**。

任务队列分为宏任务和微任务：便于对任务进行优先级区分。

### 宏任务

- `script`（整体代码）
- `setTimeout`
- `setInterval`
- `I/O`
- UI 交互
- `setImmediate`

### 微任务

- `Promise.then`
- `MutationObserver`
- `process.nextTick`

在当前执行栈为空时，主线程会查看微任务队列是否有事件存在。存在，依次执行微队列中的事件对应的回调，若微任务在执行过程中产生了新的微任务，则继续执行微任务，直到微任务队列为空；如果不存在，那么就从宏任务队列中取出一个事件并把对应的回调加入当前执行栈，执行宏任务，并执行该宏任务产生的微任务。

## 垃圾回收

所谓的垃圾回收就是找出那些不再继续使用的变量，然后释放出其所占用的内存。垃圾回收会按照固定的时间间隔周期性地执行这一操作。

JavaScript 使用的垃圾回收机制自动管理内存 -- 垃圾回收是不可见的，可以大幅简化程序的内存管理代码，降低程序员的负担，减少因长时间运行而带来的内存泄露问题。但是程序员无法掌控内存，无法强迫进行垃圾回收，无法干预内存处理。

### 引用计数

跟踪记录每个值被引用的次数，被引用加 1，被释放减 1。如果一个值引用次数是 0，就表示这个值不再用到了，因此可以将这块内存释放。但是循环引用会有 bug。

### 标记清除

现代浏览器采用标记清除的方式：当变量进入执行环境时，这个变量被标记为「进入环境」。当变量离开执行环境时标记为「离开环境」。最后垃圾收集器完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间（所谓的环境就是执行环境）。

某个执行环境中的所有代码执行完毕后，该环境被销毁，保存在其中的所有变量和函数定义也随之销毁。全局执行环境只有关闭网页的时候才销毁。

## ts

### interface 和 type 的区别

{{% admonition type="info" title="1.都可以用来描述对象或函数的类型，但语法不同" details="true" %}}

```ts
interface SetPoint {
  (x: number, y: number): void;
}

type SetPoint = (x: number, y: number) => void;
```

{{% /admonition %}}

{{% admonition type="info" title="2.type 类型别名还可以用于其他类型" details="true" %}}

```ts
//number
type MyNumber = number;

//dom
let div = document.createElement("div");
type MyDiv = typeof div;
```

{{% /admonition %}}

{{% admonition type="info" title="3.extends 语法不同" details="true" %}}

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

{{% admonition type="info" title="4.interface 可以定义多次，并会合并多次，但 type 不可以" details="true" %}}

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

{{% admonition type="info" title="5.type 能使用 in 关键字生成映射类型，但 interface 不行" details="true" %}}

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

{{% admonition type="info" title="6.默认导出方式不同" details="true" %}}

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
