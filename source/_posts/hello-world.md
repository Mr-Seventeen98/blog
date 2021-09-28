---
title: ECMAScript 2016,2017和2018中所有新功能的示例
---

我们很难及时得知 JavaScript（ECMAScript）中的最新功能，同时找到相关的示例代码更加困难。

为了解决这个问题，我将在本文中介绍在 ES2016，ES2017 和 ES2018（最终草案）中添加的 18 个功能，这些功能在 TC39’s finished proposals 中列出，并展示相关的例子。

尽管这篇文章很长，不过应该很容易读懂。 你可以把它想象成“Netflix binge reading”。到此为止，我保证你将对所有这些功能有很多了解。

## ECMAScript 2016

- Array.prototype.includes

  - includes 是 Array 上的一个简单实例方法，能帮助我们轻松查找某项是否存在于数组中(处理 NaN 的方式与 indexOff 不同)。

```javascript
const arr = [1, 2, 3, 4, NaN];

// Instead of
if (arr.indexOf(3) >= 0) {
  console.log(true);
}

// Use
if (arr.includes(3)) {
  console.log(true);
}

// PS: 请注意 indexof 不适用于搜索 NaN
arr.includes(NaN); // true
arr.indexOf(NaN); // -1 没有找到 NaN
```

`小细节: 在 JavaScript 规范中人们想要把这个方法命名为 contains , 但是这个名字已经被 Mootools 使用了，所以只好被命名为 includes .`

- 幂运算符

  - 加法和减法等数学运算分别对应 + 和 - 运算符。 同样，**运算符通常用于指数运算。 在 ECMAScript 2016 中，引入 ** 来代替 Math.pow。

```javascript
// Instead of
Math.pow(7, 2); // 49

// Use
7 ** 2; //49
```

## ECMAScript 2017

- Object.values()
  - `Object.values()`是一个与 `Object.keys()`类似的新函数，不过它返回的是 `Object` 自身属性的所有值，不包括原型链中的任何值。

```javascript
const cars = { BMW: 3, Tesla: 2, Toyota: 1 };
// ES2015
// Instead of
const vals = Object.keys(cars).map(key => cars[key]);
console.log(vals); // [3,2,1]

// ES2017 and onwards
// Use
const values = Object.values(cars);
console.log(values); //[3,2,1]
```

- Object.entries()
  - `Object.entries()`与`Object.keys`相关，但它并不是仅返回键，而是以数组方式返回键和值。 这样一来，在循环中使用对象或将对象转换为`Maps`等操作将会变得非常简单。
  <p><b>Example 1:</b>（ECMAScript 2017 (ES8) — 在循环中使用 Object.entries() ）</p>

```javascript
const cars = { BMW: 3, Tesla: 2, Toyota: 1 };
// ES2015
// instead of extracting keys and then again looping
Object.keys(cars).forEach(function (key) {
  console.log(`key:${key} value:${cars[key]}`);
});

// ECMAScript 2017 (ES8)
// Use Object entries
for (let [key, value] of Object.entries(cars)) {
  console.log(`key:${key} value:${value}`);
}
```

  <p><b>Example 2:</b>（ECMAScript 2017 (ES8) — 使用 Object.entries() 将 Object 转化为 Map）</p>

```javascript
const cars = { BMW: 3, Tesla: 2, Toyota: 1 };
// ES2015
// instead of
// 获取对象键，然后添加每个项目以循环映射
const map1 = new Map();
Object.keys(cars).forEach(key => {
  map1.set(key, cars[key]);
});
console.log(map1); // Map { 'BMW' => 3, 'Tesla' => 2, 'Toyota' => 1 }

// ES 2017 and onwards
// Use...
const map = new Map(object.entries(cars));
console.log(map); // Map { 'BMW' => 3, 'Tesla' => 2, 'Toyota' => 1 }
```

### 字符串填充

- String 中添加了两个实例方法—— String.prototype.padStart 和 String.prototype.padEnd ，允许将空字符串或其他字符串附加/前置到原始字符串的开头或结尾。
  - 可以在想要用漂亮的格式打印输出时或者打印对齐等场景中派上用场。

