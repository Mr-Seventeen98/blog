---
layout: _posts
title:  Javascript 中的事件循环机制
date: 2021-10-13 12:09:51
tags:
---

### 前言
javascript是一门单线程非阻塞的脚本语言，最初的用途是用来和浏览器交互.
单线程意味着javascript运行任何代码的时候，都只有一个主线程来处理任务。javascript的单线程与它的用途有关，javascript的主要用途是与用户互动及操作DOM。这一用途决定了javascript只能是单线程的。如果javascript是多线程的，一个线程在DOM节点上添加内容，另一个线程在DOM上删除内容，这个时候就不能保证浏览器的一致性了。而单线程保证了执行顺序同时也限制了javascript的执行效率，因此有了web worker技术。
而非阻塞是当代码需要执行异步操作的时候（无法立即返回结果，需要花一定的时间才能返回结果，例如I/O事件）的时候，主线程会挂起（pending）这个任务，等待异步任务返回结果后按照一定的规则去执行相应的回调。
既然javascript是单线程的，而且同时又是非阻塞的，javascript是如何做到这一点的？——event loop（事件循环）

### 浏览器中js的事件循环机制

#### 执行栈与事件队列
当javascript运行的时候会将变量存到内存的堆（heap）和栈（stack）中。堆里面存放的是对象，而栈中存放的是基础类型变量及对象指针。
当javascript执行一个方法的时候会生成一个与方法对应的执行上下文（context），这个执行上下文中存放着方法的私有作用域、父级作用域的指向、方法参数，作用域中定义的变量及作用域的this对象。当一系列方法被依次调用的时候，因为js是单线程的，同一时间只能执行一个方法，于是这些方法被排队在一个单独的地方。这个地方被称为执行栈。
当一个js第一次执行的时候，js引擎会解析这段代码,并将其中的同步代码按照执行顺序加入执行栈中，然后从头开始执行。如果当前执行的是一个方法，那么js会向执行栈中添加这个方法的执行环境，然后进入这个执行环境继续执行其中的代码。当这个执行环境中的代码 执行完毕并返回结果后，js会退出这个执行环境并把这个执行环境销毁，回到上一个方法的执行环境.这个过程反复进行，直到执行栈中的代码全部执行完毕。如下图所示
![ab](/img/v2-2f761eb83b50f53d741e6aa1f15a9db1_b.gif)
通过上图可知，一个方法执行会向执行栈中加入这个方法的执行上下文，在这个执行上下文中还可以调用其他方法，甚至是自己，其结果不过是在执行栈中再添加一个执行上下文。这个过程可以无限的进行下去。除非发生栈溢出。

以上所说的都是同步代码的执行。那么当异步代码（例如ajax请求）执行的时候会发生什么？，js的另一大特点就是非阻塞，实现这一点的关键在于事件队列（Task Queue）。

js引擎遇到一个异步事件后不会一直等待其返回结果，而是将这个事件挂起，继续处理执行栈中的其它任务。当这个异步事件返回结果后，js会将这个事件加入到当前执行栈不同的另一个队列中（事件队列）。被放入事件队列中的事件不会立即执行回调，而是等待当前执行栈中的所有任务都执行完毕后，主线程处于空闲状态时，主线程会去查找事件队列中是否有任务。如果有，那么主线程会从中取出排在第一位的事件，并把这个事件对应的回调放入执行栈中，然后执行其中的同步代码。如此反复形成一个循环。这个过程被称为事件循环（Event Loop）
![event loop](/img/v2-da078fa3eadf3db4bf455904ae06f84b_1440w.png)
`图中的stack表示执行栈，web apis则是代表一些异步事件，而callback queue即事件队列`

#### macro task与micro task
以上的事件循环过程是一个宏观的表述，实际上因为异步任务之间并不相同，因此他们的执行优先级也有区别。不同的异步任务被分为两类：微任务（micro task）和宏任务（macro task）。

以下事件属于宏任务：
- `setInterval()`
- `setTimeout()`

以下事件属于微任务
- `new Promise()`
- `new MutaionObserver()`

在一个事件循环中，异步事件返回结果后会被放到一个任务队列中。然而，根据这个异步事件的类型，这个事件实际上会被对应的宏任务队列或者微任务队列中去。并且在当前执行栈为空的时候，主线程会 查看微任务队列是否有事件存在。如果不存在，那么再去宏任务队列中取出一个事件并把对应的回到加入当前执行栈；如果存在，则会依次执行队列中事件对应的回调，直到微任务队列为空，然后去宏任务队列中取出最前面的一个事件，把对应的回调加入当前执行栈...如此反复，进入循环。

<b>当当前执行栈执行完毕时会立刻先处理所有微任务队列中的事件，然后再去宏任务队列中取出一个事件。同一次事件循环中，微任务永远在宏任务之前执行</b>

```javascript
setTimeout(function () {
    console.log(1);
});

new Promise(function(resolve,reject){
    console.log(2)
    resolve(3)
}).then(function(val){
    console.log(val);
})
```

输出结果：
```json
2
3
1
```



