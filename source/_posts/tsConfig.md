---
layout: _posts
title: typesceipt 编译上下文
date: 2021-10-18 12:20:25
tags:
---

## 编译上下文

TypeSrcipt 编译上下文，可以用它来给文件分组，告诉 TypeScript 哪些文件是有效的，哪些是无效的。除了有效文件所携带信息外，编译上下文还包含有正在被使用的编译选项的信息，定义这种逻辑分组。配置 tsconfig.json 文件来处理这些事情。

### tsconfig.json

#### 基础配置

开始使用 tsconfig.json 是一件比较容易的事，你仅仅需要写下：

```json
{}
```

在项目的根目录下创建一个名为 tsconfig.json 的空 JSON 文件。通过这种方式，TypeScript 将 会把此目录和子目录下的所有 .ts 文件作为编译上下文的一部分，它还会包含一部分默认的编译选项。

#### 基本选项

###### Files

Files：指定要包含在程序中的文件列表。如果找不到任何文件，则会抛异常。

```json
"files": [
    "core.ts",
    "sys.ts",
    "types.ts",
    "scanner.ts",
    "parser.ts",
    "utilities.ts",
    "binder.ts",
    "checker.ts",
    "tsc.ts"
  ]
```

**当只有少量文件并且不需要使用[glob](https://github.com/isaacs/node-glob)来匹配文件时，这很有用。如果需要，请使用 include。**

###### Extends

extends 的值是一个字符串，其中包含要继承的另一个配置文件的路径。路径可以使用 Node.js 样式解析。
基础文件中的配置首先加载，然后被继承配置文件中的配置覆盖。在配置文件中找到的所有相对路径都将相对于它们起源的配置文件进行解析。
值得注意的是，继承配置文件中的`files`，`include`和`exclude`属性会覆盖父级配置文件中的相应属性，并且不允许配置文件之间存在循环关系。
目前，唯一被排除在继承之外的顶级属性是`references`。

`configs/base.json:`

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

`tsconfig.json:`

```json
{
  "extends": "./configs/base",
  "files": ["main.ts", "supplemental.ts"]
}
```

`tsconfig.nostrictnull.json:`

```json
{
  "extends": "./tsconfig",
  "compilerOptions": {
    "strictNullChecks": false
  }
}
```

在配置文件中找到的具有相对路径的属性（未从继承中排除）将相对于它们起源的配置文件进行解析。

###### Include

Include：指定要包含在程序中的文件名或模块数组。这些文件名是相对于 tsconfig.json 文件的目录解析的。

```json
{
  "include": ["src/**/*", "tests/**/*"]
}
```

结果

```json
.
├── scripts                ⨯
│   ├── lint.ts            ⨯
│   ├── update_deps.ts     ⨯
│   └── utils.ts           ⨯
├── src                    ✓
│   ├── client             ✓
│   │    ├── index.ts      ✓
│   │    └── utils.ts      ✓
│   ├── server             ✓
│   │    └── index.ts      ✓
├── tests                  ✓
│   ├── app.test.ts        ✓
│   ├── utils.ts           ✓
│   └── tests.d.ts         ✓
├── package.json
├── tsconfig.json
└── yarn.lock
```

`include`和`exclude`支持通配符来全局匹配：
`*` 匹配零个或多个字符（不包括目录分隔符）
`?` 匹配任何一个字符（不包括目录分隔符）
`**/` 匹配嵌套到任何级别的任何目录
如果全局匹配模式不包含文件扩展名，则仅支持默认扩展名的文件（例如，默认情况下为 .ts、.tsx 和 .d.ts，如果 allowJs 设置为 true，则为 .js 和 .jsx）。

###### Exclude

指定解析`include`时应屏蔽的文件名或模块数组。
重要提示：仅屏蔽`include`设置而包含的文件的更改。代码中的`import`语句、`types`、`/// <reference` 指令或在文件列表中指定，`exclude`指定的文件仍然可以成为代码库的一部分。
它不是一种阻止文件被包含在代码库中的机制——它只是改变了包含设置发现的内容。

###### References

References：是一种将 TypeScript 程序组织成更小的部分的方法。使用`References`可以大大缩短构建和编辑器的交互时间，强制组件之间的逻辑分离，并以更合理的方式来组织代码。

#### 编译选项

可以通过 compilerOptions 来定制你的编译选项：
编译选项按照功能可以分为以下几大类

- Type Checking（类型检测）

```json
{
  "compilerOptions": {
    // 配置项.....
  }
}
```

##### Type Checking

类型检测的相关配置详解

###### `allowUnreachableCode`

allowUnreachableCode：是否允许出现无法访问的代码
配置

```json
"allowUnreachableCode":false
```

可选值：

- `undefined` `默认值` 向编辑提供建议作为警告
- `true` 忽略无法访问的代码
- `false` 使用无法访问的代码时编译报错
  从以下示例可以看出这些警告与`javaScript` 语法没有有关系，而是与无法访问的代码有关系

```javascript
function fn(n: number) {
  if (n > 5) {
    return true;
  } else {
    return false;
  }
  return true;
}
```

当 `allowUnreachableCode`配置为 `false`时:

```javascript
function fn(n: number) {
  if (n > 5) {
    return true;
  } else {
    return false;
  }
  return true; // 检测到无法访问的代码
}
```

检测到无法访问的代码这个错误会导致代码的正常运行。

###### `allowUnusedLabels`

allowUnusedLabels：是否运行使用未定义的字段
配置

```json
"allowUnusedLabels":false
```

可选值：

- `undefined` `默认值` 向编辑提供建议作为警告
- `true` 忽略未定义的字段
- `false` 使用未定义的字段的时候编译报错
  `字段在javaScript中很少见，通常是使用对象字面量`

```javascript
function verifyAge(age: number) {
  // Forgot 'return' statement
  if (age > 18) {
    verified: true; // 未定义verified字段
  }
}
```

###### `alwaysStrict`

alwaysStrict: 确保您的文件在 `ECMAScript` 严格模式下解析，并为每个源文件里面加入`use strict`
严格模式是在 ES5 中引入的，它为 JavaScript 引擎的运行时提供行为调整以提高性能，并抛出一组错误而不是忽略它们

```json
"alwaysStrict": true
```

###### `exactOptionalPropertyTypes`

exactOptionalPropertyTypes：启用`exactOptionalPropertyTypes`后，`TypeScript`会应用更严格的规则来验证属性的类型类型或`interfaces`具有?的属性，默认值为`false`
配置

```json
"exactOptionalPropertyTypes":false
```

例如，接口中定义了一个属性可以是字符串：`dark`或`light`或者它该属性不存在对象中

```javascript
interface UserDefaults {
  colorThemeOverride?: "dark" | "light";
}
```

如果`exactOptionalPropertyTypes`设置为`false`，可以给`colorThemeOverride`设置 3 个值`dark`,`light`,和`undefined`
将值设置成`undefined`大多数情况下`javaScript`检查是否存在的时候是失败的。但这不是很准确，实际上`colorThemeOverride`:`undefined`和`colorThemeOverride`属性不存在是不一样的。例如：属性没有定义和`undefined`，在使用`as`运算符时会有不同的结果。
当`exactOptionalPropertyTypes`设置为`true`时，`TypeScript`在给可选属性赋值`undefined`时会出现以下错误。

```javascript
const settings = getUserSettings();
settings.colorThemeOverride = "dark";
settings.colorThemeOverride = "light";

// But not:
settings.colorThemeOverride = undefined;
// error info: Type 'undefined' is not assignable to type '"dark" | "light"' with 'exactOptionalPropertyTypes: true'.
// Consider adding 'undefined' to the type of the target.
```

###### `noFallthroughCasesInSwitch`

noFallthroughCasesInSwitch：不允许 `switch` 的 `case` 语句贯穿，确保 `switch` 语句中的任何非空 `case` 包含 `break` 或 `return`
配置

```json
"noFallthroughCasesInSwitch":true
```

示例：

```javascript
const a: number = 6;

switch (a) {
  case 0:
    // error info: Fallthrough case in switch.
    console.log("even");
  case 1:
    console.log("odd");
    break;
}
```

###### `noImplicitAny`

noImplicitAny：在表达式和声明上有隐含的`any`类型时报错
配置

```json
"noImplicitAny":true
```

在某些情况下，当`TypeScript`无法推断出数据类型时候，会默认将变量的类型推断为`any`，可能会导致一些其他的错误。例如：

```javascript
function fn(s) {
  console.log(s.subtr(3));
}
fn(42);
```

当配置了`noImplicitAny`为`true`时，`TypeScript`推断出`any`时候会抛出错误。例如：

```javascript
function fn(s) {
  console.log(s.subtr(3)); // Parameter 's' implicitly has an 'any' type.
}
```

###### `noImplicitOverride`

noImplicitOverride：在使用类的继承时候，如果子类有重载父类的方法时，随着父类的方法改名，或者发生参数的变化时候，子类重载的方法和父类的方法造成了不统一。
配置

```json
"noImplicitOverride":false
```

例如：

```javascript
class Album {
  download() {
    // Default behavior
  }
}

class SharedAlbum extends Album {
  download() {
    // Override to get info from many sources
  }
}
```

当出于一些业务需求的时候父类的`download`方法需要重构成`setup`,在这种情况下`TypeScript`并没有任何提示说明`SharedAlbum`类中的`download`函数是要覆盖父类的`setup`函数

```javascript
class Album {
  setup() {
    // Default behavior
  }
}

class MLAlbum extends Album {
  setup() {
    // Override to get info from algorithm
  }
}

class SharedAlbum extends Album {
  download() {
    // Override to get info from many sources
  }
}
```

在这种情况下`TypeScript`没有任何警告或提示说明`SharedAlbum`类中的`download`是，重写父类的函数。

将`noImplicitOverride`设置为`true`,子类重写的方法用`override`修饰，当父类发生变化时候，子类重写的方法会抛出错误，达到永远保持同步的效果。

如果将`noImplicitOverride`设置为`true`,子类重写方法的时候不能缺少`override`修饰符，如果确实会抛出错误。例如：

```javascript
class Album {
  setup() {}
}

class MLAlbum extends Album {
  override setup() {}
}

class SharedAlbum extends Album {
  setup() {}// error info: This member must have an 'override' modifier because it overrides a member in the base class 'Album'.
}
```

###### noImplicitReturns

noImplicitReturns：TypeScript 将检查函数中的所有代码是否都有返回值，当出现没有返回值的代码时候抛出错误（当函数的返回值为`void`时除外）,默认值为`false`
配置

```json
"noImplicitReturns":false
```

例如：

```javascript
function lookupHeadphonesManufacturer(color: "blue" | "black"): string {
  // error inf : Function lacks ending return statement and return type does not include 'undefined'.
  if (color === "blue") {
    return "beats";
  } else {
    ("bose");
  }
}
```

###### noImplicitThis
noImplicitThis：在任何隐式为`any`类型的时候，使用`this`，抛出错误（当严格模式时候默认值为`true`，其他情况默认值为`false`）
配置
```json
"noImplicitThis":true
```
例如：
`width`和`height`在`getAreaFunction`函数内的匿名函数中隐式类型是`any`
```javascript
class Rectangle {
  width: number;
  height: number;
 
  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }
 
  getAreaFunction() {
    return function () {
      return this.width * this.height;
      // error info： 'this' implicitly has type 'any' because it does not have a type annotation.
      // error info： 'this' implicitly has type 'any' because it does not have a type annotation.
    };
  }
}
```

###### noPropertyAccessFromIndexSignature
noPropertyAccessFromIndexSignature：此设置保证通过( obj.key) 语法访问字段和( obj["key"]) 来访问类型中声明的属性。当设置为`true`时访问一个不存在的属性时`TypeScript`会抛出一个错误。在调用对象属性时候，确保这个属性是存在的。
配置
```json
"noPropertyAccessFromIndexSignature":false
```
例如：设置为`false`，`TypeScript`将允许访问没有定义的属性
```javascript
interface GameSettings {
  // Known up-front properties
  speed: "fast" | "medium" | "slow";
  quality: "high" | "low";
 
  // Assume anything unknown to the interface
  // is a string.
  [key: string]: string;
}
 
const settings = getSettings();
settings.speed;
settings.quality;
// Unknown key accessors are allowed on
// this object, and are `string`
settings.username;
```
例如：设置为`true`，在`TypeScript`中访问对象未定义的属性时候，会抛出错误
```javascript
const settings = getSettings();
settings.speed;
settings.quality;
 
// This would need to be settings["username"];
settings.username;
// error info : Property 'username' comes from an index signature, so it must be accessed with ['username'].
```

###### noUncheckedIndexedAccess
noUncheckedIndexedAccess： `TypeScript` 有一种方法可以通过索引签名来访问对象的未知属性的值。
配置
```json
"noUncheckedIndexedAccess":false
```
例如：
```javascript
interface EnvironmentVars {
  NAME: string;
  OS: string;
  // Unknown properties are covered by this index signature.
  [propName: string]: string;
}

declare const env: EnvironmentVars;

// Declared as existing
const sysName = env.NAME;
const os = env.OS;
// os: string
// Not declared, but because of the index
// signature, then it is considered a string
const nodeEnv = env.NODE_ENV;
// nodeEnv: string
```
例如：当将`noUncheckedIndexedAccess`设置为`true`时，对象中未声明的属性值将会是`undefined`。
```javascript
declare const env: EnvironmentVars;
// Declared as existing
const sysName = env.NAME;
const os = env.OS;
// os: string
// Not declared, but because of the index
// signature, then it is considered a string
const nodeEnv = env.NODE_ENV;
// nodeEnv: string | undefined
```

###### noUnusedLocals
noUnusedLocals：有未使用的变量时，抛出错误
配置
```json
"noUnusedLocals": true
```
示例：
```javascript
const createKeyboard = (modelID: number) => {
  const defaultModelID = 23;
 // error info : 'defaultModelID' is declared but its value is never read.
  return { type: "keyboard", modelID };
};
```

###### noUnusedParameters
noUnusedParameters：有未使用的参数时，抛出错误
配置：
```json
"noUnusedParameters": true
```
示例：
```javascript
const createDefaultKeyboard = (modelID: number) => {
 // error info : 'modelID' is declared but its value is never read.
  const defaultModelID = 23;
  return { type: "keyboard", modelID: defaultModelID };
};
```

###### strict
strict：启用所有严格类型检查选项
配置
```json
"strict": true,
```

###### strictBindCallApply
strictBindCallApply：TypeScript是否对函数的`call`,`bind`,`apply`的入参进行强类型校验，默认值（如果`strict`为`true`则为`true`，其他情况下为`false`）
配置
```json
"strictBindCallApply":true
```
示例：
```javascript
// With strictBindCallApply on
function fn(x: string) {
  return parseInt(x);
}
 
const n1 = fn.call(undefined, "10");
 
const n2 = fn.call(undefined, false);
// error info : Argument of type 'boolean' is not assignable to parameter of type 'string'.
```
设置为`false`时候，表示可以是`any`类型
```javascript
// With strictBindCallApply off
function fn(x: string) {
  return parseInt(x);
}
 
// Note: No error; return type is 'any'
const n = fn.call(undefined, false);
```

###### strictFunctionTypes
strictFunctionTypes ：启用此配置后将会检查函数参数的数据类型。默认值（如果`strict`为`true`则为`true`，其他情况下为`false`）
配置
```json
"strictFunctionTypes":true
```
示例：
- 设置为`false`的时候
```javascript
function fn(x: string) {
  console.log("Hello, " + x.toLowerCase());
}
 
type StringOrNumberFunc = (ns: string | number) => void;
 
// Unsafe assignment
let func: StringOrNumberFunc = fn;
// Unsafe call - will crash
func(10);
```
随着`strictFunctionTypes`设置为`true`，错误就会显示出来
```javascript
function fn(x: string) {
  console.log("Hello, " + x.toLowerCase());
}
 
type StringOrNumberFunc = (ns: string | number) => void;
 
// Unsafe assignment is prevented
let func: StringOrNumberFunc = fn;
// error info : Type '(x: string) => void' is not assignable to type 'StringOrNumberFunc'.
// Types of parameters 'x' and 'ns' are incompatible.
// Type 'string | number' is not assignable to type 'string'.
// Type 'number' is not assignable to type 'string'.
```
这个功能呢在开发阶段可以提前帮我们检查语法上的错误包括DOM层次的一些不合理
```javascript
type Methodish = {
  func(x: string | number): void;
};
 
function fn(x: string) {
  console.log("Hello, " + x.toLowerCase());
}
 
// Ultimately an unsafe assignment, but not detected
const m: Methodish = {
  func: fn,
};
m.func(10);
```

###### strictNullChecks
strictNullChecks：启用严格的 null 检查，
- 当`strictNullChecks`为`false`时，`null`和`undefined`会被语法忽略，导致运行时报错。
- 当`strictNullChecks`为`true`时，`null`和`undefined`会有自己独特的类型，代码编译的时候就会提示错误。
配置
```json
"strictNullChecks":true
```
示例
例如用`find`函数不能保证查到对象，但是在后面对象中又用了对象的属性。
```javascript
declare const loggedInUsername: string;
 
const users = [
  { name: "Oby", age: 12 },
  { name: "Heera", age: 32 },
];
 
const loggedInUser = users.find((u) => u.name === loggedInUsername);
console.log(loggedInUser.age);
```
如果把`strictNullChecks`设置为`true`，编译的时候就会接受到一个错误（在使用`loggedInUser`前没有保证它是一定存在的）
```javascript
declare const loggedInUsername: string;
 
const users = [
  { name: "Oby", age: 12 },
  { name: "Heera", age: 32 },
];
 
const loggedInUser = users.find((u) => u.name === loggedInUsername);
console.log(loggedInUser.age);
// error info : object is possibly 'undefined'.
```
如下两个列子看起来`find`永远是可以得到值的
```javascript
// 当 strictNullChecks: true
type Array = {
  find(predicate: (value: any, index: number) => boolean): S | undefined;
};
// 当 strictNullChecks: false 时候 undefined 默认从类型中移除,
// 这个时候代码总是认为它能得到值的
type Array = {
  find(predicate: (value: any, index: number) => boolean): S;
};
```

###### strictPropertyInitialization
strictPropertyInitialization：定义的属性未赋默认值或者在构造函数中赋值时候会抛错误
配置
```json
"strictPropertyInitialization":true
```
示例
```javascript
class UserAccount {
  name: string;
  accountType = "user";
  email: string;
// error info : Property 'email' has no initializer and is not definitely assigned in the constructor.
  address: string | undefined;
 
  constructor(name: string) {
    this.name = name;
    // Note that this.email is not set
  }
}
```















##### 配置项详解

###### target

target：指定 ECMAScript 目标版本: 'ES3' (default), 'ES5', 'ES6'/'ES2015', 'ES2016', 'ES2017', or 'ESNEXT'

```json
"target": "es5",
```

###### module

module：指定使用模块: 'commonjs', 'amd', 'system', 'umd' or 'es2015'

```json
"module": "commonjs",
```

###### lib

lib：指定要包含在编译中的库文件

```json
"lib": [
  "dome",
  "es6"
],
```

###### allowJs

allowJs：指定是否允许编译 JS 文件，默认 false,即不编译 JS 文件

```json
"allowJs": true,
```

###### checkJs

checkJs：指定是否检查和报告 JS 文件中的错误，默认 false

```json
"checkJs": true,
```

###### jsx

jsx：指定 jsx 代码用于的开发环境:'preserve','react-native',or 'react

```json
"jsx": "preserve",
```

###### declaration

declaration：指定是否在编译的时候生成相的 d.ts 声明文件，如果设为 true,编译每个 ts 文件之后会生成一个 js 文件和一个声明文件，但是 declaration 和 allowJs 不能同时设为 true

```json
"declaration": true,
```

###### declarationMap

declarationMap：指定编译时是否生成.map 文件

```json
"declarationMap": true,
```

###### sourceMap

sourceMap：

```json
"sourceMap": true, // 生成相应的 '.map' 文件
```

{
"compilerOptions": {
'ES2016', 'ES2017', or 'ESNEXT'
"outFile": "./", // 将输出文件合并为一个文件
"outDir": "./", // 指定输出目录
"rootDir": "./", // 用来控制输出目录结构 --outDir.
"removeComments": true, // 删除编译后的所有的注释
"noEmit": true, // 不生成输出文件
"importHelpers": true, // 从 tslib 导入辅助工具函数
"isolatedModules": true, // 将每个文件作为单独的模块 （与 'ts.transpileModule' 类似）.

    /* 严格的类型检查选项 */
    "alwaysStrict": true,                  // 以严格模式检查每个模块，并在每个文件里加入 'use strict'

    /* 额外的检查 */

    /* 模块解析选项 */
    "moduleResolution": "node",            // 选择模块解析策略： 'node' (Node.js) or 'classic' (TypeScript pre-1.6)
    "baseUrl": "./",                       // 用于解析非相对模块名称的基目录
    "paths": {},                           // 模块名到基于 baseUrl 的路径映射的列表
    "rootDirs": [],                        // 根文件夹列表，其组合内容表示项目运行时的结构内容
    "typeRoots": [],                       // 包含类型声明的文件列表
    "types": [],                           // 需要包含的类型声明文件名列表
    "allowSyntheticDefaultImports": true,  // 允许从没有设置默认导出的模块中默认导入。

    /* Source Map Options */
    "sourceRoot": "./",                    // 指定调试器应该找到 TypeScript 文件而不是源文件的位置
    "mapRoot": "./",                       // 指定调试器应该找到映射文件而不是生成文件的位置
    "inlineSourceMap": true,               // 生成单个 soucemaps 文件，而不是将 sourcemaps 生成不同的文件
    "inlineSources": true,                 // 将代码与 sourcemaps 生成到一个文件中，要求同时设置了 --inlineSourceMap 或 --sourceMap 属性

    /* 其他选项 */
    "experimentalDecorators": true,        // 启用装饰器
    "emitDecoratorMetadata": true          // 为装饰器提供元数据的支持
