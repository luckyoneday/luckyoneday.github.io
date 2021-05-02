---
title: "TypeScript索引类型"
date: 2020-06-15T20:32:14+08:00
lastmod: 2020-06-15T20:32:14+08:00
draft: false
keywords: ["daily"]
description: "TypeScript索引类型"
author: "youting"
tags: ["TypeScript"]
---

一般来说我们可以使用联合类型来代表几种类型，如下：

```typescript
const value: string | number;
```

字符串字面量类型也是一种联合类型。一个字符串字面量只能被赋值给特定的字符串值。可以作为一个字符串类型的子类型。

```typescript
type UserRole = "admin" | "moderator" | "author";
```

对于下面这个简单的接口：

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  role: UserRole;
}
```

使用 `keyof` 就可以得到一个字符串字面量联合类型：

```typescript
type UserKeysType = keyof User; // 'id' | 'name' | 'email' | 'role'
```

使用对象属性获取值的类似方式可以获取到类型：

```typescript
type IdType = User["id"]; // number
```

索引类型( `index type` )就是类似上面的方式，通过索引得到的值的类型，可以通过几个例子来感受一下：

## 获取对象属性

我们可以使用更动态的方式来获取到值的类型，比如以下这个例子：

```typescript
const user: User = {
  id: 15,
  name: "John",
  email: "john@smith.com",
  role: "admin",
};

function getProperty<T, K extends keyof T>(object: T, prop: K): T[K] {
  return object[prop];
}

getProperty(user, "id"); // 15
getProperty(user, "id").toLowerCase(); // 类型“number”上不存在属性“toLowerCase”。
```

`extends` 可以用来继承一个类，也可以用来继承一个接口。此时 `K` 就类似上面提到的 `UserKeysType`。所以`T[K]` 指的就是 `User[UserKeysType]` 的类型。

## 对象生成 Map

通常我们会用 `Object.entries` 来对对象进行操作生成 Map:

```typescript
const settings = {
  isModalOpened: true,
  canDelete: false,
  role: "Admin",
};

const settingsMap = new Map(Object.entries(settings));
```

上面的代码虽然是合理的，但是无法验证更具体的类型：

```typescript
settingsMap.get("role").toLowerCase(); // 类型“string | boolean”上不存在属性“toLowerCase”。类型“false”上不存在属性“toLowerCase”。
```

上面的错误表示 `settingsMap.get` 返回的是 `string | boolean` 的联合类型，怎么让他知道 `role` 属性对应的是 `string` 类型呢，可以创建一个从 `Map` 继承来的类型并且提供更具体的类型：

```typescript
interface MapFromObject<T, K extends keyof T> extends Map<K, T[K]> {
  get: <U extends keyof T>(key: U) => T[U];
}

const settingsMap = new Map(Object.entries(settings)) as MapFromObject<
  Settings,
  keyof Settings
>;

settingsMap.get("role").toLowerCase(); // 'admin'
```

## 映射类型

映射类型( `Mapped types` )允许我们从现有类型中额外创建新的类型。一个常见的使用场景是将一个对象的所有属性变为*只读 (read-only)*。

```typescript
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

`Readonly` 类型的作用是将传入的属性变为只读。例如 TypeScript 中是这样定义 [`Object.freeze`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)的：

```typescript
interface ObjectConstructor {
  /**
   * 防止修改现有的属性和值
   * 也防止新增属性
   * @param o 需要被锁的对象
   */
  freeze<T>(o: T): Readonly<T>;
}
```

如我们所见，`Object.freeze` 函数返回使用 Readonly 修饰符映射的对象。

还有一些很多用处的别的内置修饰符，比如 `Pick` 、 `Partial`：

```typescript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

还有条件类型：

```typescript
type WithNumbersInsteadOfStrings<T> = {
  [K in keyof T]: T[K] extends number ? string : T[K];
};

const user: WithNumbersInsteadOfStrings<User> = {
  id: "15", // id也变为了string类型
  name: "John",
  email: "john@smith.com",
  role: "admin",
};
```
