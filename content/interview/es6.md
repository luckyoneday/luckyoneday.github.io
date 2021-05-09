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

### 函数

- 默认传参的参数变量是默认声明的，所以不能用 `let` 或 `const` 再次声明。
- `length` 属性可返回**没有指定默认值**的参数个数。
  - `...rest` 参数也不会计入 `length` 属性，`...rest` 需要是最后一个参数。
  - 如果设置了默认值的参数不是尾参数，那么 `length` 属性也不再计入后面的参数了。

<!-- <> -->

```md
(function (a, b, c = 5) {}).length // 2
(function(...args) {}).length // 0
(function (a = 1, b, c) {}).length // 0
```

- 函数的 `name` 属性，返回该函数的函数名。

### 箭头函数

- 箭头函数没有 `arguments`，没有 `constructor`，无法被 `new`。
- 函数体内的 `this` 对象，就是**定义时**所在的对象，而不是使用时所在的对象。
- 不能用 `yield` 命令，无法用作 Generator 函数。

### 数组

- 扩展运算符可以将字符串转化为数组
- 任何定义了 `Iterator` 接口的对象都可以用扩展运算符转化为数组
- 对于那些没有部署 `Iterator` 接口的类似数组的对象 `{ length: 3 }`，扩展运算符无法将其转为真正的数组，但是 `Array.from` 可以
- `Map`、`Set`、`Generator` 使用扩展运算符都可以转化为数组
- `Array.from` 可以把两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）

### 属性遍历

ES6 一共有 5 种方法遍历属性：

- `for...in` 循环遍历对象**自身的和继承的**可枚举属性（不含 Symbol 属性）
- `Object.keys(obj)` 返回一个数组，包括对象**自身的（不含继承的）**所有可枚举属性（不含 Symbol 属性）的键名。
- `Object.getOwnPropertyNames` 返回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。
- `Object.getOwnPropertySymbols` 返回一个数组，包含对象自身的所有 Symbol 属性的键名。
- `Reflect.ownKeys` 返回一个数组，包含对象**自身的（不含继承的）**所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

### Set 和 Map

`Set` 和 `Map` 中 `NaN` 和 `NaN` 相同。`.keys()`、`.values()`、`.entries()` 返回的都是遍历器。

#### `Set`

- 去重：不会重复添加
- 遍历顺序是插入顺序
- WeakSet 结构与 Set 类似，但是成员只能是对象，没有 `size` 属性，不能遍历

#### `Map`

- 对象键只能是字符串，`Map` 的 `key` 可以是对象，当是对象的时候只要内存地址不一样，就视为两个键；需要严格相等，类型不一样也是两个键
- 遍历顺序是插入顺序
- WeakMap 只接受对象作为键名，没有 `size` 属性，不能遍历

### Reflect

- 将 Object 对象的一些明显属于语言内部的方法（比如 `Object.defineProperty`），放到 Reflect 对象上。
- 让 Object 操作都变成函数行为。某些 Object 操作是命令式，比如 `name in obj` 和 `delete obj[name]`，而 `Reflect.has(obj, name)`和 `Reflect.deleteProperty(obj, name)` 让它们变成了函数行为。
- Reflect 对象的方法与 Proxy 对象的方法一一对应。不管 Proxy 怎么修改默认行为，总是可以在 Reflect 上获取默认行为。

### Proxy

用于修改某些操作的默认行为，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。

- `get(target, propKey, receiver)`：拦截对象属性的读取，比如 `proxy.foo` 和 `proxy['foo']`。
- `set(target, propKey, value, receiver)`：拦截对象属性的设置，比如 `proxy.foo = v` 或 `proxy['foo'] = v`，返回一个布尔值。
- `has(target, propKey)`：拦截 `propKey in proxy` 的操作，返回一个布尔值，对 `for...in` 不生效。
- `deleteProperty(target, propKey)`：拦截 `delete proxy[propKey]` 的操作，返回一个布尔值。
- `ownKeys(target)`：拦截 `Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy`)、`Object.keys(proxy)`、`for...in` 循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而 `Object.keys()` 的返回结果仅包括目标对象自身的可遍历属性。
- `getOwnPropertyDescriptor(target, propKey)`：拦截 `Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
- `defineProperty(target, propKey, propDesc)`：拦截 `Object.defineProperty(proxy, propKey, propDesc)`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
- `getPrototypeOf(target)`：拦截 `Object.getPrototypeOf(proxy)`，返回一个对象。
- `setPrototypeOf(target, proto)`：拦截 `Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- `apply(target, object, args)`：拦截 Proxy 实例作为函数调用的操作，比如 `proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。
- `construct(target, args)`：拦截 Proxy 实例作为构造函数调用的操作，比如 `new proxy(...args)`，需要返回一个对象。

### Promise

手写 XHR

```js
const getJSON = function (url) {
  const promise = new Promise(function (resolve, reject) {
    const handler = function () {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };

    const client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();
  });

  return promise;
};
```

### Class

- 构造函数的另一种写法： `class Point {}; typeof Point // "function"`
- 类的所有方法都定义在类的 `prototype` 属性上面，且都是不可枚举的
- 类必须使用 `new` 调用，否则会报错。普通构造函数不用 `new` 也可以调用
- 不存在提升
- 也有 `name` 属性
- 如果在一个方法前，加上 `static` 关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。静态方法可以被子类继承，但是不会被实例继承

### ES6 module

`<script>` 设置 `type="module"` 来引入一个 ES6 模块，会异步加载，不会造成堵塞浏览器，即等到整个页面渲染完，再执行模块脚本，等同于打开了`<script>` 标签的 defer 属性，会按照在页面出现的顺序依次执行。

也可以设置 `async` 属性，但是不会按照在页面出现的顺序执行，而是只要该模块加载完成，就执行该模块。

```html
<script type="module" src="./foo.js" async></script>
```

#### 与 CommonJS 的区别

- CommonJS 主要用于服务端。
- CommonJS 输出的是一个值的拷贝，ES6 输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
- CommonJS 的 `require()` 是同步加载模块，ES6 模块的 import 命令是异步加载，有一个独立的模块依赖的解析阶段。
- ES6 模块的 `import` 命令可以加载 CommonJS 模块，但是**只能整体加载**，不能只加载单一的输出项。

`require` 第一次加载脚本时，会执行整个脚本，然后在内存中生成一个对象：

```js
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
```

之后再用到这个模块时，就会到 `exports` 对象上面取值。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。

#### AMD

- 主要用于浏览器端
- 引用模块的时候，将模块名放在 `[]` 中作为 `reqiure()` 的第一参数，如果定义的模块本身也依赖其他模块,那就需要将它们放在 `[]` 中作为 `define()` 的第一参数
- 依赖前置、提前执行

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

返回指定对象所有**自身属性（非继承属性）**的描述对象。

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
