---
title: js中Array类型检测的小坑
date: 2021-09-29 22:47:09
tags:
---

JavaScript 的 Array 对象是用于构造数组的全局对象，数组是类似于列表的高阶对象。

数组是一种类列表对象，它的原型中提供了遍历和修改元素的相关操作。JavaScript 数组的长度和元素类型都是非固定的。因为数组的长度可随时改变，并且其数据在内存中也可以不连续，所以 JavaScript 数组不一定是密集型的，这取决于它的使用方式。

### Array 类型检测

```javascript
function a(obj) {
  // ....
}
```

如果`obj`是个数组对象，想以不同的方式把它输出到其它对象。可以这样做

```javascript
if (obj.constructor === Array) {
  // ....
}
```

但同样的做法对于数组的子类来说是错误的：

```javascript
class ChildArray extends Array {}
const child = new ChildArray();
console.log(child.constructor === Array); // false
console.log(child.constructor === ChildArray); // true
```

所以如果想检查子类的类型，应该用`instanceof`

```javascript
console.log(child instanceof Array); // true
console.log(child instanceof ChildArray); // true
```

但是当引入多个 realm 时,情况会变得更加复杂

### Multiple realms

realm 包含`self`引用得 JavaScript 全局对象。因此，可以说在 worker 中运行得代码与在页面中运行的代码处于不同的 realm。在 iframe 之间亦是如此，但在同源的 iframe 中共享一个 ECMAScript 代理，意味着对象可以穿越 realm

```javascript
<iframe srcdoc="<script> var arr = []; </script>"></ifarme>
<script>
  const iframe = document.querySelect('iframe');
  const arr = iframe.contentWindow.arr;
  console.log(arr.constructor === Array); // false
  console.log(arr.constructor instanceof Array); // false
</script>
```

两个都是 fals，因为 iframe 有自己的数组构造函数，它与父页面中的构造函数不同

```javascript
console.log(Array === iframe.contenWindow.Array); // false
```

### Array.isArray

```javascript
console.log(Array.isArray(arr)); // true
```

`Array.isArray` 将为数组返回 true,即使它们是在另外一个 realm 中创建的。对于任何 realm 的`Array`的子了，它都会返回 true，这就是`JSON.stringify`内部的处理方法。

但是，这并不意味着`arr`有 array 方法。有些甚至所有的方法都设置为 undefined，或者数组可能已将其整个原型删除

```javascript
const noProtoArray = [];
Object.setPrototypeOf(noProtoArray, null);
console.log(noProtoArray.map); // undefined
console.log(noProtoArray instanceof Array); // false
console.log(Array.isArray(noProtoArray)); // true
```

如果要杜绝上述问题，可以通过 Array 的原型调用 Array 的方法：

```javascript
if (Array.isArray(noProtoArray)) {
  const mappedArray = Array.prototype.map.call(noProtoArray, callback);
}
```

### Symbols 与 realms

```javascript
<iframe srcdoc="<script>var arr = [1, 2, 3];</script>"></iframe>
<script>
  const iframe = document.querySelector('iframe');
  const arr = iframe.contentWindow.arr;

  for(const item of arr){
    console.log(item); // 1,2,3
  }
</script>
```

输出 1，2，3 并不奇怪，但 for-of 循环通过调用 arr[Symbol.iterator]来工作，在某种程度上可以跨域 realm，示例如下：

```javascript
const iframe = document.querySelector("iframe");
const iframeWindow = iframe.contentWindow;
console.log(Symbol === iframeWindows.Symbol); // false
console.log(Symbol.iterator === iframeWindows.Symbol.iterator); // true
```

虽然每个 realm 都有自己的 Symbol 实例，但 Symbol.iterator 在各个 realm 都是相同的。

Symbols 同时也是 javaScript 中最独特的内容

#### 多唯一性

```javascript
const symbolOne = Symbol("foo");
const symbolTwo = Symbol("foo");
console.log(symbolOne === symbolTwo); // false
const obj = {};
obj[symbolOne] = "hello";
console.log(obj[symbolTwo]); // undefined
console.log(obj[symbolOne]); // hello
```

传递给`Symbol`函数的字符串只是一个描述，即使在同一 realm 内，这些 Symbol 也是独一无二的

#### 最小唯一性

```javascript
const symbolOne = Symbol.for("foo");
const symbolTwo = Symbol.for("foo");
console.log(symboleOne === symbolTwo); // true
const obj = {};
obj[symbolOne] = "hello";
console.log(obj[symbolTwo]); // hello
```

`Symbol.for(str)`创建一个与传递它的字符串唯一的 symbol。有趣的是它在各个 realms 中都是一样的。

```javascript
const iframe = document.querySelector("iframe");
const iframeWindow = iframe.contentWindow;
console.log(Symbol.for("foo") === iframeWindow.Symbol.for("foo")); // true
```

这就是 `Symbol.iterator`大致的工作原理。

### 自定义 is 函数

如果我们想要定义自己的 is 函数并跨越 realm，Symbol 可以帮我们做到

```javascript
const typeSymbol = Symbol.for("whatever-type-symbol");
class Whatever {
  static isWhatever(obj) {
    return obj && Boolean(obj[typeSymbol]);
  }
  constructor() {
    this[typeSymbol] = true;
  }
}

const whatever = new Whatever();
Whatever.isWhatever(whatever); // true
```

即使来自另一个 realm，即使它是一个子类，即使他在原型上已被删除，也是可以的。
唯一的问题是，需要确定自己的 Symbol 名称在所有代码中是唯一的，如果其他人创建了一样名称的 Symbol，肯定返回 false。
