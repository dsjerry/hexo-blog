---
title: 异步JavaScript
toc: true
date: 2021-05-17 16:19:36
tags: js
categories: 前端学习
thumbnail: https://z3.ax1x.com/2021/05/23/gX8pOs.png
cover: https://z3.ax1x.com/2021/05/23/gX8pOs.png
---

🌎 参考：<a href="https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous">MDN-异步 JavaScript</a>，《深入理解 ES6》- NICHOLAS C.ZAKAS，黑马程序员，王磊同学<!-- more -->

# 同步和异步

## 同步

按顺序**等待执行**，代码传入调用栈，执行完毕后再从调用栈中移除。

```js
console.log("global begin");

function bar() {
  console.log("bar task");
}

function foo() {
  console.log("foo task");
  bar();
}

foo();

console.log("global end");
```

输出结果为：

```
global begin
foo task
bar task
global end
```

## 异步

📖 JavaScript 引擎是基于单线程事件循环的概念构建的，在同一时间只允许一个代码块在运行。如果一个函数依赖于另一个函数的结果，它只能等待那个函数结束才能继续执行，这样就容易造成代码阻塞。异步就是用于解决这些问题。

> 当前任务为异步的话，不会等待当前任务执行结束，而是立即执行下一个任务

在 JavaScript 中实现异步的方法有许多种：`setTimeout()`和`setInterval()`、事件模型、回调函数、Promise、Fetch、async/await、axios

## setTimeout 作为例子

```js
console.log("global begin");

setTimeout(function timer1() {
  console.log("timer1 invoke");
}, 1800);

setTimeout(function timer2() {
  console.log("timer2 invoke");
  setTimeout(function inner() {
    console.log("inner invoke");
  }, 1000);
}, 1000);

console.log("global end");
```

分为四个部分：调用栈，任务队列，事件循环，api 环境（浏览器、nodejs）

1. 同步代码放进`调用栈`
2. 遇到计时器，将计时器放进`api环境`，计时器开始计时（异步操作，不影响第`3`的进行）
3. 遇到同步代码，继续放进`调用栈`
4. 计时器时间到，将计时器（计时器中的回调函数）放进`任务队列`
5. `事件循环`检查`调用栈`是否为空，不为空则继续执行`调用栈`中的代码
6. `调用栈`为空，`事件循环`检查`任务队列`是否为空，不为空则将`任务队列`中的代码放进`调用栈`

> JavaScript 是单线程，但是运行它的环境并不是单线程。具体要看运行环境提供的 API 到底是同步还是异步

# 回调函数

> 被作为实参传入另一函数，并在该外部函数内被调用，用以来完成某些任务的函数，称为回调函数。

提一下**事件模型**（onclick、onmouseover 等等），其实和回调函数是类似的，比如按钮点击，代码都是在按钮点击的时候执行。不同的是回调函数中被执行的函数不是一赋值的形式传递（=），而是作为参数传入。

回调函数在`Node.js`中广泛应用，所以下面的例子中

```js
readFile("example.txt", (err, contents) => {
  if (err) {
    throw err;
  }
  console.log(contents);
});
console.log("Hi!");
```

`readFile()`会开始执行，读取参数 1 中指定的文件，读取结束后执行参数 2(回调函数)，但是读取文件可以说是一个阻塞的过程，所以浏览器是先输出**Hi!**，然后当`readFile()`执行结束的时候，会在任务队列末尾添加一个新任务，用于处理回调函数里面的内容。

回调函数是一个很好的异步操作，回调函数嵌套多的时候，就会造成*回调地狱*。如果想实现复杂的功能，这样的代码很难理解其意思。

```js
readFile("example.txt", (err, contents) => {
  if (err) {
    throw err;
  }
  writeFile("example.txt", (err, contents) => {
    if (err) {
      throw err;
    }
    otherFunc1((err, contents) => {
      if (err) {
        throw err;
      }
      otherFunc2((err, contents) => {
        if (err) {
          throw err;
        }
        // .....
      });
    });
  });
});
```

