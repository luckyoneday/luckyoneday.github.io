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

### let const

ES6 之前只有函数作用域和全局作用域。

- `var` 会带来变量提升，声明提升到当前作用域顶部，会造成污染。
- 在花括号 `{}` 内部由 `let` 关键字声明的函数，才是真正的处于块级作用域内部。
- 不存在变量提升
- 暂时性死区：只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。
- 不允许重复声明变量。

### 解构

{{% admonition info "数组解构" %}}

- 只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值
- 只有当一个数组成员严格等于 `=== undefined`，解构默认值才会生效  
  `let [x = 1] = [null]; // x: null`
- 惰性求值，能取到就不会走默认值  
  `function f() { return 'aaa' }; let [x = f()] = [1]; // x: 1`
- 默认值可以引用解构赋值的其他变量（从左到右），但该变量必须已经声明。
- 字符串也可以解构，会被转化成类数组解构

{{% /admonition %}}

{{% admonition info "对象解构" %}}

- 数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值
- 对象的解构赋值可以取到继承的属性
- 只有当一个属性严格等于 `=== undefined`，解构默认值才会生效

{{% /admonition %}}

### 箭头函数

- 箭头函数没有 `arguments`，没有 `constructor`，无法被 `new`。
- 函数体内的 `this` 对象，就是**定义时**所在的对象，而不是使用时所在的对象。

### Reflect

### Proxy

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