```javascript
'someString'.padStart(numberOfCharcters [,stringForPadding]);
'5'.padStart(10) // '          5'
'5'.padStart(10, '=*') //'=*=*=*=*=5'
'5'.padEnd(10) // '5         '
'5'.padEnd(10, '=*') //'5=*=*=*=*='
```

### padStart 示例:

- 下面的示例中列出了不同长度的数字。 我们希望前置“0”，以便在显示时所有项目都具有相同的 10 位长度。 使用 padStart(10, '0')轻松实现这一目标。

```javascript
// ECMAscript 2017
// 如果您有一个不同长度的项目列表并希望将它们格式化以用于显示目的，您可以使用 padStart
const formatted = [0, 1, 12, 123, 1234, 12345].map(num => {
  num.toString().padStart(10, "0"); // 添加 0 直到 长度 为 10
});
console.log(formatted);
// prints...
// [
//   '0000000000',
//   '0000000001',
//   '0000000012',
//   '0000000123',
//   '0000001234',
//   '0000012345'
// ];
```

### padEnd 示例:

- 当我们打印不同长度的多个项目并希望它们正确对齐时，padEnd 真的很方便。
- 下面的示例是 padEnd，padStart 和 Object.entries 如何组合在一起以产生漂亮输出的一个很好的现实示例。

```javascript
const cars = {
  BMW: "10",
  Tesla: "5",
  Lamborghini: "0",
};
Object.entries(cars).map(([name, count]) => {
  // 名称末尾添加 ‘ -’直到满了20个字符
  // 数量前添加 ‘0’ 直到满了3个字符
  console.log(`${name.padEnd(20, " -")} Count: ${count.padStart(3, "0")}`);
});
// prints...
// BMW - - - - - - - - - Count: 010
// Tesla - - - - - - - - Count: 005
// Lamborghini - - - - - Count: 000
```

### 使用 padStart 与 padEnd 处理 Emojis 和其它双字节字符

- Emojis 和其他双字节字符使用多个字节的 unicode 表示。 所以 padStart 和 padEnd 可能无法按预期工作！
  - 例如：假设我们想要在字符串中填充 10 个 ❤️ 表情符号，结果如下所示

```javascript
// 请注意，不是 5 颗心，而是只有 2 颗心和 1 颗看起来很奇怪的心！
"heart".padStart(10, "❤️"); // prints.. '❤️❤️❤heart'
```

这是因为 ❤️ 的长度是 2 个码点（'\ u2764 \ uFE0F'）！ 单词 heart 本身是 5 个字符，所以我们只剩下 5 个字符来填充。 所以会发生什么事情，JS 使用’\ u2764 \ uFE0F’来填充两颗心并产生 ❤️❤️。 对于最后一个，它只使用第一个码点\ u2764 产生 ❤ 字符。

所以我们最终得到：❤️❤️❤heart

`可以通过此链接(https://encoder.internetwache.org/#tab_uni) 查看怎样对unicode char进行转换。`

### Object.getOwnPropertyDescriptors

- 此方法返回给定对象的所有属性的全部详细信息（包括 getter 方法 get 和 setter 方法 set）。 添加它的主要目的是允许浅层拷贝/克隆对象到另一个对象，该对象也复制 getter 和 setter 函数而不是 Object.assign。

- Object.assign 用于浅层拷贝除了原始源对象的 getter 和 setter 函数之外的所有细节。

- 下面的示例显示了 Object.assign 和 Object.getOwnPropertyDescriptors 以及 Object.defineProperties 之间的区别，以将原始对象 Car 复制到新对象 ElectricCar 中。 你将看到，通过使用 Object.getOwnPropertyDescriptors，discount 的 getter 和 setter 函数也会复制到目标对象中。

<b>以前</b>

