---
title: TypeScript快速手册
toc: true
date: 2021-10-07 11:07:15
tags: [js, ts]
categories: 前端学习
thumbnail: https://z3.ax1x.com/2021/10/07/5pKt6e.png
cover: https://z3.ax1x.com/2021/10/07/5pKt6e.png
---

一些基本使用笔记
一些注意点，不全

<!-- more-->

# 使用

1. 在*ts*文件中如果没有`typescript`语法代码，是可以直接在*html*文件中引用并且运行的
2. ts 中使用的关键字`let`在编译成 js 的时候会变成`var`

# 变量

如果声明变量的时候直接赋值，TS 可以自动检测变量的类型（即使不指定）

```tsx
let xm = '小明';
// 此时 xm 的类型是 string，赋予其他类型的值会报错
xm = 18;  ❌
```

可以在声明的时候使用字面量指定类型，后面就不能改了（类似于常量）

```tsx
let xm: 'xm';
xm = 'zs';  ❌
xm = 'xm';  ✔️
```

可以使用 `|`来声明多个类型（联合类型）

```tsx
let xm: 'male' | 'female'
xm = 'male'
xm = 'female'

let zs: boolean | string
zs = true
zs = 'zs'
```

在声明变量时不指定类型，则默认是`any`类型，也可以指定`any`类型，但是不好。必要时，使用`unknown`类型来代替`any`类型

`unknown`在定义的时候和`any`一样，可以随便值，**但是不能将自己的值直接赋给别的变量**（除非对方是 any）。

```tsx
let xm: unknown;
xm = 18;  xm = true;  xm = 'xm';
let zs: string;
zs = xm;  ❌
// 对于unknown类型，要想赋值，先进行判断
if (typeof xm === 'string') {
    zs = xm;
}
```

**类型断言**，用来告诉解析器变量的实际类型。有两种写法：

1. 变量 as 类型
2. <类型>变量

```tsx
zs = xm as string
zs = <string>xm
// 如果两个变量的类型不一样，则报错
```

- 在`jsx`中只能使用`as`关键字

`never`类型，表示永远不会有返回值（如抛出异常）

```tsx
function err(): never {
  throw new Error('错错错...')
}
```

- 变量也可以是`never`，即使是`any`也不能赋值给`never`
- 和`void`的区别是，`void`是没有返回值，不过可以做其他事情，`never`里面是无法到达的事情，如死循环

在声明对象的时候可以指定里面的属性，属性后面加`?`表示当前属性可选

```tsx
let xm: {name: string, age?: number};
xm = {name: 'xm'};  ✔️
```

也可以指定任意类型的属性

```tsx
let xm: { name: string; [prop: string]: any }
xm = { name: 'xm', age: 18, gender: 'male' }
```

**数组类型**，有两种方式：

1. 变量: 类型[]
2. 变量: Array<类型>

```tsx
let array_1: string[]
array = ['zs', 'ls', 'xm']
let array_2: Array<number>
array_2 = [1, 2, 3]
```

**元组**(tuple)，固定长度的数组：

1. 变量: [类型, 类型, 类型]

```tsx
let array: [string, number, boolean]
array = ['xm', 18, true]
```

**枚举**(enum)：

```tsx
enum Gender {
  Male = 0,
  Female = 1
}
let xm: { name: string; Gender: Gender }
xm = { name: 'xm', grnder: Gender.Male }
```

合并

```tsx
let xm: { name: string } & { age: number }
xm = { name: 'xm', age: 18 }
```

类型别名，用户可以创建自己想要的类型，然后给它取个名字便于调用

```tsx
type myType = 1 | 2 | 3 | 4 | 5;
let age: myType;
age = 1;  ✔️
age = 6;  ❌
```

# 类

其中主要有：_属性_ 和*方法*

## 基本

直接定义的属性是实例属性，需要实例化后访问。`static`为静态属性，通过类直接访问。`readonly`表示只读属性，可以单独使用，也可在`static`后面使用

