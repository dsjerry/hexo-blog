---
title: Typescript 快速手册 2.0
toc: true
date: 2023-08-12 14:25:33
tags: [ts]
categories: 前端学习
thumbnail: https://z1.ax1x.com/2023/08/13/pPuHFwF.jpg
cover: https://z1.ax1x.com/2023/08/13/pPuHFwF.jpg
---

努力学习 🏞️🏞️

<!-- more -->

# 基础知识

## 启动

生成配置文件 _ts.config.json_

```shell
tsc --init
```

监听文件变化

```shell
tsc -w index.ts
```

安装 _ts-node_ 直接运行 ts 文件，不用先手动编译

```shell
npm install ts-node -g
```

## 类型、接口

### 基础

类型的级别（从顶级到低级排）

1. any, unknown
2. Object
3. Number, String, Boolean
4. number, string, boolean
5. 1, 'hello', false
6. never

其中：

- `any`: 任意类型，可给随便赋值
- `unknown`: 不知道的类型，只能给自己或者 any 赋值

`unknown`的对象是无法读取属性的

```ts
let person: unknown = { name: "Jerry", age: 18 };
person.name; // ❌ 这里会报错
```

- 所以`unknown`类型会比较安全，当不知道用什么类型的时候可以选它

---

`Object`, `object`, `{}`的区别：🙃

▶️ Object：在原型链上，可包含所有

```ts
let test: Object = 1;
test = "hello";
test = false;
// ...
```

▶️ object：代表非原始类型的类型（引用类型），比较常用于*泛型约束*

▶️ {}：相当于`new Object`，和`Object`一样，**但是变量不能复制**

```ts
let person: {} = { name: "Jerry" };
person.age; // ❌ 这里会报错
```

---

### 接口

- 两个重名的 interface 会合并起来
- 多行时，属性之间可以用`;`，也可以不用

`索引签名(index signature)`的可能使用场景：后端返回对象，但是前端只想要其中的几个属性

```ts
interface Person {
  readonly id: number;
  name: string;
  age: number;
  [propName: string]: any;
}
```

```ts
interface Fn {
  (nameL string): number[]
}

const fn: Fn = (name:string) => [1]
```

> interface 里面有唯一 id 或者函数，可以用`readonly`限制一下（在对应 key 前面加上）

_数组接口_ 的几种使用方式

```ts
let name1: string[] = ["Tom", "Jerry"];

let name2: Array<string> = ["Tom", "Jerry"];

interface Name {
  name: string;
}

let name3: Name[] = [{ name: "Tom" }, { name: "Jerry" }];
```

二维数组或者更多

```ts
let name1: string[][] = [[{ name: "Tom" }], [{ name: "Jerry" }]];
let name2: Array<Array<string>> = [[{ name: "Tom" }], [{ name: "Jerry" }]];
```

如果数组里面啥都有。直接用`any`或者用`元组`

```ts
let name: any[] = [1, "qaq"];
let name: [number, string] = [1, "qaq"];
```

_剩余参数_：有属于自己的接口

```ts
function func(...args: string[]) {
  console.log(arguments); // 类型是 IArgments
}
```

### 类型别名

就是用`type`来指定某个类型

```ts
type s = string;

type strArr = string[];
```

`interface`不能使用联合类型，但可以使用`extends`

- 遇到重复的会合并

`type`没有继承，但可以使用联合类型，如`type s = string | number`

- 遇到重复的不会合并

🤤`type`的别的用法

```ts
// 左边的值会作为右边类型的子类型
type num = 1 extends number ? 1 : 0;
```

- 这里是 1
- type 中的 extends 是包含的意思

---

### 类型断言、交叉类型、联合类型

```ts
let fn = function (type: number): boolean {
  return !!type;
};
```

- `!0` 是 `true`，再`!`就是`false`了，这时类型就从`number`变成了`boolean`，但是表达的意义还是对、错（例如：有时后端返回数字而不是 true false 的时候）

联合类型，`interface`的话，使用一个`&`来联合，是一个。

类型断言，就是那个`as`，可以用来欺骗 TypeScript，如果有错的话，编译的时候还是会报错的。另外的写法是`<类型>变量`

```ts
(window as any).abc = 123;
```

- any 可以 断言为任何类型

## 函数