# Promise

- CommonJS 率先提出，ES6 标准化

更多：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise">全局对象--Promise</a>，<a href="https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous/Promises">异步 JavaScript</a>

promise 承诺在未来的某一时间会返回执行的内容，但是不确定在什么时候，不论执行的结果是对的还是错的都会有返回。

## 实例

使用 Promise()构造函数创建自己的 promise，这个构造函数接收一个参数，这个参数是一个*执行器函数*，这个执行器函数有两个参数，这两个参数是两个函数，这两个函数分别是：处理成功执行的 resolve()、处理失败执行的 reject()

```js
let promise = new Promise(function (resolve, reject) {
  console.log("你好");
  resolve("世界！");
  // 或者失败
  // reject('失败！')
});

promise.then((val) => {
  console.log(val);
});

console.log("，");
```

上面代码依次输出`你好，世界！`；

- 首先因为*Promise 的执行器*中的代码会**立刻执行**，然后再执行其他的代码
- 调用`resolve()`后触发一个**异步操作**，传入`then`和`catch()`的函数会被添加到任务队列中并*异步执行*
- 虽然上面的`then()`在`console.log('，')`之前，但是与执行器不同的是并没有立即执行，这是因为完成处理程序和拒绝处理程序总是在执行器完成后被添加到*任务队列的末尾*。

promise 中可以使用 promise 的 _原形_ 和 _静态方法_。👉 <a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise">更多</a>

`promise.prototype.then()`：处理成功执行

`promise.prototype.catch()`：处理失败执行

`promise.prototype.finally()`：不管是成功还是失败都执行

promise.then()，接收两个参数（可选），这两个参数是处理函数，参数 1 是成功执行的处理函数，参数 2 是失败的处理函数，所以`then(null, func) 和 catch()`的作用是一样的

```js
promise.then((value) => {
  // value是上面成功执行时resolve()传过来的参数
  console.log(value); // 成功！
});

promise.then(null, (value) => {
  console.log(value); // 失败！
});
promise.catch((value) => {
  console.log(value); // 失败
});

promise.finally(() => {
  console.log("finish.");
});
```

- Promise 对象的`then()`方法返回一个新的 Promise 实例，所以可以链式调用
- 后面的`then()`就是为上一个`then()`的返回的 Promise 添加处理函数
- 前面的`then()`返回的值会作为后面`then()`的参数
  - 如果这个值是一个 Promise 实例，那么后面的`then()`会等待这个 Promise 实例执行完毕

▶️ 使用第二个参数作为`catch()`而不是使用链式`catch()`，区别是第二个参数只捕获当前的错误，而不是整条链的。有利有弊：

- 优点：可以在链中的任何位置处理错误，或者说并且容易找出错误发生的位置
- 缺点：如果在链中的多个位置都需要处理错误，就需要多次调用`catch()`，这样就会造成代码冗余

除了使用链条最后的`catch()`捕获整条链的错误，还可以使用`unhandledrejection`事件捕获整条链的错误

**web**环境中（全小写命名）

```js
window.addEventListener("unhandledrejection", (event) => {
  console.log(event.reason); // 失败原因，一般是一个错误对象
  console.log(event.promise); // 失败的 Promise 实例
});
```

**node**环境中（驼峰式命名）

```js
process.on("unhandledRejection", (reason, promise) => {
  console.log(reason); // 失败原因，一般是一个错误对象
  console.log(promise); // 失败的 Promise 实例
});
```

### 例子

加载图片

```js
function loadImage(url) {
  let promise = new Promise((resolve, reject) => {
    var image = new Image();
    image.src = url;
    image.onload = () => {
      resolve(image);
    };
    image.onerror = () => {
      reject("加载图片出错！");
    };
  });
  return promise;
}
loadImage("coffee.jpg").then(
  (result) => {
    document.body.appendChild(result);
  },
  (error) => {
    console.log(error);
  }
);
```

