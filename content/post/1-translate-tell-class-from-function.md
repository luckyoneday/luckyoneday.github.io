---
title: "React 怎么区分类组件和函数组件"
date: 2020-03-08T21:03:31+08:00
lastmod: 2020-03-08T21:03:31+08:00
draft: false
keywords: ["React"]
description: "how does react tell a class from a function 翻译"
tags: ["React", "翻译"]
author: "youting"
---

> 原文标题: How Does React Tell a Class from a Function?
>
> 原文链接: https://overreacted.io/how-does-react-tell-a-class-from-a-function/

<!--more-->

## 太长不看版：

React 会顺着原型链检查`isReactComponent`这个属性：

```js
// React 内部
class Component {}
Component.prototype.isReactComponent = {};

// 检查机制
class Greeting extends Component {}
console.log(Greeting.prototype.isReactComponent);
```

存在就是 class，不存在就是 function。

## 正文从这里开始

先来看下面的一个`Greeting`组件，这是一个函数组件：

```js
function Greeting() {
  return <p>Hello</p>;
}
```

React 也支持定义类组件：

```js
class Greeting extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}
```

（在[最近的 Hooks](https://reactjs.org/docs/hooks-intro.html)出现之前，类组件是唯一可以使用`state`之类属性的方式。）

当我们想要渲染`<Greeting />`，我们并不需要在意它是怎么定义的：

```js
// 不论是类组件还是函数组件
<Greeting />
```

但是 React 会关心他们的不同！

如果`Greeting`是一个函数，React 需要去调用这个函数：

```js
// 我们的代码
function Greeting() {
  return <p>Hello</p>;
}

// React内部
const result = Greeting(props); // <p>Hello</p>
```

但是如果`Greeting`是一个类，React 需要用`new`操作符来创造它的一个实例并且接下来会调用它的`render`方法：

```js
// 我们的代码
class Greeting extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}
// React内部
const instance = new Greeting(props); // Greeting {}
const result = instance.render(); // <p>Hello</p>
```

两种情况中，React 的目的都是得到要渲染的节点（上例中是`<p>Hello</p>`）。但是更具体的步骤取决于`Greeting`是怎么定义的。

**所以 React 是怎么知道定义的组件是 class 还是 function 呢？**

和[上一篇博客](https://overreacted.io/why-do-we-write-super-props/)一样，**我们并不需要了解这个知识点一样可以用 React 进行开发。**这篇文章更多的是在讲关于 JavaScript 的知识点而不是 React。

（这篇文章适合对 React 内部如何工作好奇的人阅读。你是这样的人吗？那让我们一起来深入学习呀~~）

**这是一个很长的旅途，让我们系好安全带~这篇文章没有多少关于 React 的信息，但是我们会深入一些概念比如`new`，`this`，`class`，箭头函数，`prototype`，`__proto__`，`instanceof`，以及他们在 JavaScript 中如何一起生效。幸运的是，我们在使用 React 的时候不需要知道这么多。**

（如果只是想知道答案，可以直接到文章末尾。）

---

首先，我们需要理解函数和类是不一样的，这是很重要的。注意我们在调用类的时候需要使用`new`操作符：

```js
// 如果 Greeting 是一个函数
const result = Greeting(props); // <p>Hello</p>

// 如果 Greeting 是一个类
const instance = new Greeting(props); // Greeting {}
const result = instance.render(); // <p>Hello</p>
```

让我们看一下`new`操作符在 JavaScript 中的作用。

---

在之前，JavaScript 中并没有 class。然而，我们可以使用普通函数表达类似类的模式。**具体来说，我们可以通过在调用函数之前添加`new`，使任何函数像类构造函数：**

```js
// 只是一个函数
function Person(name) {
  this.name = name;
}

var fred = new Person("Fred"); // Person {name: 'Fred'}
var george = Person("George"); // 不起作用
```

我们现在依然可以这样写！可以在 devTools 中试一下~

当我们不用`new`调用`Person('Fred')`时，函数内部的`this`会指向全局和无用的东西（例如`window`或`undefined`）。所以我们的代码会垮掉或者执行一些奇怪的操作比如设置了`window.name`。

通过使用`new`操作符调用，我们大致传达了这样的意思：“Hey JavaScript，我知道`Person`只是一个函数，但是让我们假装他是一个类构造函数。**创建一个空的对象`{}`并且将`Person`内的`this`指向这个对象，这样我可以将一些属性赋值给它比如`this.name`。然后将这个新对象返回给我。**”

这就是`new`操作符做的事情。

```js
var fred = new Person("Fred");
```

`new`操作符也使我们放在`Person.prototype`上面的属性或方法可以被`fred`对象访问到：

```js
function Person(name) {
  this.name = name;
}
Person.prototype.sayHi = function () {
  alert("Hi, I am " + this.name);
};

var fred = new Person("Fred");
fred.sayHi();
```

这是 JavaScript 增加 class 语法前人们模仿 class 语法的方式。

---

所以`new`操作符已经在 JavaScript 中存在了一段时间。相比之下，class 语法是最近的语法，允许我们使用更接近我们意图的方式重写这段代码：

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  sayHi() {
    alert("Hi, I am " + this.name);
  }
}

let fred = new Person("Fred");
fred.sayHi();
```

语言和 API 设计中*捕获开发者的意图*是非常重要的。

如果我们写了一个函数，JavaScript 不能猜出这个函数是会像`alert()`一样被调用，还是类似于`new Person()`的构造函数被调用。忘记使用`new`调用`Person`这样的函数会导致很奇怪的行为。

**使用 class 语法大致就是在对 JavaScript 说：“这不仅仅是个函数--这是一个类并且它拥有构造函数。”**如果我们在调用的时候忘记使用`new`操作符，JavaScript 会报错哒：

```js
let fred = new Person("Fred");
// 如果 Person 是一个函数，正常工作
// 如果 Person 是一个类：也会正常起作用

let george = Person("George"); // 我们忘记了 `new`
// 如果 Person 是一个类构造函数的函数： 奇怪的行为
// 如果 Person 是一个类： 直接失败
```

这会帮助我们提早捕获错误，而不是等到奇怪的 bug 出现，例如`this.name`被处理成`window.name`而不是`george.name`。

然而，这意味着 React 需要在调用类前使用`new`操作符。不能被作为平常的函数调用，因为 JavaScript 会把这种情况视为错误！

```js
class Counter extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}

