---
title: 你不了解的 undefined
date: 2021-10-01 18:35:56
tags:
---

Undefined 这个概念听起来很简单，不过你知道应该怎样检查 JavaScript 中的变量或属性是否真的存在吗？ 做这件事最好的方法是什么？ 我们如何涵盖所有的边界值？ 要回答这些问题，首先让我们来看看究竟什么是 undefined

### undefined 概述

变量的值被赋予一个类型，JavaScript 中有几个内置的本地类型：

- Undefined
- Null
- Boolean
- String
- Number
- Object
- Reference

首先看第一个，内置的 Undefined 类型只能有一个值，它称为 undefined 。 这是一个原始值，只要声明了变量，就会为其分配此 undefined 值，直到您以编程的手段为其分配不同的值。

此外，每当函数完成执行并返回一个没有给定的值时，它默认返回 undefined 。

```javascript
var foo,
  bar = (function () {
    // .....
  })(),
  baz = (function () {
    var hello;
    return hello;
  })();

typeof foo; // undefined
typeof bar; // undefined
typeof baz; // undefined
```

因此，当声明一个变量但还未赋值时，它将被赋予 undefined 值。 我们还应该注意的是：undefined 本身是一个在全局范围内可用的变量/属性，它的值也是 undefined 。

```javascript
typeof undefined; // undefined

var foo;

foo === undefined; // true
```

但是，全局变量 undefined 并不是保留字，因此它可以被重新定义。 幸运的是，从 ECMA 5 开始，就不允许重新定义 undefined 了，但是在以前的版本和旧版浏览器中，可以执行以下操作：

```javascript
typeof undefined; // undefined
undefined = 99;
typeof undefined; // number
```

### null 到底代表了什么？

先看下面的示例：

```javascript
null == undefined; // true
null !== undefined; // true
```

很多人对此都感到困惑，实际上很简单。 null 和 undefined 之间唯一真正的关系是：它们在类型强制过程中都判断为 false。

之所以所以 null == undefined // true 是因为 == 没有执行严格的比较，因为在比较类型时使用 !== 更严格。 每当您把 null 看作是一个值时，它会始终以编程方式进行指定，并且在默认情况下从不设置。

### 访问对象的属性

尝试使用对象上一个不存在的属性时，也会得到 undefined ，如果把不存在的属性作为函数使用有时会引发错误。

```javascript
var foo = {};

foo.bar; // undefined
foo.bar(); // TypeError
```

如果想分辨“有未定义值的属性”和“根本不存在的属性”这两者，应该怎么做呢？

使用 typeof 或者 ===都会给你一个 undefined 的值。

使用 in 运算符能够检查对象中是否存在某个属性：

```javascript
var foo = {};

// undefined (这样不好，bar从未在window对象中被声明过)
typeof foo.bar;

// false (如果您不关心原型链，这样用)
"bar" in foo;

// false (如果你关心原型链，就这样用)
foo.hasOwnProperty("bar");
```

### 应该用 typeof 还是 in/hasOwnProperty？

这很显然。一般来说，如果要测试是一个属性否存在，那么就用 in/hasOwnProperty，如果要检查属性或变量的值，则用 typeof。

### 总结

- 检查变量是否存在

```javascript
if (typeof foo !== "undefined") {
}
```

- 检查对象上的属性是否存在，无论是否已经为它分配了值：

```javascript
// 存在于对象上，同时也检查原型
if ("foo" in bar) {
}

// 直接存在于对象上，不检查原型
if (bar.hasOwnProperty("foo")) {
}
```

- 检查对象上是否存在属性，并且属性具有值集（真值或假）

```javascript
var bar = {
  foo: false,
};

if ("foo" in bar && typeof bar.foo !== "undefined") {
  // bar.foo存在，并且它包含以编程方式分配的值
}
```
