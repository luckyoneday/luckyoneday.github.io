---
title: "手写代码"
date: 2021-04-17T12:09:37+08:00
lastmod: 2021-04-17T12:09:37+08:00
draft: false
toc: true
keywords: ["interview"]
description: "手写代码"
tags: ["interview"]
author: "youting"
---

## `Promise`

- promise 有三个状态 `pending`、`fulFilled` 和 `rejected`，默认状态为 `pending`；只能从 `pending` 到 `rejected`, 或者从 `pending` 到 `fulfilled`，状态一旦确认，就不会再改变.
- `new Promise` 时，需要传递一个 `executor()` 执行器，并立即执行；`executor` 接受两个参数，分别是 `resolve` 和 `reject`。
- promise 有一个 `value` 保存成功状态的值，可以是 `undefined/thenable/promise`；有一个 `reason` 保存失败状态的值。
- promise 必须有一个 `then`方法，`then` 接收两个参数，分别是 promise 成功的回调 `onFulfilled`，和 promise 失败的回调 `onRejected`。
- `then` 会返回一个新的 promise，并且将返回值透传，即值穿透。

<!-- <br /> -->

```js
const PENDING = "PENDING";
const FULFILLED = "FULFILLED";
const REJECTED = "REJECTED";

class WPromise {
  constructor(executor) {
    this.status = PENDING;
    this.value = undefined;
    this.reason = undefined;

    this.callbacks = [];

    try {
      executor(this._resolve.bind(this), this._reject.bind(this));
    } catch (e) {
      this._reject(e);
    }
  }

  _resolve(value) {
    if (value instanceof WPromise) {
      value.then(this._resolve.bind(this), this._reject.bind(this));
      return;
    }
    if (this.status === PENDING) {
      this.status = FULFILLED;
      this.value = value;
      this.callbacks.forEach((cb) => {
        this._handle(cb);
      });
    }
  }

  static resolve(value) {
    // 判断是否是thenable对象
    if (
      value instanceof WPromise ||
      (typeof value === "object" && "then" in value)
    ) {
      return value;
    }

    return new WPromise((resolve) => resolve(value));
  }

  _reject(reason) {
    if (reason instanceof WPromise) {
      reason.then(this._resolve.bind(this), this._reject.bind(this));
      return;
    }
    if (this.status === PENDING) {
      this.status = REJECTED;
      this.reason = reason;
      this.callbacks.forEach((cb) => {
        this._handle(cb);
      });
    }
  }

  _handle(cb) {
    const { onFailed, onFulfilled, nextReject, nextResolve } = cb;
    if (this.status === PENDING) {
      this.callbacks.push(cb);
      return;
    }

    if (this.status === FULFILLED) {
      const nextValue =
        typeof onFulfilled === "function"
          ? onFulfilled(this.value)
          : this.value;

      nextResolve(nextValue);
      return;
    }
    if (this.status === REJECTED) {
      const nextReason =
        typeof onFailed === "function" ? onFailed(this.reason) : this.reason;

      nextReject(nextReason);
    }
  }

  then(onFulfilled, onFailed) {
    // 值穿透
    return new WPromise((nextResolve, nextReject) => {
      this._handle({
        nextReject,
        nextResolve,
        onFailed,
        onFulfilled,
      });
    });
  }

  static all(arr) {
    return new WPromise((resolve, reject) => {
      const resultArr = [];
      let len = 0;

      Array.from(arr).forEach((item, idx) => {
        WPromise.resolve(item).then((data) => {
          resultArr[idx] = data;
          len++;

          if (len === arr.length) {
            resolve(resultArr);
          }
        }, reject);
      });
    });
  }

  static race(arr) {
    return new WPromise((resolve, reject) => {
      Array.from(arr).forEach((item) => {
        WPromise.resolve(item).then(resolve, reject);
      });
    });
  }
}
```

## `compose` 和 `pipe`

`compose` 是从右往左依次执行，`pipe` 是从左往右。

```js
function compose() {
  const args = [].slice.call(arguments);
  return function (initial) {
    return args.reduceRight((prev, cur) => {
      return cur(prev);
    }, initial);
  };
}

function pipe() {
  const args = [].slice.call(arguments);
  return function (initial) {
    return args.reduce((prev, cur) => {
      return cur(prev);
    }, initial);
  };
}
```

## `memoize`

将上次的运行结果进行缓存，下次调用时如果遇到相同的参数，直接使用缓存中的数据。

```js
function memoize(func) {
  const cache = {};

  return function (key) {
    if (!cache[key]) {
      cache[key] = func.apply(this, arguments);
    }

    return cache[key];
  };
}
```

## `curry`