// React 不能这样
const Instance = Counter(props);
```

这意味着有麻烦啦！

---

在我们学习 React 怎么处理这种情况之前，我们需要知道大多数人用 React 都会使用像 Babel 这样的编译工具，以便于旧版本浏览器能识别像 class 这样的新语法。所以我们的设计中需要考虑编译。

在 Babel 的早期版本，是可以不用`new`来调用类的。然而，这点已经被修好了--通过生成一些额外的代码：

```js
function Person(name) {
  // 一个简化的Babel输出：
  if (!(this instanceof Person)) {
    throw new TypeError("Cannot call a class as a function");
  }

  // 我们的代码：
  this.name = name;
}

new Person("Fred"); // Okay
Person("George"); // 'Cannot call a class as a function'
```

我们之前在打包完的代码里可能见到过这种代码。这是`_classCallCheck`函数做的所有的事情。（我们可以通过选择“松散模式”而不进行检查来减小打包完后的代码体积，但是这样做可能会使转换代码的过程变的复杂。）

---

到现在为止，我们应该粗略地理解了调用函数用`new`和不用`new`的区别：

|            | `new Person()`         | `Person()`                      |
| ---------- | ---------------------- | ------------------------------- |
| `class`    | `this`是`Person`的实例 | `TypeError`                     |
| `function` | `this`是`Person`的实例 | `this`指向`window`或`undefined` |

这就是 React 需要正确调用组件的原因。**如果组件定义为一个类组件，那么 React 需要用`new`来调用它。**

所以 React 可以检查出组件是不是类组件吗？

不是那么简单的！尽管[JavaScript 可以区分是 class 还是函数](https://stackoverflow.com/questions/29093396/how-do-you-check-the-difference-between-an-ecmascript-6-class-and-function)，但是依然不适用于被 Babel 处理过的情况。对于浏览器来说，他们都是单纯的函数。

---

所以，React 可以在每次都用`new`来调用函数吗？不幸的是，这并不是每次都奏效。

对于普通的函数，用`new`来调用他们会返回一个对象实例。作为构造函数来说这样做是没问题的（比如上面的`Person`），但是对于函数组件来说就很奇怪：

```js
function Greeting() {
  // 我们不会想在这里返回一个 `this` 的实例
  return <p>Hello</p>;
}
```

但是这样还可以忍，这里有别的两个原因会杜绝我们的这个想法。

---

不能总使用`new调用的`第一个原因是原生的箭头函数（不是被 Babel 编译过的），使用`new`调用会报错：

```js
const Greeting = () => <p>Hello</p>;
new Greeting(); // Greeting 不是构造函数
```

这个行为是有意的，并且遵循了箭头函数的设计。箭头函数的一个特点就是没有他自己的`this`值--相反的，`this`指向的是最近的外层函数：

```js
class Friends extends React.Component {
  render() {
    const friends = this.props.friends;
    return friends.map((friend) => (
      // `this`是指向`render`方法的
      <Friend size={this.props.size} name={friend.name} key={friend.id} />
    ));
  }
}
```

所以**箭头函数没有自己的`this`。**这意味着它作为构造函数是完全无用的。

```js
const Person = (name) => {
  // 这是无用的
  this.name = name;
};
```

**JavScript 不允许使用`new`来调用箭头函数。**如果我们这样做了，就了一个错误，所以我们需要避免这样做。这和 JavaScript 不允许没有`new`调用类组件是一样的。

这是一个很好的规定但是它会把我们的计划打乱了。React 不能用`new`来调用每个函数因为在箭头函数中会报错的。我们可以尝试通过缺少`prototype`来检测箭头函数，而不是直接使用`new`来调用：

```js
(() => {}).prototype(
  // undefined
  function () {}
).prototype; // {constructor: f}
```

但是这个对于 Babel 编译出的函数[不起作用](https://github.com/facebook/react/issues/4599#issuecomment-136562930)。这可能也不是什么大问题，但是还有另一个原因让这种方式不成立。

---

我们不能总是用`new`调用函数的另一个原因是 React 需要支持返回字符串或者别的原生类型的组件。

```js
function Greeting() {
  return "Hello";
}