## 静态方法

`Promise.resolve()`：返回一个**通过**的 promise

😅 如果里面传入了一个对象，里面又刚好有`then()`方法，那么这个对象就会被当作一个 promise 实例，然后返回这个对象（thenable）

- Promise 为普及的时候，其他库可能会有自己的 promise 实现，这些实现可能不会遵循 Promise/A+ 规范，但是会有`then()`方法，这样就可以使用`Promise.resolve()`将其转换为 Promise 实例

`Promise.reject()`：返回一个**拒绝**的 promise

🎇 以下静态方法参数接收一个 _数组_ 作为参数，数组里面的是 promise 实例（`Array<Promise>`）

- 返回值是一个 promise 实例

`Promise.all()`：只要有一个**拒绝**，就是拒绝。

`Promise.race()`：只要有一个**通过**，就是通过

`Promise.allSettled()`：不管是**拒绝**还是**通过**，都会执行

### 例子

ajax 请求超时

```js
const request = ajax("api/xxx");
const timeout = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error("请求超时")), 5000);
});

Promise.race([request, timeout])
  .then((result) => {
    console.log(result);
  })
  .catch((error) => {
    console.log(error);
  });
```

node.js 中的读取文件

```js
let fs = require("fs");

function readFile(filename) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, { encoding: "utf-8" }, (err, contents) => {
      if (err) {
        reject(err);
        return;
      }
      resolve(contents);
    });
  });
}

let promise = readFile("example.txt");
promise.then((contents) => {
  console.log(contents);
});
promise.catch((error) => {
  console.log(error);
});

// 或者
promise.then(
  (contents) => {
    console.log(contents);
  },
  (error) => {
    console.log(error);
  }
);

let p1 = readFile("happy.txt");
let p2 = readFile("happyok.txt");
let p3 = readFile("happyno.txt");

Promise.all([p1, p2, p3]).then((result) => {
  console.log(result);
});
Promise.race([p1, p2, p3]).then((result) => {
  // 由于p1执行较快，Promise的then()将获得结果'P1'。p2,p3仍在继续执行，但执行结果将被丢弃。
  console.log(result);
});
```

# async 和 await

> 对比生成器，`*`去掉，换成`async`，`yield`换成`await`，不需要执行器函数，返回的是一个 promise 实例

> `async`和`await`关键字让我们可以用一种更简洁的方式写出基于[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)的异步行为，而无需刻意地链式调用`promise`。

## async

使用 `async` 关键字，把它放在函数声明之前，使其成为 [async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)

- async 函数是使用`async`关键字声明的函数，其中允许使用`await`关键字。

将 `async` 关键字加到函数申明中，可以告诉它们返回的是 promise，而不是直接返回值。

## await

- 原生 Promise，等待
- thenable 对象，构造成新的 Promise
- 不是 thenable 对象，包装成 `Promise.resolve()`

~~await 只在异步函数里面才起作用~~。它可以放在任何异步的，基于 promise 的函数之前。

> `nodejs 14.8.0` es 模块支持顶级`await`; 浏览器环境，也是在模块顶级可用

它会暂停代码在该行上，直到 promise 完成，然后返回结果值。在暂停的同时，其他正在等待执行的代码就有机会执行了。(这样的异步代码看起来像同步代码)

## 例子

将上面的 promise 改为 async 模式

```js
async function load(url) {
  var image = new Image();
  image.src = url;
  return image;
}
async function append(image) {
  document.body.appendChild(image);
}
async function loadImage(url) {
  let image = await load(url);
  await append(image);
}
loadImage("coffee.jpg");
```

结合 axios 使用