```ts
function add1(a: number, b: number): number;
const add2 = (a: number, b: number): number => {};
```

> 对于函数的一个参数：默认值 和 可选 不能一起用

在定义函数的时候，**第一个参数**可以定义`this`，用来指定函数当前所处的对象

- 在调用的时候，`this`不用传递

```ts
interface Obj {
  user: number[]
  add: (this.Obj, num:number)=>void
}

let obj: Obj = {
  user: [1,2,3],
  add(this.obj, num:number) {
    this.user.push(num)
  }
}

obj.add(6)
```

- 这个语法在 `js` 中不能使用

**函数重载**

可能场景

```ts
let user: number[] = [1, 2, 3];
// 如果传进来的是number类型的数组，那就添加
function findNum(add: number[]): number[];
// 如果传进id就单个查询
function findNum(id: number): number[];
function findNum(ids: number | number[]): number[] {
  if (typeof ids === "number") {
    return user.filter((v) => v === ids);
  } else if (Array.isArray(ids)) {
    user.push(...ids);
    return user;
  } else {
    return user;
  }
}
```

## class

`implements`：用来实现接口，约束类的结构

```ts
interface Options {
  el: string | HTMLElement;
}

// 定义接口约束
interface VueClass {
  options: Options;
  init(): void;
}

class Vue implements VueClass {
  options: Options;
  constructor(options: Options) {
    this.options = options;
  }
  init(): void {
    console.log("init");
  }
}
```

`extends`：用来继承类

- extends 需要写在 implements 前面，因为 extends 会继承 implements 的接口
- super 原理是父类的`prototype.constructor.call`
  - 里面的参数是父类的参数如果有个话，需要向里面传递实参
  - 可以直接通过`super.`来调用父类的方法或者属性

```ts
class Dom {
  createElement(el: string) {
    return document.createElement(le);
  }
  setText(el: HTMLElement, text: string) {
    el.textContent = text;
  }
  render() {}
}

class Vue extends Dom implements VueClass {
  options: Options;
  constructor(options: Options) {
    // 调用父类的构造函数，需要写在最上面
    super();
    this.options = options;
  }
  init(): void {
    this.render();
  }
}
```

其他关键字：

- `readonly`：只能在声明属性的时候或者构造函数中赋值
- `private`: 类内部使用
- `protected`: 类内部和子类使用
- `public`: 类内部、子类、类外部都可以使用
- `static`: 静态属性，可以直接通过类名来调用。
  - **只能调用静态的属性和方法**，因为不需要实例化，自然就不在实例上面了
  - 想要数据的话，可以把数据作为函数的参数传进来

`get`、`set`: 可拦截属性的读取和赋值操作

```ts
class Vue {
  private _el: string | HTMLElement;
  constructor(el: string | HTMLElement) {
    this._el = el;
  }
  get el() {
    return this._el;
  }
  set el(val) {
    this._el = val;
  }
}

let vue = new Vue("div");
vue.el = "span";
console.log(vue.el);
```

`abstract`：抽象类，不能被实例化，只能被继承 🤤

```ts
abstract class Dom {
  abstract createElement(el: string): HTMLElement;
  abstract setText(el: HTMLElement, text: string): void;
  abstract render(): void;
}

class Vue extends Dom {
  options: Options;
  constructor(options: Options) {
    super();
    this.options = options;
  }
  createElement(el: string): HTMLElement {
    return document.createElement(el);
  }
  setText(el: HTMLElement, text: string): void {
    el.textContent = text;
  }
  render(): void {
    let el = this.createElement(this.options.el);
    this.setText(el, "hello");
  }
}
```

- 下面实现的叫派生类，抽象类中的东西，派生类必须实现

## 内置对象

👉🙃 有规可循

**ECMA**

```ts
let num: Number = new Number(1);
let date: Date = new Date();
let xhr: XMLHttpRequest = new XMLHttpRequest();
```

- 一般是 _new_ 什么，就是什么类型

**DOM**

一般是什么`HTMLxxxElement`

section、header 这种，会归为 `HTMLElement`

`querySelectorAll`这种，就是`NodeList`，如果里面是有几个的，不固定的，使用`NodeListOf`

```ts
let div: NodeListOf<HTMLDivElement | HTMLElement> =
  document.querySelectorAll("div");
```