Greeting(); // 'Hello'
new Greeting(); // Greeting {}
```

这里又需要来解决[`new`操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new)引起的奇怪行为。与我们之前看到的一样，`new`告诉 JavaScript 引擎去创建一个新的对象，让这个函数的`this`指向这个对象，并且之后将`new`后的结果返回。

然而，JavaScript 也允许使用`new`调用的函数返回别的对象来覆盖使用`new`来调用的结果。如果我们想复用这个实例时，这是一个有用的模式：

```js
var zeroVector = null;

function Vector(x, y) {
  if (x === 0 && y === 0) {
    if (zeroVector !== null) {
      // 复用同样的实例
      return zeroVector;
    }
    zeroVector = this;
  }
  this.x = x;
  this.y = y;
}

var a = new Vector(1, 1);
var b = new Vector(0, 0);
var c = new Vector(0, 0); // b === c
```

然而，`new`操作符会在函数返回值不是对象时忽略它的返回值。意思就是说如果返回了一个字符串或者数字，就跟没写`return`是一样一样的。

```js
function Answer() {
  return 42;
}
Answer(); // 42
new Answer(); // Answer()
```

使用`new`调用函数时，无法从返回原始值（如字符串或数字）的函数中读取返回值，所以如果 React 始终通过`new`来调用函数，是不能支持返回字符串的组件的。

以上的种种问题导致我们需要妥协。

---

到现在我们学习了什么呢？React 需要使用`new`来调用类，对于寻常的函数或者箭头函数则需要直接调用而不能使用`new`。并且也没有可靠的方式可以区分他们。

**如果我们不能解决一般的情况，那我们可以解决一些更具体的吗？**

当定义一个类组件时，我们都会扩展`React.Component`以便于可以使用内置的方法例如`this.setState()`。**相比于检测所有的类，我们不能只检查`React.Component`的子组件吗？**

剧透：这正是 React 所采取的方式。

---

或许，常用来检查`Greeting`是不是 React 组件的方法是检查`Greeting.prototype instanceof React.Component`是否为`true`：

```js
class A {}
class B extends A {}

console.log(B.prototype instanceof A); // true
```

我知道你在想什么。这里发生了什么呢？为了理解这里，我们需要知道 JavaScript 原型。

我们都对“原型链”的概念很熟悉。JavaScript 的每个对象都有一个“原型”。当我们写下`fred.sayHi()`但是`fred`并没有一个`sayHi`的属性时，JavaScript 就会在`fred`的原型上寻找`sayHi`属性。如果还是找不到，就会沿着原型链寻找--`fred`的原型的原型，以此类推。

**令人困惑的是，类和函数的`prototype`属性并没有指向相应的原型**，我不是在开玩笑。

```js
function Person() {}

