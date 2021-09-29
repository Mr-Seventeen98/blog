---
title: 正确使用异步函数的姿势
date: 2021-09-29 21:59:07
tags:
---

在编写异步函数时，await 、 return 与 return await 之间存在差异，选择正确的处理方式非常重要。

让我们先从这个异步函数开始：

```javascript
async function waitAndMaybeReject() {
  // Wait one second
  await new Promise(r => setTimeout(r, 1000));
  // Toss a coin
  const isHeads = Boolean(Math.round(Math.random()));

  if (isHeads) return "yay";
  throw Error("Boo!");
}
```

这段代码将返回一个等待一秒的 promise，同时各有一半的可能性返回'yay’或者错误。 接下来我们以一些微妙的其他方式使用它：

### 仅仅调用

```javascript
async function foo() {
  try {
    waitAndMaybeReject();
  } catch (e) {
    return "caught";
  }
}
```

在这里，如果你调用 foo，返回的 promise 将始终得到 undefined，而不是 waiting。

由于我们没有 await 或返回 waitAndMaybeReject()的结果，因此代码不会对它做出任何反应。 像这样的代码通常是错误的。

### 使用 await

```javascript
async function foo() {
  try {
    await waitAndMaybeReject();
  } catch (e) {
    return "caught";
  }
}
```

在这里，如果你调用 foo，返回的 promise 将始终等待一秒钟，然后得到返回值 undefined，或者返回'caught'。

因为我们 await waitAndMaybeReject()的结果，它将异常情况将变为 throw，并且 catch 块将会被执行。 但是，如果 waitAndMaybeReject()完成，我们不会对该值执行任何操作。

### 使用 return

```javascript
async function foo() {
  try {
    return waitAndMaybeReject();
  } catch (e) {
    return "caught";
  }
}
```

在这里，如果你调用 foo，返回的 promise 将始终等待一秒，然后得到'yay'，或者是 Error(‘Boo!’)。

通过返回 waitAndMaybeReject()，我们推迟了它的结果，所以 catch 块永远不会运行。

### 使用 return await

你在 try/catch 块中真正想要的东西是 return await：

```javascript
async function foo() {
  try {
    return await waitAndMaybeReject();
  } catch (e) {
    return "caught";
  }
}
```

在这里，如果你调用 foo，返回的 promise 将始终等待一秒，然后返回‘yay'，或者得到'caught'。

因为我们 await waitAndMaybeReject()的结果，所以它的异常情况将变为 throw，并且我们的 catch 块将被执行。 如果 waitAndMaybeReject()完成，那么返回其结果。

如果上述内容令你感到困惑，可能把它分解为两个单独的步骤更容易理解：

```javascript
async function foo() {
  try {
    //等待waitAndMaybeReject()的结果结算，
    //并将履行的值分配给fulfilledValue：
    const fulfilledValue = await waitAndMaybeReject();
    //如果waitAndMaybeReject（）的结果拒绝，我们的代码
    //抛出异常，并跳到catch块。
    //否则，此块继续运行：
    return fulfilledValue;
  } catch (e) {
    return "caught";
  }
}
```

注意：在 try/catch 块之外，return await 是多余的，甚至有一个 ESLint 规则来检测它，但是它允许存在于 try/catch 中。
