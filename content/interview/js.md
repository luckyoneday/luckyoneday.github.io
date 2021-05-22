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

{{% admonition info "new" %}}

在 JavaScript 中，构造函数只是一些使用 `new` 操作符时被调用的普通函数。`new` 操作符调用时，会执行下面的操作：

- 创建一个新对象
- 将新对象的 `__proto__` 设置为构造函数的 `prototype`
- 为这个对象添加属性、方法等
- 判断返回类型，如果不是返回了对象，则返回 this

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

创建一个新对象，使用现有对象提供新对象的 `__proto__`

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

但是闭包会引用它外部函数作用域中的变量，当外层函数执行完毕退出函数调用栈的时候，函数中的变量可能并不会被 js 引擎的垃圾回收器回收，因而会引发内存泄露。

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

## 继承

继承的优点：继承可以使子类具有父类别的各种属性和方法，而不需要再次编写相同的代码。

{{% admonition "info" "原型链继承"  %}}

子类的原型为父类的一个实例。

```ts
function Parent1() {
  this.name = "parent1";
  this.play = [1, 2, 3];
}

function Child1() {
  this.name = "child1";
}

Child1.prototype = new Parent1();

const s1 = new Child1();
const s2 = new Child1();
s1.play.push(4);
console.log(s1.play, s2.play); // [1, 2, 3, 4]
```

两个实例使用的是同一个原型对象，内存共享。

{{% /admonition %}}

{{% admonition "info" "构造函数继承"  %}}

借用 `call` 调用 `Parent`。

```ts
function Parent2() {
  this.name = "parent1";
  this.play = [1, 2, 3];
}

Parent2.prototype.getName = function () {
  return this.name;
};

function Child2() {
  Parent1.call(this);
  this.name = "child2";
}

const ch = new Child2();
console.log(ch.getName()); // error
```

引用类型属性不会被共享，但是只会继承父类的实例属性和方法，不能继承原型属性或者方法。

{{% /admonition %}}

{{% admonition "info" "组合继承"  %}}

结合上述两种方法。

```ts
function Parent3() {
  this.name = "parent3";
  this.play = [1, 2, 3];
}

Parent3.prototype.getName = function () {
  return this.name;
};

function Child3() {
  Parent3.call(this);
  this.name = "child3";
}

Child3.prototype = new Parent3();
Child3.prototype.constructor = Child3;
```

属性不互相影响，也可以获取到原型上的方法和属性，但是 `Parent3` 执行了两次。

{{% /admonition %}}

{{% admonition "info" "原型式继承"  %}}

主要借助 `Object.create` 实现普通对象的继承。

```ts
const Person4 = {
  name: "parent4",
  play: [1, 2, 3],
  getName: function () {
    return this.name;
  },
};

const p1 = Object.create(Person4);
p1.name = "child4";
p1.play.push(4);
const p2 = Object.create(Person4);
console.log(p1.play, p2.play); // [1, 2, 3, 4]
```

`Object.create` 实现的是浅拷贝，多个实例的引用类型属性指向相同的内存，会被共享。

{{% /admonition %}}

{{% admonition "info" "寄生式继承"  %}}

在上述的继承方式上进行优化，增强浅拷贝能力。

```ts
function clone(original) {
  const clonedItem = Object.create(original);
  clonedItem.getPlay = function () {
    return this.play;
  };
  return clonedItem;
}
```

没啥变化啊这样写。

{{% /admonition %}}

{{% admonition "info" "寄生组合式继承"  %}}

```ts
function create(parent, child) {
  child.prototype = Object.create(parent.prototype);
  child.prototype.constructor = child;
}

function Parent6() {
  this.name = "parent6";
  this.play = [1, 2, 3];
}

Parent6.prototype.getName = function () {
  return this.name;
};

function Child6() {
  Parent1.call(this);
  this.name = "child6";
}

create(Parent6, Child6);

const n1 = new Child6();
const n2 = new Child6();

n1.play.push(4);
console.log(n1.play, n2.play);
console.log(n1.getName());
```

{{% /admonition %}}
