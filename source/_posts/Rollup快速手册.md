---
title: Rollup快速手册
toc: true
date: 2023-09-03 16:10:14
tags: [打包]
categories: 前端学习
thumbnail: https://z1.ax1x.com/2023/09/13/pPRmJGd.png
cover: https://z1.ax1x.com/2023/09/13/pPRmJGd.png
---

【🚧 施工中】

[ ] 相关插件案例

<!-- more -->

# 介绍

## 常见词汇

**捆绑包（bundle）**

是指将多个源代码文件或模块组合成一个单独的文件或一组文件的过程

- 减小文件数量，更有效在浏览器或者其他环境中加载和执行
- 减少网络请求

用于部署在生产环境，一般会经过压缩和混淆

## rollup 概念

1. 向未来兼容
2. 除屑优化（Tree-shaking）

**向未来兼容**

虽然`ES6`新特性在现代浏览器中已经稳定，但在 Nodejs 中还未完全落地

Rollup 的作用就是可以使用新的特性，然后打包成`CommonJS`或`AMD`等格式，即使以后这些特性在 Nodejs 中完整支持，现在的代码也不需要改动（向未来兼容）

**除屑优化**

静态分析导入的代码，然后去除没有使用的代码，减少打包后的体积

📄 原始文件

```js
const utils = require('./utils');`，
```

但使用的时候只用到`utils`中的`ajax`方法，打包出来会是：

📄 打包结果

```js
import { ajax } from "./utils";
```

## 安装

全局安装

```shell
npm install rollup --global
```

本地安装

```shell
npm install rollup --save-dev
```

`package.json`中的`scripts`节点

```json
{
  "scripts": {
    "build": "rollup --config"
  }
}
```

# 基本使用

## 命令行

```js
// src/main.js
import foo from "./foo.js";
export default function () {
  console.log(foo);
}
```

```js
// src/foo.js
export default "hello world!";
```

```shell
rollup src/main.js -f cjs
```

- `-f`全称`--format`，指定为*CommonJS*
- 没有指定输出文件，默认以`stdout`输出

得到输出内容

```js
"use strict";

const foo = "hello world!";

const main = function () {
  console.log(foo);
};

module.exports = main;
```

使用`-o`指定输出文件

```shell
rollup src/main.js -o bundle.js -f cjs
```

## 配置文件

```js
// rollup.config.mjs
import xxplugin from "xxplugin";

export default {
  input: "src/main.js",
  output: {
    file: "bundle.js", // 还可以使用 dir
    format: "cjs",
  },
  plugins: [xxplugin()],
};
```

- 如果输出一个以上的文件，需要使用`output.dir`

*CJS*的话，使用`module.exports = {/* config */}`

使用`-c`（`--config`）指定配置文件

```shell
rollup -c
```

- 会自动读取`rollup.config.js`或`rollup.config.mjs`，如果没有，会报错

可以在命令行随便覆盖配置文件的配置

```shell
rollup -c -o bundle-2.js # `-o` 等价于 `--file`（曾用名为 "output"）
```

🎇 一般会使用`defindConfig`函数，这样会有代码提示

```js
// rollup.config.js
import { defineConfig } from "rollup";

export default defineConfig({
  /* 你的配置 */
});
```

### mjs 和 cjs

默认就是`cjs`，即使是`.js`文件。如果要使用`mjs`，需要在`package.json`中配置：

```json
{
  "type": "module"
}
```

- 使用`cjs`的话，`__dirname`可用

### TypeScript

`@rollup/plugin-typescript`插件可以让 Rollup 支持 TypeScript

- 配置文件`.ts`
- `tsconfig.json`中的`include`字段包含配置文件

在 _TypeScript_ 中支持使用`RollupOptions`类型

```ts
// rollup.config.ts
import type { RollupOptions } from "rollup";

const config: RollupOptions = {
  /* 你的配置 */
};
export default config;
```

## 注意事项

> `import.meta.url`的路径是当前文件的路径，而不是`process.cwd()`的路径

### 路径使用

在 CommonJS 中，经常会用到`__dirname`，但是在 Rollup 原生 ES 模块中，是没有这个的，推荐使用`import.meta.url`来获取

```js
// rollup.config.js
import { fileURLToPath } from 'node:url'

