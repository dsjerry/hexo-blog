---
title: ES6解构
toc: true
date: 2021-02-14 14:25:42
tags: js
categories: 前端学习
thumbnail: https://s3.ax1x.com/2021/02/14/yyIoWQ.png
cover: https://s3.ax1x.com/2021/02/14/yyIoWQ.png
---

主要内容：对象解构、数组解构、混合解构

<!-- more -->

> 注意一点：`解构` 和 `解构赋值` 不是同一个概念，解构赋值是解构之后，赋予一个新值

如果不使用解构：

```javascript
let options = {
    repeat: true,
    save: false
};
let repeat = options.repeat,
    save = options.save;
```

# 对象解构

对象解构的语法形式是在一个赋值操作符（=）左边放置一个`对象字面量`

> 对象字面量：和 JSON 看起来差不多，JSON 是基于对象字面量的语法构建的
>
> > 对象字面量：`name: jerry`
> >
> > JSON：`"name": "jerry"`

## 基本形式

```js
let node = {
    type: "festival",
    name: "Spring Festival"
};
let { type, name } = node;

console.log(type);		// "festival"
console.log(name);		// "Spring Festival"
```

使用解构的时候需要`初始化`（赋值），不然会报错。

```js
❌ var { type,name };
```

## 解构赋值

解构也可以重新给变量赋值

```javascript
let node = {
    type: "festival",
    name: "Spring Festival"
},
	type = "jieri",
	name = "chunjie";

({ type, name } = node);

console.log(type);		// festival
console.log(name);		// Spring Festival
```

- 在JavaScript语法规则中，`=` 左边不能出现花括号，所以要使用 () 包起来

在传参的时候使用解构赋值

```javascript
let node = {
    type: "festival",
    name: "Spring Festival"
},
	type = "jieri",
	name = "chunjie";

function outPutInfo(value) {
    console.log(value == node);		// true
}

outPutInfo( { type, name } = node );

console.log(type);		// festival
console.log(name);		// Spring Festival
```

- 解构赋值的时候 `=` 右边的值如果是 `null` 或 `undefined`，会报错。

## 默认值

如果表达值指定的局部变量的名称（= 左边）在对象中（=右边）不存在，则这个局部变量会被赋值为 `undefined`

```javascript
let node = {
    type: "festival",
    name: "Spring Festival"
};
let { type, name, value } = node;

console.log(type);		// "festival"
console.log(name);		// "Spring Festival"
console.log(value);		// undefined
```

同时可以给这个 `value` 随意定义一个默认值，在 value 后面使用 `=`

```javascript
let { type, name, value = "lunar" } = node;
console.log(value);		// lunar
```

## 非同名局部变量赋值

```javascript
let node = {
    type: "festival",
    name: "Spring Festival"
};

let { type: localType, name: localName } = node;

console.log(localType);		// "festival"
console.log(localName);		// "Spring Festival"
```

- type: localType 的意思是读取名为 type 的属性的值然后存到 localType 中。可以想象为：使用 localType 来表示 type
- 这种语法和传统的是相悖的，<u style="color: red;">原本应该是名称在冒号左边，值在右边；这个变成了名称在冒号右边，需要读取的值在冒号左边</u>

也是可以使用 `=` 设置默认值的

```javascript
let { type: localType, name: localName = "chunjie" } = node;
```

## 嵌套对象结构

```javascript
let node = {
    type: "festival",
    name: "Spring Festival",
    loc: {
        start: {
            line: 1,
            column: 1
        },
        end: {
            line: 1,
            column: 4
        }
    }
};

let { loc: { start } } = node;

console.log(start.line);		// 1
console.log(start.column);		// 1
```

- 这里是指在找到 node 对象中的 loc 属性后，再深入一层找 start 属性
- 冒号左侧的代表在对象中检索的位置，右侧代表<span style="color: orange">被赋值的变量名</span>

同时可以使用在 非同名的局部变量

```javascript
let { loc: { start: localStart } } = node;

console.log(localStart.line);		// 1
console.log(localStart.column);		// 1
```

> `let { loc: {} } = node;`，是合法的✔，但是不推荐使用❌

# 数组解构

相对对象结构，较为简单

## 基本形式

```javascript
let colors = [ "red", "green", "blue" ];
let [ firstColor, secondColor ] = colors;

console.log(firstColor);		// "red"
console.log(secondColor);		// "green"
```

- 未显式声明的元素直接忽略
- 数组本身不会变化
- 和对象解构一样，需要初始化

同时也可以省略前面的声明

```javascript
let colors = [ "red", "green", "blue" ];
let [ , , thirdColor ] = colors;
console.log(thirdColor);		// "blue"
```

## 解构赋值

在数组结构中，在重新赋值的时候，不同于对象解构，不需要使用 () 包起来

```javascript
let colors = [ "red", "green", "blue" ],
    firstColor = "black",
    secondColor = "purpel";

[ firstColor, secondColor ] = colors;

console.log(firstColor);		// "red"
```

- 这里的结构赋值和上一个**基本形式**中的其实是差不多的，唯一区别就是`firstColor` 和 `secondColor` 已经定义了。

### 用于变量值交换

🔻 非解构中交换变量

```javascript
let a = 1, b = 2, tmp;

tmp = a;
a = b;
b = tmp;

console.log(a);		// 2
console.log(b);		// 1
```

🔻 解构中交换变量

```javascript
let a = 1, b = 2;

[ a, b ] = [ b, a ];

console.log(a);		// 2
console.log(b);		// 1
```

