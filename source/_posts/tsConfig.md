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
**当只有少量文件并且不需要使用[glob](https://github.com/isaacs/node-glob)来匹配文件时，这很有用。如果需要，请使用include。**

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

```json
{
  "compilerOptions": {
    // 配置项.....
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
allowJs：指定是否允许编译JS文件，默认false,即不编译JS文件
```json
"allowJs": true,
```

###### checkJs
checkJs：指定是否检查和报告JS文件中的错误，默认false
```json
"checkJs": true,
```

###### jsx
jsx：指定jsx代码用于的开发环境:'preserve','react-native',or 'react
```json
"jsx": "preserve",
```

###### declaration
declaration：指定是否在编译的时候生成相的d.ts声明文件，如果设为true,编译每个ts文件之后会生成一个js文件和一个声明文件，但是declaration和allowJs不能同时设为true
```json
"declaration": true,
```

###### declarationMap
declarationMap：指定编译时是否生成.map文件
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
    "noFallthroughCasesInSwitch": true,    // 报告 switch 语句的 fallthrough 错误。（即，不允许 switch 的 case 语句贯穿）

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