export default {
  ...,
  // 为 <currentdir>/src/some-file.js 生成绝对路径
  external: [fileURLToPath(new URL('src/some-file.js', import.meta.url))]
};
```

### 导入 pakage.json

例如自动将依赖项标记为`external`

在`node 17.5+`中，可使用导入断言

```js
import pkg from "./package.json" assert { type: "json" };

export default {
  // Mark package dependencies as "external". Rest of configuration
  // omitted.
  external: Object.keys(pkg.dependencies),
};
```

在旧版`node`中，可以使用`@rollup/plugin-json`。或者使用 node 提供的`createRequire`：

```js
import { createRequire } from "node:module";
const require = createRequire(import.meta.url);
const pkg = require("./package.json");

// ...
```

或者直接使用`fs`模块读取文件内容

## 使用插件

<a href="https://github.com/rollup/awesome" target="_blank">🌎 Awesome Rollup</a>

### 导入现有的 CommonJS 模块

```shell
npm install @rollup/plugin-commonjs --save-dev
```

- 需要 node14+

### 从 JSON 文件中导入数据

```shell
npm install @rollup/plugin-json --save-dev
```

```js
// src/main.js
import { version } from "../package.json";

export default function () {
  console.log("version " + version);
}
```

### 输出插件

- 等 Rollup 主要分析完了之后，再对代码进行别的操作，比如压缩加密之类的

```shell
npm install @rollup/plugin-terser --save-dev
```

👉 对于不同的输出格式，可以使用不同的插件

```js
// rollup.config.mjs
import json from "@rollup/plugin-json";
import terser from "@rollup/plugin-terser";

export default {
  input: "src/main.js",
  output: [
    {
      file: "bundle.js",
      format: "cjs",
    },
    {
      file: "bundle.min.js",
      format: "iife",
      name: "version",
      plugins: [terser()],
    },
  ],
  plugins: [json()],
};
```

`iife`格式是将变量放到一个自动执行的函数中，这样就不会污染全局变量了

```js
var version = (function () {
  "use strict";
  var n = "1.0.0";
  return function () {
    console.log("version " + n);
  };
})();
```

# 配置选项

## external

类型：`(string | RegExp)[]| RegExp| string| (id: string, parentId: string, isResolved: boolean) => boolean`

需要排除在打包之外的模块

- 这些模块不会出现在打包后的代码中
- rollup 会生成一个`require`或`import`语句，让运行时去加载这些模块，以避免不必要的依赖项重复打包。
- 部署到生产环境时，需要确保`external`指定的这些模块已经存在

**匹配方式：**

1. 外部依赖名称

如果将`import dependency.js`标记为外部依赖，那么就传递`dependency.js`；`import "dependency"`的话，传递`dependency`

2. 解析过的模块 ID（疑问）

解析过的模块 ID，例如文件的绝对路径，或者`node_modules`中的模块

如果通过正则表达式`/node_modules/`匹配

3. 自定义函数

三个参数可使用：`(id, parent, isResolved) => boolean`

- `id`: 模块的 ID
- `parent`: 引入这个模块的模块 ID
- `isResolved`: 模块是否已经被插件解析，比如用`@rollup/plugin-node-resolve`插件解析过了

```js
// rollup.config.js
export default {
  external: (id) => {
    // 排除所有以 'react' 开头的模块
    return id.startsWith("react");
  },
  // 其他配置选项...
};
```

## input

类型：字符串、字符串数组、`{[entryName]: string}`

打包的入口，例如`main.ts`、`index.js`

如果传过来的是一个数组，或者是上面的第三个类型，打包的产物也是分开的，除非输出的时候指定了`output.file`。那输出的文件名会跟随`output.entryFileNames`

## output.file 和 output.dir

类型：字符串

`output.file`指定输出的文件，是一个具体文件；如果需要输出多个文件，需要使用`output.dir`

# 案例

## 打包 TypeScript

需要提前安装的有`typescript` 和`tslib`

```shell
npm install @rollup/plugin-typescript --save-dev
```

插件默认使用`tsconfig.json`中的配置，也可以在配置文件中指定

```js
/// ...
export default {
  input: "./main.ts",
  output: {
    dir: "output",
    format: "cjs",
  },
  plugins: [
    typescript({
      compilerOptions: { lib: ["es5", "es6", "dom"], target: "es5" },
    }),
  ],
};
```

> 其他配置错误根据报错提示修改就好了
