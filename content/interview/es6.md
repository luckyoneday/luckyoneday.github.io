---
title: "ES6及新特性相关"
date: 2021-04-27T20:43:32+08:00
lastmod: 2021-04-27T20:43:32+08:00
draft: false
toc: true
keywords: ["interview"]
description: "ES6及新特性相关"
tags: ["interview"]
author: "youting"
---

## ES6

### 块级作用域

ES6 之前只有函数作用域和全局作用域。

在花括号 `{}` 内部由 `let` 关键字声明的函数，才是真正的处于块级作用域内部。

`var` 会带来变量提升，声明提升到当前作用域顶部，会造成污染。

### 箭头函数

箭头函数没有 `arguments`，没有 `constructor`，无法被 `new`。

## ES7

### `Array.prototype.includes`

```js
[1, 2, 3, 4, NaN].includes(NaN); // true
[1, 2, 3, 4, NaN].indexOf(NaN) > -1; // false

// 第二个参数是开始的下标
[1, 2, 3, 4, NaN].includes(2, 3); // false
```

### 指数函数中缀表示

```js
2 ** 2; // 等同于 2*2
2 ** 3; // 等同于2*2*2
```

## ES8

### `Object.values` `Object.entries`

```js
const obj = { x: "xxx", y: 1 };
Object.values(obj); // ['xxx', 1]
Object.entries(obj); // [['x', 'xxx'], ['y', 1]]
```

### `String.prototype.padStart` 和 `String.prototype.padEnd`

- 第一个参数为需要达到的长度，若小于当前字符串则返回字符串本身
- 第二个参数为填充使用的字符串，默认为空格。若比需要填充的空格长，则会被截断。

```js
"es8".padStart(2); // 'es8'
"es8".padStart(5); // '  es8'
"es8".padStart(6, "1891"); // '189es8'

"es8".padEnd(14, "coffe"); // 'es8coffecoffec'
"es8".padEnd(7, "9"); // 'es89999'
```

### `Object.getOwnPropertyDescriptors`

```js
const obj = { aaa: "1111" };

Object.getOwnPropertyDescriptors(obj);
{
  aaa: {
    configurable: true;
    enumerable: true;
    value: "1111";
    writable: true;
  }
}
```

### `Async` 和 `Await`

## ES9

### 异步迭代器

`await` 可以和 `for...of` 一起使用。

```js
async function func(array) {
  for await (let i of array) {
    //异步迭代
    someFunc(i);
  }
}
```

### Rest / Spread

ES6 只支持数组，ES9 开始支持对象。

## ES10

### `String.prototype.trimStart` 和 `String.prototype.trimEnd`

去除字符串首尾空格。

### `Array.prototype.flat` 和 `Array.prototype.flatMap`

数组降维度。

```js
let r;
r = ["1", ["8", ["9", ["1"]]]].flat(); //4维数组，默认降维1，变成3维数组
console.log(r); // [ '1', '8', [ '9', ['1'] ] ]

r = ["1", ["8", ["9", ["1"]]]].flat(2); //4维数组，降维2，变成2维数组
console.log(r); // [ '1', '8', '9', ['1'] ]

r = ["1", ["8", ["9", ["1"]]]].flat(Infinity); //4维数组，最多变成1维数组
console.log(r); // [ '1', '8', '9', '1' ]

// 先map再flat
r = ["I love", "coffe 1891"].flatMap((item) => item.split(" "));
console.log(r); //>>[ 'I', 'love', 'coffe', '1891' ]
```

## ES2020

- Optional chaining 可选链操作符：`a?.b?.c`
- Nullish coalescing Operator 空值处理：左侧为 `undefined` 或 `null` 就返回右边 `a ?? {}`