```javascript
var Car = {
  name: "BMW",
  price: 1000000,
  set discount(x) {
    this.d = x;
  },
  get discount() {
    return this.d;
  },
};

//打印 Car 对象的 discount 属性的详细信息
console.log(Object.getOwnPropertyDescriptor(Car, "discount"));
// prints..
// {
//   get:[Function:get],
//   set:[Function:set],
//   enumerable:true,
//   configurable:true
// }

// 使用 Object.assign 将 Car 的属性复制到 ElectricCar
const ElectricCar = Object.assign({}, Car);
// 打印 ElectricCar 对象的折扣属性的详细信息
console.log(Object.getOwnPropertyDescriptor(ElectriceCar, "discount"));
// prints..
// {
//   value:undefined,
//   writable:true,
//   enumerable:true,
//   configurable:true
// }
// 请注意，为了减少代码，electricCar 对象中缺少 getter 和 setter
```

<b>之后</b>

```javascript
var Car = {
  name: "BMW",
  price: 1000000,
  set discount(x) {
    this.d = x;
  },
  get discount() {
    return this.d;
  },
};

// 使用 object.defineProperties 将 Car 的属性复制到electricCar2
// 并使用 Object.getOwnPropertyDescriptors 提取 Car 的属性
const ElectricCar2 = Object.defineProperties(
  {},
  Object.getOwnPropertyDescriptors(Car)
);

// 打印 ElectricCar2 对象的折扣属性的详细信息
console.log(Object.getOwnPropertyDescriptors(ElectricCar2, "discount"));

// prints..
// {
//   get:[Function:get],
//   set:[Function:set],
//   enumerable:true,
//   configurable:true
// }
// 请注意，ElectricCar2 对象中discount属性存在 getter 和 setter
```

### 在函数参数中添加尾随逗号

- 这是一个次要更新，允许我们在最后一个函数参数后面有逗号。 为什么？ 帮助使用像 git blame 这样的工具来确保只有新的开发人员的代码被标注。

  - 以下示例显示了问题和解决方案。

```javascript
// #1
function Person(
  name,
  age // < - - -年龄参数后的尾随逗号引发错误
) {
  this.name = name;
  this.age = age;
}
// #2
function Person(
  name,
  age, // 添加逗号 < - - -因为逗号会把git工具认为是改动过的
  address // 添加新参数
) {
  this.name = name;
  this.age = age;
  this.address = address;
}

// ECMScript 2017 允许添加尾随逗号
function Person(
  name,
  age // < - - - 由于尾随逗号#2不需要更改
) {
  this.name = name;
  this.age = age;
}
```

`注意：也可以使用尾随逗号调用函数！`

### Async/Await

- 到目前为止，这是最重要和最有用的功能。 异步函数允许我们不必处理回调并使整个代码看起来很简单。

- async 关键字告诉 JavaScript 编译器以不同方式处理函数。 只要到达该函数中的 await 关键字，编译器就会暂停。 它假定 await 之后的表达式返回一个 promise 并等待，直到 promise 被解决或被拒绝，然后才进一步移动。

- 在下面的示例中，getAmount 函数调用两个异步函数 getUser 和 getBankBalance。 我们可以做到这一点，但是使用 async await 更加优雅和简单。

```javascript
// ES2015 Promise
function getAmount(userId) {
  getUser(userId)
    .then(getBankBalance)
    .then(amout => {
      console.log(amout);
    });
}

// Use ES2017
async function getAmout2(userId) {
  var user = await getUser(userId);
  var amout = await getBankBalance(user);
  console.log(amout);
}

getAmout("1"); // $1,000
getAmout2("1"); // $1,000

function getUser(userId) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve("john");
    }, 1000);
  });
}

function getBankBalance(user) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (user === "john") {
        resolve("$1,000");
      } else {
        reject("unknown user");
      }
    }, 1000);
  });
}
```

### 并行调用 async/await

- 在前面的例子中，我们调用了两次 await，每次会等待一秒钟（总共 2 秒）。 不过我们可以并行化处理它，因为 a 和 b 不使用 Promise.all 相互依赖。