**BOM**

```ts
let local: Storage = localStorage;
let lo: Location = location;
let prop: Promise<number> = new Promise((resolve, reject) => {
  resolve(1);
});

// cookie 是字符串哇
let coo: string = document.cookie;
```

## 元组、枚举

### 元组

```ts
let arr: [number, boolean] = [1, false];

arr.push(null); // ❌ 虽然长度过了，但是这个元组会被推断为联合类型
arr.push(1); // ✅ 所以只能装上面指定的类型
```

其他限制：只读、指定名称、可选

```ts
let arr: readonly [x: number, y?: boolean] = [1];
```

使用场景：例如一个*excel*表格

```ts
let excel: [string, string, number][] = [
  ["小满", "女", 18],
  ["小满", "女", 18],
  ["小满", "女", 18],
];
```

❔ 要获取元组中某一个的类型

```ts
type first = (typeof arr)[0];
```

❔ 要获取它的长度

```ts
type first = (typeof arr)["length"];
```

### 枚举

```ts
enum Color {
  red,
  green,
  blue,
}

console.log(Color.red); // 0 ,后面的是 1、2
```

给它指定值

```ts
enum Color {
  red = 1,
  green,
  blue,
}

console.log(Color.red); // 1 ,后面的是 2、3
```

普通枚举和使用`const`定义的枚举

```ts
enum Test {
  red = 1,
  green,
}

const enum Test {
  red = 1,
  green,
}
```

会被编译成一个对象

```js
var Test;
(function (Test) {
  Test[(Test["red"] = 1)] = "red";
  Test[(Test["green"] = 2)] = "green";
})(Test || (Test = {}));
```

加了`const`的，会直接编译称一个常量 ??

> 但是我的编译出一个空白文档。typescript 5.1.6

👉 反向映射 通过 value 获得 key，正向反之

```ts
enum Types {
  success,
}

let success: number = Types.success;

console.log(success); // 0

let key = Types[success]; // success
```

## never、symbol

### never

永远无法到达的类型 🙃

```ts
type A = string & number;
```

- `type A`就是一个`never`类型

如果在联合类型中使用到`never`，是会直接被忽略掉的

常用的场景：兜底逻辑，直接代码报错，找到问题所在

```ts
type A = "唱" | "跳" | "rap";

function kun(value: A) {
  switch (value) {
    case "唱":
      break;
    case "跳":
      break;
    case "rap":
      break;
    default:
      const error: never = value;
      break;
  }
}
```

- 默认值得时候，`value`会是`never`类型

这样的话代码没有问题。然后以后要改的时候，突然想在`A`中再加上类型`'篮球'`，这样代码就会报错，因为这时候`default`就会是一个`string`篮球。`string`无法赋值给`never`

这样的话防止代码多了，都不知道错误在哪

### symbol

- 可避免对象中重复出现的*key*

> for in、Object.keys 不能读取到 symbol

每一个都是**唯一的**

```ts
let a1: symbol = Symbol(1);
let a2: symbol = Symbol(1);

console.log(a1 == a2); // false
```

😅 就是想要得到两个相等的？？？

```ts
console.log(Symbol.for("xiaoman") == Symbol.for("xiaoman"));
```

- 使用`.for`会在全局的*symbol*中找有没有注册过这个*key*

  - 有的话**直接**拿来用
  - 没有的话，先创建一个

`Object.getOwnPropertyNames`：获取对象中普通的属性名

`Object.getOwnPropertySymbols`：获取对象中 symbol 类型的属性

`Reflect.ownKeys`：上面两个都可同时获取

# 配置、声明

## 命名空间

_ts_ 和 _es6_ 中，包含顶级`import`或者`export`的文件会被当成一个模块，反之文件里面的内容被视为全局可见

这样的话，在两个不同的文件中声明相同的变量，**会报错说变量已经存在**

📄index.ts

```ts
const a = 1;
```

📄index2.ts

```ts
const a = 1;
```

要解决这个问题，可以将这个文件变成一个模块

```ts
export const a = 1;
```

除了这个方法，还可以使用 👉**命名空间**

📄index.ts

```ts
namespace A {
  export const a = 1;
}
```

📄index2.ts

```ts
namespace B {
  export const a = 1;
}
```

使用的时候，和对象一样

