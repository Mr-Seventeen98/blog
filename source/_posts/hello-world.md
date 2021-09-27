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