```javascript
// 异步函数本身返回一个承诺
asyn function doubleAndAdd(a,b){
  // 请注意，我正在使用 promise.all
  // 所以使用数组结构来接受结果
  [a,b] = await Promise.all([doubleAfterlSec(a),doubleAfterlSec(b)]);
  return a + b;
}

// 用法
doubleAndAdd(1,2).then(console.log);

function doubleAfterlSec(param){
  return new Promise(resolve=>{
    setTimeout(resolve(param*2),1000);
  });
}
```

### async/await 错误处理功能

- 使用异步等待时，有多种方法可以处理错误。

  - 在函数中使用 try catch

```javascript
// 方法1 在函数内使用 try catch
asyn function doubleAndAdd(a,b){
  try{
    a = await doubleAfterlSec(a);
    b = await doubleAfterlSec(b);
  } catch(e){
    return NaN;
  }
  return a + b;
}

// 用法
doubleAndAdd('one',2).then(console.log); // NaN
doubleAndAdd(1,2).then(console.log); // 6

function doubleAfterlSec(param){
  return new Promise((resolve,reject) => {
    setTimeout(function(){
      let val = param * 2;
      isNaN(val) ? reject(NaN) : resolve(val);
    },1000);
  });
}
```

- 捕获每个等待表达式 由于每个 await 表达式都返回一个 Promise，因此可以捕获每行的错误，如下所示。

```javascript
// 方法2 每个await都捕获错误
// 因为每个 await 表达式本身就是一个Promise
asyn function doubleAndAdd(a,b){
  a = await doubleAfterlSec(a).catch(e => console.log('"a" is NaN'));
  b = await doubleAfterlSec(b).catch(e => console.log('"a" is NaN'));
  if(!a || !b){
    return NaN;
  }
  return a + b;
}

// 用法
doubleAndAdd('one',2).then(console.log); // NaN and logs: "a" is NaN
doubleAndAdd(1,2).then(console.log); // 6

function doubleAfterlSec(param){
  return new Promise((resolve,reject) => {
    setTimeout(function(){
      let val = param * 2;
      isNaN(val) ? reject(NaN) : resolve(val);
    },1000);
  });
}
```

- 捕获整个 async-await 函数

```javascript
// 方法3 除了在函数之外处理之外什么都不做
// 由于 async / await 返回一个Promise，我们可以捕获整个函数的错误
asyn function doubleAndAdd(a,b){
  a = await doubleAfterlSec(a);
  b = await doubleAfterlSec(b);
  return a + b;
}

// 用法
doubleAndAdd('one',2).then(console.log).catch(console.log); // use catch
```

## ECMAScript 2018

### 共享内存和 Atomics