console.log(Person.prototype); // Not Person's prototype
console.log(Person.__proto__); // Person's prototype
```

所以“原型链”更像是`__proto__.__proto__.__proto__`而不是`prototype.prototype.prototype`。

那函数或类的`prototype`属性是什么呢？**它是使用`new`创建的类或者函数的对象的`__proto__`**

```js
function Person(name) {
  this.name = name;
}
Person.prototype.sayHi = function () {
  alert("Hi, I am" + this.name);
};

var fred = new Person("Fred"); // 将 `fred.__proto__`指向`Person.prototype`
```

`__proto__`链就是 JavaScript 寻找属性的地方：

```js
fred.sayHi();
// 1、fred有一个叫做sayHi的属性吗？没有。
// 2、fred.__proto__上有叫做sayHi属性吗？是的，调用的就是这个！

fred.toString();
// 1、fred有叫做toString的属性吗？没有。
// 2、fred.__proto__有叫做toString的属性吗？没有。
// 3、fred.__proto__.__proto__有叫做toString的属性吗？是的，调用的就是这个！
```

通常，除非是要调试有关于原型链的代码，否则很少会触及到`__proto__`属性。如果想在`fred.__proto__`上增加一些功能，最好是加在`Person.prototype`上面。至少这就是当初设计的方式。

`__proto__`属性甚至都不应该被浏览器暴露出来，因为原型链被视为语言内部的概念。但是一些浏览器增加了`__proto__`属性，渐渐地被勉强标准化（但赞成使用`Object.getPrototypeOf()`）。

**到现在我依然觉得一个叫做`prototype`的属性并没有指向该值的原型是很奇怪的。**（例如，`fred.prototype`是 undefined 因为`fred`不是一个函数。）就个人而言，我认为这是即便经验丰富的开发者都会误解 JavaScript 的原型的主要原因。

---

这是很长的一篇文章，我们已经到了 80%，让我们继续！

我们知道当我们调用`obj.foo`时，JavaScript 会在`obj.__proto__`，`obj.__proto__.__proto__.`中寻找。

使用类，我们并不能直接接触到这种机制，但是`extends`也可以在原型链中起到很好的作用。这是 React 的类实例能调用`setState`等方法的原因：

```js
class Greeting extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}

let c = new Greeting();
console.log(c.__proto__); // Greeting.prototype
console.log(c.__proto__.__proto__); // React.Component.prototype
console.log(c.__proto__.__proto__.__proto__); // Object.prototype

c.render(); // 在c.__proto__处找到（Greeting.prototype）
c.setState(); // 在c.__proto__.__proto__处找到（React.Component.prototype）
c.toString(); // 在c.__proto__.__proto__.__proto__处找到（Object.prototype）
```

换句话说，**当我们使用了类，它的实例的`__proto__`链就是这个类继承结构的一个“镜像”。**

```js
// `extends`链
Greeting
   -> React.Component
       -> Object（隐式）

// `__proto__`链
new Greeting()
    -> Greeting.prototype
        -> React.Component.prototype
            -> Object.prototype
```

双链！

---

既然`__proto__`链反映了 class 继承结构，我们可以通过从`Greeting.prototype`来检查`Greeting`是否从`React.Component`继承而来，然后再沿着它的`__proto__`链：

```js
// `__proto__`链
new Greeting()
    -> Greeting.prototype // 从这里开始
        -> React.Component.prototype // 找到了
            -> Object.prototype
```

更方便的方法，可以使用`X instanceof Y`来进行这种搜索，它会沿着`x.__proto__`链寻找`Y.prototype`。

通常的，它可以用来检查某个对象是不是某个类的实例：

```js
let greeting = new Greeting();

console.log(greeting instanceof Greeting); // true
// greeting (从这里开始)
//     .__proto__ -> Greeting.prototype（找到啦！）
//         .__proto_ -> React.Component.prototype
//             .__proto__ -> Object.prototype

console.log(greeting instanceof React.Component); // true
// greeting (从这里开始)
//     .__proto__ -> Greeting.prototype
//         .__proto_ -> React.Component.prototype（找到啦！）
//             .__proto__ -> Object.prototype