```tsx
class Person {
  name: string = 'xm'
  static age: number = 18
  readonly gender: string = 'male'
  static readonly status: string = 'good'
}
let xm = new Person()
xm.name
Person.age
```

## 构造函数

之所以称为构造函数，是因为实在对象创建时（实例化）调用的

```tsx
class Dog {
  name: string
  age: bumber
  constructor(name: string, age: number) {
    // 其中this指向当前实例
    ;(this.name = name), (this.age = age)
  }
  bark() {
    console.log('wang~wang~~')
  }
}
const dog_1 = new Dog('tom', 2)
```

## 继承

```tsx
class LittleDog extends Dog {
  // 重写
  bark() {
    console.log('hahaha')
  }
}
```

- 继承会获得父类的所有属性和方法
- 在子类中有和父类相同名字的方法，就叫做**重写**

### super

在继承的时候，子类想继承父类的属性，又想添加另外的属性，就得用`super()`来**调用**父类的构造函数，并且传入想要继承的属性

```tsx
class LittleDog extends Dog {
  kind: string
  constructor(name: string, kind: string) {
    super(name)
    this.kind = kind
  }
}
```

## 抽象类

`abstract`开头，抽象类**不能**用于创建对象，专门用来被继承的

抽象类中有抽象方法，抽象方法必须在抽象类中

```tsx
abstract class Animal {
    name: string;
    constructor(name: string) {
        this.name = name
    }
    abstract hello():void {}
}
class Dog extends Animal {
    hello() {
        console.log('wangwang');
    }
}
class Cat extends Animal {
    hello() {
        console.log('miaomiao');
    }
}
const fish = new Animal();  ❌
const dog = new Dog();  ✔️
dog.hello();
```

## 接口

接口用来定义一个类的结构，规定里面要包含的内容。接口可以重复声明，不能有实际的值

```tsx
interface theInter {
  name: string
  age: number
}
interface theInter {
  gender: string
  hello(): void
}
```

```tsx
const obj: theInter = {
  name: 'xm',
  age: 18,
  gender: 'male',
  hello() {
    console.log('hello')
  }
}

class Male implements theInter {
  name: string
  age: number
  gender: string
  constructor(name: string, age: number, gender: string) {
    ;(this.name = name), (this.age = age)
    this.gender = gender
  }
  hello() {
    console.log('hello')
  }
}
```

- 在对象中用接口时，接口可以当成一个类型，这是`interface`关键字可以和`type`关键字互换

## 属性封装

```tsx
class Person {
  public _name: string
  private _age: number // 只能在类内部访问
  protected _gender: string // 在类内部和其子类内部访问
  constructor(name: string, age: number) {
    this._name = name
    this._age = age
  }
  // getter、setter存取器
  getName() {
    return this._name
  }
  setName(value: string) {
    this._name = value
  }
}
```

在 TS 中，存取器有另外的写法

```tsx
get name() {
    return this._name;
}
set name(value: string) {
    this._name = value
}
```

这样写，在实例化之后，和调用属性时一样的写法

```tsx
const xm = new Person('xm', 18, 'male')
xm._name // 可以获得name属性，通过实例属性
xm.name // 可以后的name属性，通过存取器
xm.name = 'zs' // 设置属性值，通过存取器
```

## 泛型

在声明函数或者时类的时候，如果*不知道用什么类型*，就可以使用泛型。等调用的时候再根据参数指定

```tsx
function fn<T>(a: T): T {
  return a
}
let result_1 = fn(10) // 不指定泛型，TS自动检测，此时为number
let result_2 = fn<string>('hello') // 指定类型，string
```

可以有多个泛型

```tsx
function fn<T, U>(a: T, b: U): T {
  return a
}
```

泛型还可以继承于接口

```tsx
interface Inter {
  length: number
}
function fn<T extends Inter>(a: T): number {
  return a.length
}
```

```tsx
class MyClass<T> {
  name: T
  constructor(name: T) {
    this.name = name
  }
}
const myclass = new MyClass<string>('xm')
```