- 这是一个巨大的，非常先进的功能，是 JS 引擎的核心增强功能。
- <b>主要思想是为 JavaScript 提供某种多线程功能，以便 JS 开发者可以通过自己管理内存——而不是让 JS 引擎管理内存——来编写高性能的并发程序。</b>
- 这是通过一种名为 [SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer) 的新型全局对象完成的，该对象实质上将数据存储在共享内存空间中。因此，这些数据可以在主 JS 线程和 Web 工作线程之间共享。
- 到目前为止，如果我们想在主 JS 线程和 Web 工作者之间共享数据，就必须复制数据并使用 postMessage 将其发送到另一个线程。以后不会再这样了！
- 只需使用 SharedArrayBuffer，主线程和多个 Web 工作线程都可以立即访问数据。
- 但是在线程之间共享内存会导致竞争条件。为了帮助避免竞争条件，引入了“Atomics”全局对象。 Atomics 提供了各种方法，使得线程在使用其数据时锁定共享内存。它还提供了安全地更新共享内存中数据的方法。
- 建议通过某个库使用此功能，但是现在没有基于此功能构建的库。 如果你有兴趣，我建议阅读：
  - [From Workers to Shared Memory](https://lucasfcosta.com/2017/04/30/JavaScript-From-Workers-to-Shared-Memory.html)*—* [lucasfcosta](https://lucasfcosta.com/)
  - [A cartoon intro to SharedArrayBuffers](https://hacks.mozilla.org/category/code-cartoons/a-cartoon-intro-to-sharedarraybuffers/) *—* [Lin Clark](https://medium.com/@linclark)
  - [Shared memory and atomics](https://2ality.com/2017/01/shared-array-buffer.html) *—*[ Dr. Axel Rauschmayer](http://dr-axel.de/)

### 删除了标记模板文字限制

- 首先，我们需要澄清“标记模板文字”是什么，以便我们更好地理解这个功能。
- 在 ES2015 +中，有一个称为标记模板文字的功能，允许开发人员自定义字符串的插值方式。 例如，在标准方式中，字符串被插入如下…

```javascript
// 标准字符串文字插值
const firstName = "Raja";
const greetings = `Hello ${firstName}!`;
console.log(greetings); // "Hello Raja!"
```

- 在标记的文字中，你可以编写一个函数来接收字符串文字的硬编码部分，例如['Hello'，'！']，或者替换变量，例如['Raja']，作为参数进入自定义函数（例如 greet），并从该自定义函数返回您想要的任何内容。
- 下面的示例演示自定义“标记”函数 greet，根据当前时间返回例如“Good Morning!” “Good afternoon!”之类的字符串。

```javascript
// 根据一天的时间在 后面拼接上 “Good Morning!” “Good afternoon!”之类的字符串
function greet(hardCodePartsArray, ...replacementPartsArray) {
  console.log(hardCodePartsArray); // [ 'Hello', '!' ]
  console.log(replacementPartsArray); // [ 'Raja' ]
  let str = "";
  hardCodePartsArray.forEach((string, i) => {
    if (i < replacementPartsArray.length) {
      str += `${string} ${replacementPartsArray[i]} || ''`;
    } else {
      str += `${string} ${timeGreet()}`; // < - - 这里拼接上 Good Morning/afternoon/evening here
    }
  });
  return str;
}

// 使用
const firstName = "Raja";
const greetings = greet`Hello ${firstName}!`; // 标记文字
console.log(greetings); // "Hello Raja! Good Morning"

function timeGreet() {
  const hr = new Date().getHours();
  return hr < 12
    ? "Good Morning"
    : hr < 18
    ? "Good Afternoon"
    : "Good Evening!";
}
```

- 现在我们讨论了“Tagged”函数是什么，许多人想要在不同的领域中使用此功能，例如在终端中使用命令行或 HTTP 请求来拼接 URIs 等等。
- ⚠️ 标记字符串字符的问题 问题是在 ES2015 和 ES2016 规范不允许使用转义字符，如“\u”（unicode），“\x”（十六进制），除非它们看起来完全像\u00A9 或\u{2F804}或\xA9。
- 因此，如果你有一个 Tagged 函数在内部使用其他领域的规则（如终端的规则），可能需要使用\ubla123abla 这样的字符，它看起来一点也不像\u0049 或\u {@F804}的样子，最后你将会得到一个语法错误。
- 不过在 ES2018 中，只需要 Tagged 函数返回一个具有“cooked”属性（赋值为“undefined”）和“raw”属性（ 你想要的任何内容）的对象即可。

```javascript
function myTagFunc(str) {
  return { cooked: "undefined", raw: str.raw[0] };
}

var str = myTagFunc`hi\ubla123abla`;
console.log(str); // { cooked: "undefined", raw: "hi \\unicode" }
```

### 正则表达式的“dotall”标志

- 目前在正则表达式中，虽然点（“.”）应该与单个字符匹配，但它不能与\n \r \f 等新行字符匹配。
- 例如：

```javascript
//Before
/first.second/.test("first\nsecond"); //false
```

- 这个功能使点运算符可以匹配任何单个字符。 为了确保不会破坏任何内容，我们需要在创建正则表达式时使用\s 标志才能使其正常工作。

```javascript
//ECMAScript 2018
/first.second/s.test("first\nsecond"); //true   Notice: /s 👈🏼
```

- 以下是提案文档中的全部 API：(https://github.com/tc39/proposal-regexp-dotall-flag)

```javascript
const re = /foo.bar/s;
// 或者 const re=new RegExp('foo.bar','s');
re.test("foo\nbar"); // true
re.dotAll; // true
re.flags; // 's'
```

### 正则表达式命名组

- 此增强功能带来了其他语言（如 Python，Java 等）具有的正则功能，称为“命名组”。能够允许开发者编写正则表达式，通过格式(? &lt;name&gt;...)提供不同部分的名称（标识符）来进行分组。 这样一来就可以使用该名称轻松得到需要的任何分组。

#### 基本命名组示例

- 在下面的示例中，我们使用(?&lt;year&gt;) (?&lt;month&gt;) and (?&lt;day&gt;)名称对日期正则的不同部分进行分组。 生成的对象将包含一个 groups 属性，在 groups 属性中存在相应值的 year, month 和 day 属性。

```javascript
// 之前
let rel = /(\d{4})-(\d{2})-(\d{2})/;
let result = rel.exec("2021-09-28");
console.log(result);
// ["2021-09-28","2021","09","28",index:0,input:"2021-09-28",groups:undefined]

//ECMScript 2018
let rel1 = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/u;
let result1 = rel1.exce("2021-09-28");
console.log(result1);
// ["2021-09-28","2021","09","28",index:0,input:"2021-09-28",groups:{ year:"2021",month:"09",day:"28" }]

console.log(result1.groups.year); // 2021
```

#### 在正则表达式内使用命名组

- 我们可以使用 \k<group name> 格式来反向引用正则表达式本身中的组。 以下示例显示了它的工作原理。

```javascript
// 在下面的例子中，我们有一个名为fruit的组
// 它可以匹配“apple”或“orange”，我们可以反向引用
// 该组的结果使用 "\k<group name>" 即，(\k<fruit>)
// 所以是匹配两边的词是否相等

let sameWords = /(?<fruit>apple|orange) = \k<fruit>/u;

sameWords.test("apple===apple"); // true
sameWords.test("orange===orange"); // true
sameWords.test("apple===orange"); // false
```

#### 在 String.prototype.replace 中使用命名组

- 命名组功能现在被内置到 String 的 replace 实例方法中。 所以我们可以轻松地替换字符串中的单词。
- 例如，将“firstName，lastName” 更改为“lastName，firstName”。

```javascript
// 调换 firstName lastName 成 lastname, firstName
let re = /(?<firstName>[A-Za-z]+) (?<lastName>[A-Za-z]+$)/u;
"Raja Rao".replace(re, "$<lastName>,${firstName}"); // "Rao, Raja"
```

### 对象的 rest 属性

- Rest 运算符 ...（三个点）允许我们在提取 Object 属性时丢弃一些属性。

#### 使用 rest 来帮助仅提取所需的属性

```javascript
// 提取名字和年龄
// 并将其余项目存储在剩余变量中
let { firstName, age, ...remaining } = {
  firstName: "john",
  lastName: "smith",
  age: 20,
  height: "5.10",
  race: "martian",
};
console.log(firstName); // john
console.log(age); // 20
console.log(remaining); // {lastName: "smith",height: "5.10",race: "martian"}
```

#### 更酷的是，你可以删除不需要的项目！

```javascript
// 假设要删除 ssn ，我们不必循环整个对象并创建一个新的对象
// 简单地使用rest解构来提取ssn并将剩余的属性保存到新的对象中
let { SSN, ...newObj } = {
  firstName: "john",
  lastName: "smith",
  SSN: '123-45-6789'
  race: "martian",
};
console.log(newObj); // {firstName: "john",lastName: "smith",race: "martian"}
```

### 对象的 Spread 属性

- Spread 属性看起来就像具有三个点的 rest 属性...但不同之处在于使用 spread 来创建（重构）新对象。
  - 提示：展开运算符用于等号的右侧。 其余的用在等号的左侧。

```javascript
// 合并两个对象的属性
const person = { fName: "john", age: 20 };
const account = { name: "bofa", amout: "$1000" };

// 通过Spread运算符提取 preson 和 account 的属性并将其添加到新对象中
const presonAndAccount = { ...person, ...account };
cnosole.log(presonAndAccount); // { fName:"john",age:20, name:"bofa", amout:"$1000"}
```

### 正则 Lookbehind 断言

- 这是对正则表达式的一种增强，它允许我们确认某些字符在其他字符串*之前*。
- 现在可以使用一个组 (?<=…)（问号，小于，等于）来判断前向断言。
- 此外，也可以使用 (?<!…) （问号，小于，感叹号）来查看否定断言。 基本上，只要-ve 断言通过，就会匹配。
- <b>积极断言：</b> 假设我们要确保#符号存在于 winning 之前（即： #winning），并希望正则表达式只返回字符串“winning”。应该这样写。

```javascript
\(?<=#).*\.test("winning"); // false
\(?<=#).*\.test("#winning"); // true

// ECMScript 2018 以前
"#winning".match(\#.*\)[0]; // 验证 #winning 是否以 # 开头

// ECMScript 2018
"#winning".match(\(?<=#).*\)[0]; // 验证 #winning 是否以 # 开头
```

- 否定断言： 假设我们想要从具有 € 符号的行中提取数字，同时忽略带有$符号的数字。

```javascript
//由于前面的 $ 符号，数字 3.00 不匹配
"A gallon of milk is $3.00".match(/(?<!\$)\d+\.?\d+/); // null

// 数字 2.43 匹配，因为 前面没有 $ 符号 规则不包含 ￥ 符号
"A gallon of milk is ￥2.43".match(/(?<!\$)\d+\.?\d+/); // 2.43
```

### 正则表达式 Unicode 属性转义

- 编写匹配各种 unicode 字符的正则表达式并不容易。 像 \w , \W , \d 等的东西只匹配英文字符和数字。 但是其他语言如印地语，希腊语等中的数字该怎么处理呢？
- 这就是 Unicode Property Escapes 的用武之地。实际上，Unicode 为每个符号（字符）添加元数据属性，并使用它来分组或表征各种符号。
- 例如，Unicode 数据库将所有印地语字符（हिन्दी）归为一个名为 Script 的属性，其值为 Devanagari，另一个属性为 Script_Extensions，其值为 Devanagari。 所以我们可以搜索 Script = Devanagari 并获得所有印地语字符。
- `梵文可以用于各种印度语言，如马拉地语，印地语，梵语等。`
- 从 ECMAScript 2018 开始，可以用\p 来转义字符以及用{Script = Devanagari}来匹配所有这些印度字符。 也就是说，我们可以在 RegEx 中使用：\p{Script = Devanagari}来匹配所有梵文字符。

```javascript
// 以下匹配多个印地文字符
/^\p{Script = Devanagari}+/u.test("संस्कृतम्"); // true
```

- 同样，Unicode 数据库将 Script_Extensions（和 Script）属性下的所有希腊字符组合为希腊语。 所以我们可以使用 Script_Extensions = Greek 或 Script = Greek 搜索所有希腊字符。
- <b>也就是说，我们可以在 RegEx 中使用：\p{Script = Greek}来匹配所有希腊字符。</b>

```javascript
// 以下匹配多个印地文字符
/^\p{Script = Greek}+/u.test("ώσ"); // true
```

- 此外，Unicode 数据库在布尔属性 Emoji，Emoji_Component，Emoji_Presentation，Emoji_Modifier 和 Emoji_Modifier_Base 下存储各种类型的 Emojis，其属性值为“true”。 因此，我们只需选择表情符号即可搜索所有表情符号。
- <b>也就是说，我们可以使用： \p{Emoji} ,\Emoji_Modifier 等来匹配各种表情符号。</b>
- 以下示例将演示这一点。

```javascript
// 以下匹配一个表情符号字符
/p{Emoji/u.test("❤"); // true
// 以下失败，因为黄色表情符号不需要/具有 Emoji_Modifier！
/p{Emoji}\p{Emoji_Modifier}/u.test("✌"); // false
// 以下匹配一个表情符号字符\p(Emoji} 后跟 p(Emoji_Modifier}
/p{Emoji}ip{Emoji_Modifier}/u.test("🤞"); //true
//说明：
//默认情况下，胜利表情符号为黄色。
//如果我们使用棕色、黑色或同一表情符号的其他变体，它们被视为原始表情符号的变体，并使用两个 unicode 字符表示。//一个用于原始表情符号，然后是另一个用于颜色的 unicode 字符。
//所以在下面的例子中，虽然我们只看到一个棕色的胜利表情符号，但它实际上使用了两个 unicode 字符，一个用于表情符号，另一个用于
//对于棕色。
//在 Unicode 数据库中，这些颜色具有 Emoji_Modifier 属性。
//所以我们需要同时使用 \p{Emoji} 和 p{Emoji_Modifier} 来正确和/完全匹配棕色表情符号。
/p{Emoji}p{Emoji_Modifier}/u.test("🤞"); //true
```

- 最后，我们可以使用转义字符大写“P”（\P）而不是小 p（\p）来进行否定匹配。
  参考文献：
  ECMAScript 2018 提案（https://mathiasbynens.be/notes/es-unicode-property-escapes）
  https://mathiasbynens.be/notes/es-unicode-property-escapes

### Promise.prototype.finally()

- finally()是一个添加到 Promise 的新实例方法。 其主旨是允许在 resolve 或 reject 后运行回调以帮助清理。 finally 的回调被调用时而没有任何参数，同时任何情况下都会被执行。
- 来看看各种情形。

- ECMAScript 2018 — finally() in resolve case）

```javascript
// resolve case
let started = true;

let myPromise = new Promise(function (resolve, reject) {
  resolve("all good");
})
  .then(val => {
    console.log(val); // all good
  })
  .catch(e => {
    console.log(e); //skipped
  })
  .finally(() => {
    console.log("this function is always executed!");
    started = false; // clean up
  });
```

- （ECMAScript 2018 — finally() in reject case）

```javascript
// reject case
let started = true;

let myPromise = new Promise(function (resolve, reject) {
  reject("reject apple");
})
  .then(val => {
    console.log(val); // reject apple
  })
  .catch(e => {
    console.log(e); //skipped
  })
  .finally(() => {
    console.log("this function is always executed!");
    started = false; // clean up
  });
```

- （ECMASCript 2018 — finally() in Error thrown from Promise case）

```javascript
// reject case
let started = true;

let myPromise = new Promise(function (resolve, reject) {
  throw new Error("Error");
})
  .then(val => {
    console.log(val); // skipped
  })
  .catch(e => {
    console.log(e); // catch 被调用，因为有一个错误
  })
  .finally(() => {
    // 注意这里没有传递任何值
    console.log("this function is always executed!");
    started = false; // clean up
  });
```

- （ECMAScript 2018 — Error thrown from within catch case）

```javascript
// reject case
let started = true;

let myPromise = new Promise(function (resolve, reject) {
  throw new Error("something happend");
})
  .then(val => {
    console.log(val); // skipped
  })
  .catch(e => {
    throw new Error("throw another error");
  })
  .finally(() => {
    // 注意这里没有传递任何值
    console.log("this function is always executed!");
    started = false; // clean up
    // 来自 catch 的错误将需要由其他地方的调用函数处理
  });
```

### 异步迭代

- 这是一个非常有用的功能。 它允许我们轻松的创建异步代码循环！
- 此功能添加了一个新的“for-await-of”循环，允许我们在循环中调用返回 promises（或带有一堆 promise 的 Arrays）的异步函数。更酷的是循环会在在进行下一个循环之前等待每个 Promise。

```javascript
const promises = [
  new Promise(resolve => resolve(1)),
  new Promise(resolve => resolve(2)),
  new Promise(resolve => resolve(3)),
];

// for-of 使用定期同步迭代器不等待Promise 的状态回到 resolve
async function test1() {
  for (const obj of promises) {
    console.log(obj); // 3 Promise objects
  }
}

// ECMScript 2018
// for-of 使用定期同步迭代器循环等待Promise 的状态回到 resolve
async function test2() {
  for await (const obj of promises) {
    console.log(obj); // 1,2,3
  }
}

test1(); // Promise,Promise,Promise

test2(); // 1,2,3 ...prints values
```
