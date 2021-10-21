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

    "noFallthroughCasesInSwitch": true,    // 报告 switch 语句的 fallthrough 错误。（即，不允许 switch 的 case 语句贯穿）

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
    "strict": true,                        // 启用所有严格类型检查选项
    "noImplicitAny": true,                 // 在表达式和声明上有隐含的 any类型时报错
    "strictNullChecks": true,              // 启用严格的 null 检查
    "noImplicitThis": true,                // 当 this 表达式值为 any 类型的时候，生成一个错误
    "alwaysStrict": true,                  // 以严格模式检查每个模块，并在每个文件里加入 'use strict'

    /* 额外的检查 */
    "noUnusedLocals": true,                // 有未使用的变量时，抛出错误
    "noUnusedParameters": true,            // 有未使用的参数时，抛出错误
    "noImplicitReturns": true,             // 并不是所有函数里的代码都有返回值时，抛出错误

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