```ts
console.log(A.a);
console.log(B.a);

namespace B {
  export const a = 1;
  export namespace C {}
}
```

- 命名空间还可以嵌套，在里面`export namespace`
- 最外层导出的命名空间，通过`import`来导入，`import {xx} from './index.ts'`
- 还可以这样导入`import x = B.C`

重名的命名空间会合成一个

## 三斜线指令

- 常见的是用到引入依赖

```ts
/// <reference path="index2.ts" />
```

在编译的时候开启`removeCommoent`的选项，编译后的文件就会移除掉这个引用声明的注释

## 声明文件

`xx.d.ts`文件

有些库安装的时候没有声明文件，可以尝试安装一下，也有可能是真的没有声明文件

```shell
npm i --save-dev @types/xxx
```

- `@types`开头是一个规范来的

没有的话，自己手写一下咯。要为谁写声明文件，就以这个包的名字为声明文件的名字，比如*express* 没有声明文件，那就应该是`express.d.ts`

```ts
declare module "express" {
  interface Router {}
  interface App {
    use(): void;
  }
}
```

---

```ts
declare const a: number;
declare function qa(param: any);
```

> 如果文件里面没有`import`或者`export`的话，也需要指定一下`export{}`来使得 ts 能把当前文件识别为模块

# 泛型

## 基础

**在函数中使用**

代码逻辑一样，但想返回不同的类型。如果没有使用泛型，那只能赋值两份代码，不方便，所以要使用动态类型

```ts
function func1(a:number, b:number)Array<number> {
  return [a, b]
}

function func1(a:string, b:string)Array<string> {
  return [a, b]
}
```

使用泛型

```ts
function func<T>(a: T, b: T): Array<T> {
  return [a, b];
}

// number
func(1, 2);

func<number>(1, 2); // 🎇 完整的调用形态应

// 都行
func(false, false);
```

- 不用完整的调用形态，因为可以类型推断

在`type`中使用

```ts
type A<T> = string | number | T;

let a: A<boolean> = true;
```

在`interface`中使用

```ts
interface Obj<T> {
  name: T;
}

let obj: Obj<string> = {
  name: "zs",
};
```

使用多个泛型

```ts
function func<T, K>(a: T, b: K): Array<T | K> {
  return [a, b];
}

func(1, false);
```

泛型也可以使用默认值：函数不传值，泛型使用默认泛型

```ts
function func<T = number, K = string>(a: T, b: K): Array<T | K> {
  return [a, b];
}

func();
```

## 实际应用

对接一下请求接口的数据

```ts
const axios = {
  get<T>(url: string): Promise<T> {
    return new Promise((resolve, reject) => {
      let xhr: XMLHttpRequest = new XMLHttpRequest();
      xhr.open("GET", url);
      xhr.onreadystatechange = () => {
        // 0~4, 4 代表 complete
        if (xhr.readyState === 4 && xhr.status === 200) {
          resolve(JSON.parse(xhr.responseText));
        }
      };
      // 必须滴
      xhr.send(null);
    });
  },
};

interface Data {
  message: string;
  code: number;
}

axios.get<Data>("./data.json").then((res) => {
  console.log(res);
});
```

- 传入`Data`，返回`Promise<Data>`，这样的话有约束，而且`.then()`也有提示

## 泛型约束

泛型有时候太过灵活

```ts
function add<T>(a: T, b: T) {
  return a + b; // ❌ 报错，但是这确实正确的行为
}

add(undefined, undefined); // 这两个肯定是不能相加的
```

加上约束

```ts
function add<T extends number>(a: T, b: T) {
  return a + b;
}
```

```ts
interface Len {
  length: number;
}

function fn<T extends Len>(a: T) {
  a.length;
}

fn("111"); // ✅
fn([1, 2, 3]); // ✅
fn(123); // ❌
fn(false); // ❌
```

```ts
let obj = {
  name: "xm",
  age: 18,
};

// obj 约束为 object 类型，key 约束为 obj 中的某一个 key 值
function ob<T extends object, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

ob(obj, "name");
```

`keyof`是将对象的类型的*key* 作为一个联合类型

- 注意不是直接作用在对象身上，因为对象是对*值* 的引用，而`keyof`的作用对象是*类型*

可以这样使用