```js
// 1.  async 基础用法
// 1.1 async作为一个关键字放到函数前面
async function queryData() {
  // 1.2 await关键字只能在使用async定义的函数中使用      await后面可以直接跟一个 Promise实例对象
  var ret = await new Promise(function (resolve, reject) {
    setTimeout(function () {
      resolve("nihao");
    }, 1000);
  });
  // console.log(ret.data)
  return ret;
}
// 1.3 任何一个async函数都会隐式返回一个promise   我们可以使用then 进行链式编程
queryData().then(function (data) {
  console.log(data);
});

//2.  async    函数处理多个异步函数
axios.defaults.baseURL = "http://localhost:3000";

async function queryData() {
  // 2.1  添加await之后 当前的await 返回结果之后才会执行后面的代码

  var info = await axios.get("async1");
  //2.2  让异步代码看起来、表现起来更像同步代码
  var ret = await axios.get("async2?info=" + info.data);
  return ret.data;
}

queryData().then(function (data) {
  console.log(data);
});
```

# 宏任务和微任务

宏任务，重新回到任务队列的末尾，等待下一次事件循环；而微任务，会在当前任务执行结束后立即执行。

# 生成器

## 使用

知识点：`yeild`关键字，`generator.next()`，`gernerator.throw()`，`generator.return()`

```js
function* foo() {
  console.log("start");
}

// 初次调用，里面内容尚未执行
const generator = foo();
// 只有调用next()才会执行
generator.next();
```

在里面添加`yield`关键字，可以让函数暂停执行，然后返回一个值

- 这个值是一个对象，包含两个属性：`value`和`done`，`value`是`yield`后面的值，`done`是一个布尔值，表示是否执行完毕。
- 和`return`不同，`yield`不会立即结束函数的执行，而是返回一个值，然后暂停执行，等到下一次调用`next()`的时候，再继续执行。

```js
function* foo() {
  console.log("start");
  yield "hello";
  console.log("end");
}

const generator = foo();
generator.next(); // start
generator.next(); // end
```

如果调用`next()`的时候传入了参数，那么这个参数会作为上一次`yield`的返回值

```js
function* foo() {
  console.log("start");
  const val = yield "hello";
  console.log(val);
  console.log("end");
}

const generator = foo();
generator.next(); // start
generator.next("world"); // world end
```

如果调用`gernerator.throw()`，会抛出一个错误，这个错误在*生成器函数*内部抛出，可使用`try...catch`捕获

## 管理异步

```js
function* main() {
  const users = yield ajax("api/users.json");
  console.log(users);

  const posts = yield ajax("api/posts.json");
  console.log(posts);
}

const generator = main();
const result1 = generator.next();
result1.value.then((data) => {
  const result2 = generator.next(data);
  if (result2.done) return;

  result2.value.then((data) => {
    generator.next(data);
  });
});
```

改为递归的方式

```js
function* main() {
  try {
    const users = yield ajax("api/users.json");
    console.log(users);

    const posts = yield ajax("api/posts.json");
    console.log(posts);
  } catch (err) {
    console.log(err);
  }
}

function handleResult(result) {
  if (result.done) return;
  result.value
    .then((data) => {
      handleResult(generator.next(data));
    })
    .catch((err) => {
      // 抛出异常，在生成器函数main()中使用try...catch捕获
      generator.throw(err);
    });
}

const generator = main();

handleResult(generator.next());
```

封装成执行器

```js
function run(generator) {
  const iterator = generator();

  function handleResult(result) {
    if (result.done) return;
    result.value
      .then((data) => {
        handleResult(iterator.next(data));
      })
      .catch((err) => {
        iterator.throw(err);
      });
  }

  handleResult(iterator.next());
}

run(main);
```

有一个专门的库可以实现这个功能：<a href="https://github.com/tj/co" target="_blank">co</a>

不过后来出现了`async`和`await`，所以这个库用得就少了。

<!-- <span style="font-size: 70px; color: black; background-color: yellow;padding: 20px">异步JavaScript</span> -->

```

```
