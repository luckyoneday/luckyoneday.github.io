---
title: "debounce + throttle"
date: 2019-12-16T08:26:48+08:00
lastmod: 2019-12-16T08:26:48+08:00
draft: false
keywords: ["daily"]
description: "防抖 + 节流"
author: "youting"
tags: ["React", "JavaScript"]
---

真的是每次用的时候都需要查，一直分不清...尴尬的一批

## 防抖 debounce

在事件触发 ms 之后再执行，在 ms 时间内随意触发事件，都会在这个时间内将之前的方法清除，只会在停止触发 ms 之后才会执行。

```js
const debounce = (fn, ms) => {
  let timer = null;

  return function (...args) {
    const context = this;

    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(context, args);
    }, ms);
  };
};
```

效果可见 [codeSandbox](https://codesandbox.io/s/debounce-v0cs7)

## 节流 throttle

持续触发事件，每隔一段时间只执行一次方法。可以有两种方法来实现这个效果，时间戳或者是定时器：

```js
// 时间戳
const throttle = (fn, ms) => {
  let previous = 0;
  return function (...args) {
    const context = this;
    const now = +new Date();
    if (now - previous > ms) {
      fn.apply(context, args);
      previous = now;
    }
  };
};

// 定时器
const throttleTwo = (fn, ms) => {
  let timer = null;
  return function (...args) {
    const context = this;
    if (!timer) {
      timer = setTimeout(() => {
        clearTimeout(timer);
        timer = null;
        fn.apply(context, args);
      }, ms);
    }
  };
};
```

时间戳会立即执行方法，但是一旦停下触发事件就不会执行方法。定时器第一次执行是在触发事件 ms 之后，停止触发事件之后还会执行一次方法。

如果想要个有头有尾的呢！

```js
const throttleThree = (fn, ms) => {
  let timeout = null;
  let previous = 0;

  return function (...args) {
    let now = +new Date();
    // 下次触发 func 剩余的时间
    const remaining = ms - (now - previous);
    const context = this;

    // 首次触发remaining会是负
    if (remaining <= 0) {
      if (timeout) {
        clearTimeout(timeout);
        timeout = null;
      }
      previous = now;
      fn.apply(context, args);
    } else if (!timeout) {
      // 这里还是上面用定时器的方法，只是会修改一哈过去的时间
      timeout = setTimeout(() => {
        previous = +new Date();
        clearTimeout(timeout);
        timeout = null;
        fn.apply(context, args);
      }, remaining);
    }
  };
};
```

效果可见 [codeSandbox](https://codesandbox.io/s/throttle-rby6f)