```js
function primaryCurry(fn, ...args) {
  return function (...args2) {
    var newArgs = args.concat(args2);
    return fn.apply(this, newArgs);
  };
}

function curry(fn, length) {
  const len = length || fn.length;

  return function () {
    const args = [].slice.call(arguments);

    const curryArgs = [fn].concat(args);
    if (args.length < len) {
      return curry(primaryCurry.apply(this, curryArgs), len - args.length);
    } else {
      return fn.apply(this, args);
    }
  };
}
```

实现以下效果：

```js
add(1)(2)(3) == 6;
add(1, 2, 3)(4) == 10;
add(1)(2)(3)(4)(5) == 15;

const add = function (...outer) {
  const argsArr = [...outer];

  const _addArgs = (...args) => {
    argsArr.push(...args);
    return _addArgs;
  };

  _addArgs.toString = () => {
    return argsArr.reduce((prev, cur) => {
      return prev + cur;
    });
  };

  return _addArgs;
};
```

## `debounce`

触发多次事件，最后一次执行事件处理函数。

```js
function debounce(fn, timeout) {
  let timer = null;

  return function () {
    const context = this;

    clearTimeout(timer);

    timer = setTimeout(() => {
      clearTimeout(timer);
      fn.apply(context, arguments);
    }, timeout);
  };
}
```

## `throttle`

隔一段时间执行一次事件处理函数。

```js
// 时间戳实现：触发立即执行，停止触发立马停止执行。
function throttle(fn, timeout) {
  let oldTime = 0;
  return function () {
    const context = this;
    const now = new Date();
    if (now - oldTime >= timeout) {
      oldTime = now;
      fn.apply(context, arguments);
    }
  };
}

// 定时器实现：不会立马触发执行，停止触发后也不会立即停止。
function throttle(fn, timeout) {
  let timer = null;
  return function () {
    const context = this;
    if (timer) return;
    timer = setTimeout(() => {
      fn.apply(context, arguments);
      clearTimeout(timer);
      timer = null;
    }, timeout);
  };
}

// 根据 leading 和 tailing 判断是否要首尾触发，默认首次不触发，最后一次触发
function throttle(fn, timeout, options = {}) {
  let timer = null;
  let previous = options.leading ? 0 : new Date();

  return function (...args) {
    let now = new Date();

    // 下次触发 func 剩余的时间
    const remaining = timeout - (now - previous);
    const context = this;

    // 首次触发时 remaining 会是负
    if (remaining <= 0) {
      if (timer) {
        clearTimeout(timer);
        timer = null;
      }
      previous = now;
      fn.apply(context, args);
    } else if (!timer && options.tailing !== false) {
      timer = setTimeout(() => {
        // 修改过去的时间
        previous = new Date();
        clearTimeout(timer);
        timer = null;
        fn.apply(context, args);
      }, remaining);
    }
  };
}
```

## 深拷贝&浅拷贝

针对引用类型。

- 浅拷贝：只复制一层对象，当对象的属性是引用类型时，实质复制的是其引用，当引用值发生改变时，也会跟着改变；
- 深拷贝：深拷贝是另外申请了一块内存，内容和原来一样，更改原对象，拷贝对象不会发生改变；

### 浅拷贝

```js
// for...in 遍历
function shallowCopy(obj) {
  let result = {};
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      result[key] = obj[key];
    }
  }

  return result;
}

// 扩展运算符
let newObj = { ...obj };
let newArr = [...oldArr];

// Object.assign
newObj = Object.assign({}, obj);

// 数组 slice
newArr = oldArr.slice();
```

### 深拷贝

```js
// JSON.parse/JSON.stringify
JSON.parse(JSON.stringify(obj));

// 递归实现深拷贝
function deepClone(obj) {
  const result = Array.isArray(obj) ? [] : {};
  const isObj = (obj) => obj && typeof obj === "object";
  if (!isObj(obj)) return {};

  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      if (isObj(obj[key])) {
        result[key] = deepClone(obj[key]);
      } else {
        result[key] = obj[key];
      }
    }
  }
  return result;
}
```

## codeSandbox

- [`promise`](https://codesandbox.io/s/promise-g8447)
- [`compose` 和 `pipe`](https://codesandbox.io/s/compose-pipe-isq72)
- [`memoize`](https://codesandbox.io/s/memo-v8sc1)
- [`curry`](https://codesandbox.io/s/curry-73q2c)
- [`debounce`](https://codesandbox.io/s/debounce-xpgn2)
- [`throttle`](https://codesandbox.io/s/throttle-5xj94)
- [`clone`](https://codesandbox.io/s/copy-deep-shallow-i4fl9)