console.log(greeting instanceof Object); // true
// greeting (从这里开始)
//     .__proto__ -> Greeting.prototype
//         .__proto_ -> React.Component.prototype
//             .__proto__ -> Object.prototype（找到啦！）

console.log(greeting instanceof Banana); // false
// greeting (从这里开始)
//     .__proto__ -> Greeting.prototype
//         .__proto_ -> React.Component.prototype
//             .__proto__ -> Object.prototype（没有找到）
```

它也可以用来检查类是否继承自另一个类：

```js
console.log(Greeting.prototype instanceof React.Component);
// greeting
//     .__proto__ -> Greeting.prototype（从这里开始）
//         .__proto_ -> React.Component.prototype（找到啦）
//             .__proto__ -> Object.prototype
```

这个检查就可以帮助我们确定一个对象是类还是普通函数。

---

但这并不是 React 所做的。

需要注意的一点是`instanceof`操作符在项目中有多个 React 副本时失效，我们当前检查的组件会继承自另一个`React.Component`的 React 副本。在一个项目中混合使用多个 React 版本是不好的，原因有好多，但是从历史上看我们尽可能避免这种写法。（使用 Hooks 的话，我们[可能需要](https://github.com/facebook/react/issues/13991)删除一些副本。）

另一种启发是检查在原型上是不是有`render`方法。但是，当时并[不清楚](https://github.com/facebook/react/issues/4599#issuecomment-129714112)组件的 API 会如何发展。每一次的检查都会有代价，所以我们不能增加很多个检查。如果`render`作为实例的方法存在，例如使用类属性语法，这个检查也不会起作用。

所以，React 为基础组件[增加](https://github.com/facebook/react/pull/4663)了一个特殊标志。React 会检查这个标志的存在，这就是它判断一个组件是不是 React 组件类的方法。

起初，标志是在基础的`React.Component`类上：

```js
// React 内部
class Component {}
Component.isReactClass = {};

// 我们可以这样检查
class Greeting extends Component {}
console.log(Greeting.isReactClass); // Yes
```

然而，一些我们想要定位的 class 实例[没有](https://github.com/scala-js/scala-js/issues/1900)复制静态属性（或者手动设置了`__proto__`），所以标志会丢失。

这是 React 将这个标志[移动到](https://github.com/facebook/react/pull/5021)`React.Component.prototype`上去的原因。

```js
// React内部
class Component {}
Component.prototype.isReactComponent = {};

// 我们可以这样检查
class Greeting extends Component {}
console.log(Greeting.prototype.isReactComponent); // true
```

**这才是 React 所使用的方式。**

可能会疑惑为什么这是一个对象而不是一个布尔值。其实在实践中无所谓什么类型，但是 Jest 的早期版本（Jest 的 Good^TM 之前）的`自动锁定功能？？？`默认是开启的，会忽略原生属性，[会破坏检查](https://github.com/facebook/react/pull/4663)。

`isReactComponent`的检查直到今天还[在 React 中](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L297-L300)使用。

如果不是继承自`React.Component`，React 不会在原型中找到`isReactComponent`属性，也就不会将组件视为一个类组件。现在我们知道了为什么关于`不能像调用函数一样调用类`的[最推荐的回答](https://stackoverflow.com/questions/38481857/getting-cannot-call-a-class-as-a-function-in-my-react-project/42680526#42680526)是需要继承`React.Component`。最后，当存在`prototype.render`但是没有`prototype.isReactComponent`属性时会[触发一个 warning](https://github.com/facebook/react/pull/11168)。

---

你可能会觉得这篇文章是假的吧！**最后的解决方案其实是很简单的，但是我花费了巨大的篇幅来解释为什么 React 最终使用了这种解决方案，以及备选方案都有哪些。**

在我的开发过程中，很多库的 API 通常就是这种情况，一个 API 想要易于使用，通常需要考虑语言语义（甚至对于某些语言来说，要涵盖未来的发展方向），运行性能，有没有编译步骤，生态建设和打包的解决方案，早期 warning 和很多别的方面。最后的结果可能不是最优雅的，但是它一定是最实用的。

**如果最后的 API 是成功的，使用者是不会思考实现过程的。**相反他们可以更多的专注于开发自己的应用。

但是如果你是很好奇的，了解是如何起作用的也是很好的。