```ts
type Key = keyof typeof obj;
```

🙃`keyof`的高级用法

```ts
interface Data {
  name: string;
  age: number;
  gender: string;
}

// 要使 Data 中的每一个都变成可选，除了给每个加上 ? ，还可以这样
// 类似于 for(let key in obj)
type Options<T extends object> = {
  [Key in keyof T]?: T[Key];
  // readonly [Key in keyof T]: T[Key]
};

type B = Options<Data>;
```

这时候`type B`会是

```ts
type B = {
  message?: string | undefined;
  code?: number | undefined;
  name?: string | undefined;
  age?: number | undefined;
  gender?: string | undefined;
};
```

其实使用`Partial`也可以得到相同的结果

```ts
type C = Partial<Data>;
```

`readonly`的话，就使用`Readonly`

# 装饰器

> 需要在配置文件中打开 `experimentalDecorators`和`emitDecoratorMetadata`

装饰器有：

1. 类装饰器
2. 属性装饰器
3. 参数装饰器
4. 方法装饰器
5. 装饰器工厂

类装饰器，在不破坏原有内容的情况下（或者说不知道原来有什么的情况下）增加内容

```ts
const Base: ClassDecorator = (target) => {
  target.prototype.xiaoman = "小满";
  target.prototype.fn = () => {
    console.log("我是憨憨");
  };
};

@Base
class Http {
  //...
}
const http = new Http() as any;
http.fn();
```

- `target`是一个构造函数

还有另一种写法是不用`@Base`

```ts
class Http {
  //...
}
const http = new Http() as any;
Base(Http);
```

❔❔ 如果要向`@Base`传递参数呢

- `target`是默认存在的，好像事件监听的事件对象一样
- 可以使用*函数柯里化* 事先参数传递

就是再套一层函数

- 外面一层函数接收`@Base`传过来的参数
- 里一层函数接收默认存在的参数

```ts
const Base = () => {
  const fn: ClassDecorator = (target) => {
    target.prototype.xiaoman = "小满";
    target.prototype.fn = () => {
      console.log("我是憨憨");
    };
  };
  return fn;
};

@Base("xb")
class Http {
  //...
}
```

- 这一招叫做：装饰器工厂、函数柯里化、闭包

函数装饰器

```ts
const Get = (url: string) => {
  const fn: MethodDecorator = (target, key, descriptor: PropertyDescriptor) => {
    axios.get(url).then((res) => {
      descriptor.value(res.data);
    });
  };
  return fn;
};

@Base("xb")
class Http {
  @Get("https://xxxx.com/")
  getList(data: any) {
    console.log(data);
  }
}
```

- 有三个参数，装饰器自己的

啥也不用做，直接调用`@Get()`传递个地址过去就行了

# 类型兼容

鸭子类型：两个类型有共通的地方

协变：子类型可以完全覆盖掉主类型（子类型可以多内容，但是不能少内容）

```ts
// 主类型
interface A {
  name: string;
  age: number;
}

// 子类型
interface B {
  name: string;
  age: number;
  sex: string;
}

let a: A = {
  name: "xb",
  age: 18,
};

let b: B = {
  name: "xh",
  age: 19,
  sex: "man",
};

// 协变 （可以赋值）
a = b;
```

逆变：一般在函数上出现（和协变是反过来的），类似于`b = a`

```ts
let fna = (params: A) => {};
let fnb = (params: B) => {};

fna = fnb; // ❌
fnb = fna; // ✅
```

- （双向协变）配置文件开启`strictFunctionTypes`设为`false`，第一个赋值可以成立
- 第二个之所以成立，因为赋值完了后`fnb`就是`fna`

# utility

`Record<Keys, Type>`

组建一个对象类型，这个对象类型的 key 是 传入的 _Keys_ ，对象的值是传入的 _Type_

```ts
interface CatInfo {
  age: number;
  breed: string;
}

type CatName = "miffy" | "boris" | "mordred";

const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};

cats.boris;
```

---

整理来源：📄 我的笔记、📺 bilibili`小满zs`

<!--
<div style="display: flex;">
<span style="margin:auto; font-size: 70px; color: white; background-color: #3178c6;padding: 20px">Typescript <span style="font-size: 0.7em;color: #282c34">快速手册 2.0</span></span>
</div> -->