- 右侧的值为 `null` 或 `undefined` 会报错。

## 默认值

指定位置的属性不存在或者其值为 undefined 的时候使用默认值

```javascript
let colors = [ "red" ];

let [ firstColor, secondColor = "green" ] = colors;

console.log(firstColor);		// "red"
console.log(secondColor);		// "green"
```

## 嵌套数组结构

和嵌套对象结构类似

```javascript
let colors = [ "red", [ "green", "lightgreen" ], "blue" ];

let [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);		// "red"
console.log(secondColor);		// "green"
```

如果

```javascript
let [ firstColor, [ secondColor, thirdColor ] ] = colors;
console.log(thirdColor);		// "lightgreen"
```

如果

```javascript
let [ firstColor, [ secondColor ], thirdColor ] = colors;
console.log(thirdColor);		// "blue"
```

## 不定元素

```javascript
let colors = [ "red", "green", "blue" ];

let [ firstColor, ...resColors ] = colors;

console.log(firstColor);	// "red"
console.log(resColors.length);	// 2
console.log(resColors[0]);	// "green"
console.log(resColors[1]);	// "blue"
```

### 用于数组复制

在 `ES5` 中，使用 `concat()` 方法来克隆数组

```javascript
var colors = [ "red", "green", "blue" ];
var cloneColor = colors.concat();

console.log(cloneColors);		// "[ red, green, blue ]"
```

- 其实 concat 函数起初是用来拼接两个数组的，如果调用的时候不传递参数，就返回当前数组的副本

在 `ES6` 中

```javascript
let colors = [ "red", "green", "blue" ];
let [ ...cloneColors ] = colors;

console.log(cloneColors);		// "[ red, green, blue ]"
```

> 不定元素必须是最后一位，不然会报错

# 混合解构

```javascript
let node = {
    type: "festival",
    name: "Spring Festival",
    loc: {
        start: {
            line: 1,
            column: 1
        },
        end: {
            line: 1,
            column: 4
        }
    },
    range: [0, 3]
};

let {
    loc: { start },
    range: [ startIndex ]
} = node;

console.log(start.line);		// 1
console.log(start.colume);		// 1
console.log(startIndex);		// 0
```

- 解构模式中的 `loc:` 和 `range:` 仅代表它们在 node 对象中所处的位置（也就是改对象的属性）
- 可用于提取 JSON 数据，这样不用遍历整个对象就可以获得想要的内容

# 解构参数

就是说将解构使用在参数的传递过程中

## 基本形式

在定义一个有大量可选参数的JavaScript函数的时候

🔻 不使用解构

```javascript
// options 表示其他参数
function setCookie(name, value, options) {
    options = options || {};
    
    let secure = options.secure,
        path = options.path,
        domain = options.domain,
        expires = options.expires;
    
    // ...其他代码
}

// 第三个参数映射戴 options 中
setCookies( "type", "js", {
    secure: true,
    expires: 60000
} );
```

- 方便：name 和 value 是必须参数，而其他参数可选并且顺序不限，使用这种方法方便
- 不便：如果这个可选的参数实在有太多可选，要想知道这些参数，需要进入函数体查看，不太方便

🔻 使用解构

```javascript
function setCookies(name, value, { secure, path, domain, expires }) {
    // ...其他代码
}
setCookies( "type", "js", {
    secure: true,
    expires: 60000
} );
```

- 这样就一眼能看出到底有什么参数

## 必须传值的解构参数

在<span style="color: orange">定义</span>函数的时候，形参使用解构参数，在<span style="color: darkcyan">调用</span>的时候如果不提供被解构的参数，会报错

接上面 `setCookies()`

```javascript
// 报错
setCookies( "type", "js");
```

因为实际上JavaScript引擎会将代码解析为

```javascript
function setCookies(name, value, options) {
    let { secure, path, domain, expires } = options;
    // ...其他代码
}
```

> 虽然解构参数是<span style="border: 1px solid red; padding: 1px">可选</span>的，但是必须传递第三个参数进去，但解构参数是必须时，就不会有这样的问题

解决方法

给<span style="border: 1px solid red; padding: 1px">可选</span>的结构参数赋予默认值

```javascript
function setCookies(name, value, { secure, path, domain, expires } = {} ) {
    // ...其他代码
}
```

## 解构参数的默认值

1️⃣ 为防止不传递第三个参数导致代码报错，要使用上面提到的 `{} = {}` 形式设置默认值

2️⃣ 将 `{} = {}` 中左边的 `{}` 独立抽出以简化代码

```javascript
const setCookieDefaults = {
    secure: false,
    path: "/",
    domain: "happy.com",
    expires: new Date(Date.now() + 360000000)
};

function setCookie(name, value,
                   	 {
    secure = setCookieDefault.secure,
    path = setCookieDefault.path,
    domain = setCookieDefault.domain,
    expires = setCookieDefault.expires
					} = setCookieDefaults
    ) {
    // 函数内容
}
```



# 小结

1️⃣ 在以上几种解构中，`=` 右边的值为 `null` 或 `undefined`，程序会报错。



参考引用📖：`深入理解ES6` / `Understanding ECMAScript 6` - **Nicholas C. Zakas** | 第五章 - 解构：使数据访问更便捷





<!-- <h1 style="font-size: 100px"><span style="color: gold; border: 5px dotted cyan; padding: 2px">角刀牛</span><span style="color: darkcyan;border: 5px dotted pink;padding: 2px">木勾</span></h1> -->





