---
title: 网络请求
toc: true
date: 2021-05-08 17:01:24
tags: [js]
categories: 前端学习
thumbnail: https://z1.ax1x.com/2023/09/17/pPhuHTx.jpg
cover: https://z1.ax1x.com/2023/09/17/pPhuHTx.jpg
---

🌎 列举 JavaScript 中网络请求的方式

📦 ajax, jqeury.ajax, fetch, axios, got

<!-- more -->

# AJAX

> 环境: 浏览器

全称：异步 JavaScript 和 XML（Asynchronous JavaScript and XML）

- 不是一种技术，而是一种将现有技术结合起来的使用方法。🌎 [mdn ajax 文档](https://developer.mozilla.org/zh-CN/docs/Web/Guide/AJAX)
- 虽然其中 X 代表 _XML_，但是 _JSON_ 是首选类型

使用 AJAX 发送的请求和传统的表单 (`<form>`标签) 的请求方式不一样，所以如果使用按钮，其属性类型不用设置为`submit`，`name`属性也不用写出来

## 基本使用

```js
// 1. 创建ajax对象
const xhr = new XMLHttpRequest();
// 2. 请求的方式和请求地址。第三个参数 boolean 指定是否为异步
xhr.open("get", "http:localhost/test");
// 3. 发送请求。参数可传递任何想传到服务器的内容
xhr.send();
// 4. 获取服务器端响应的内容
xhr.onload = function () {
  console.log(xhr.responseText);
};
```

在上面第 4 点中，也可以使用`xhr.onreadystatechange`事件，不同的是使用这个方法需要对响应状态进行判断

```js
xhr.onreadystatechange() = function {
    if (xhr.readyState == 4) {
        console.log(xhr.responseText);
    }
}
```

- 这个方法兼容低版本 IE 浏览器，`xhr.onload`不兼容低版本 IE 浏览器
- 这个方法会被调用多次，`xhr.onload`只调用一次
- 这个方法需要对响应状态码判断，`xhr.onload`不用

> 在上面中得到的响应数据是`String`类型，要将其转换可以使用`JSON.parse(xhr.responseText)`

## 请求参数的传递

例如有这样的一个页面

```html
<input type="text" id="username" /> <input type="text" id="age" />
```

### GET 请求

```js
const xhr = new XMLHttpRequest();
// 配置请求参数
let userName = username.value;
let userAge = age.value;
let params = "username=" + userName + "&age=" + userAge;
// 参数在请求的url中
xhr.open("get", "http:localhost/get?" + params);
xhr.send();
xhr.onload = function () {
  console.log(xhr.responseText);
};
```

- 请求地址与参数之间使用 `?` 隔开，参数与参数之间使用 `&` 隔开

### POST 请求

```js
const xhr = new XMLHttpRequest();

let userName = username.value;
let userAge = age.value;
let params = "username=" + userName + "&age=" + userAge;
xhr.open("get", "http:localhost/post");
// post请求需要设置请求参数的类型
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
// 发送请求时需要传递参数
xhr.send(params);
xhr.onload = function () {
  console.log(xhr.responseText);
};
```

求参数的类型有：

- `application/x-www-form-urlencoded`的形式：name=zs&age=20
- `application/json`的形式：{ name: 'zs', age: 20 }

在请求头中指定 _Content-Type_ 属性的值是 _application/json_ 的时候就是告诉服务器当前请求的格式是 json

在上面的代码中，是用 _json_ 格式发送请求

```js
xhr.setRequestHeader("Content-Type", "application/json");
// 然后需要将json对象转换成json字符串
xhr.send(JSON.stringify({ name: "zs", age: 20 }));
```

如果使用的是 Node.js，就在入口文件中配置，来决定请求的参数类型

```js
app.use(bodyParser.urlencoded());
// 或者
app.use(bodyParser.json());
```

> 传统的表单提交和 GET 请求都**不支持**提交 json 类型的数据

## 响应状态

### ajax 状态码

在创建 ajax 对象、配置、发送请求、接收服务端返回的内容的过程中，会返回一个数值，这就是 ajax 状态码，使用`xhr.readyState`获得。

- 0：请求没有初始化
- 1：服务连接已经建立，但还没调用 send()
- 2：已经加载完成
- 3：正在处理请求（这时候有一部分的响应数据是可以使用的了）
- 4：响应完成

数值可通过`XMLHttpRequest.名称获得`：有`DONE`,`OPENED`,`HEADERS_RECEIVED`,`LOADING`,`DONE`

### http 状态码

1. 网络正常，服务器接收请求之后返回的数据不是预期的内容，可以使用`xhr.status`获取服务器返回的 http 状态码来判断（_服务器状态码和 ajax 状态码不一样_）
2. 网络正常，服务端返回了 _404_ 状态码，可能是请求的地址有错
3. 网络正常，服务器返回了 _505_ 状态码，服务器出了问题
4. 网络断了，使用 `xhr.onerror()`来处理错误

> 区分 ajax 状态码和 http 状态码
>
> - ajax 状态码：表示 ajax 的请求过程状态，是 ajax 对象返回的
>
> - http 状态码：表示请求的结果，是服务器返回的

### 其他

#### 低版本 IE 浏览器缓存

在低版本的 IE 浏览器中，请求的内容只基于第一次请求，即是请求的文件里面的内容发生了变化，客户端依然使用旧的数据。

解决方法，使得每次请求的参数不一样（但是不能和现有的参数相同）

```js
xhr.open("get", "http://test.com?page=" + Math.random());
```

#### ajax 异步

```js
xhr.onload = () => {
  console.log("2");
};
console.log("1");
```

- 先输出 1，再输出 2

# jQuery.ajax

使用大致分为两种：

- 浏览器环境`<script>`标签引入，引入从官网下载的[jqeury](https://jquery.com/download/)，或者通过 CDN
- node 环境 npm 包

**浏览器环境**

```html
<head>
  <script src="jquery-3.7.1.min.js"></script>
</head>
<body>
  <button id="btn">BTN</button>
</body>
<script>
  $("#btn").on("click", () => {
    $.ajax({
      url: "xxx",
      succuess: () => {},
      error: () => {},
    });
  });
</script>
```

**node 环境使用**

```shell
npm install jquery --save
```

```js
const { ajax } = require("jqeury");

ajax({
  url: "",
  success: () => {},
  error: () => {},
});
```

# fetch

> 环境: 浏览器，nodejs v17.5.0, v16.15.0 开始支持 fetch

和 _jquery.ajax()_ 不同有：

- 返回的`Promise`不会因为 HTTP 错误状态而被拒绝（404、500），正常 resolve，只不过 _ok_ 状态设置成了 _false_。网络故障或者情趣阻止时才会 reject
- 默认不发送跨源 cookie

## 使用

```js
fetch("http://example.com/movies.json")
  .then((response) => response.json())
  .then((data) => console.log(data));
```

- 通过`response.ok`来判断请求是否成功（值是 boolean 类型）。`true`的时候，状态码范围是 `200-299`
- `response.status`，状态码，默认值是 `200`
- `response.statusText`：默认值是 `""`。
  - OK -> 200
  - Continue -> 100
  - Not Found -> 404

第二个参数设置配置对象(init 对象)，常用的有

- method：请求方法
- mode：no-cors, \*cors, same-origin
- headers: { "Content-Type": "application/json" },
- body: JSON.stringify(data)

请求体可以配合 `FormData`使用

```js
const formData = new FormData();
const fileField = document.querySelector('input[type="file"]');

formData.append("username", "abc123");
formData.append("avatar", fileField.files[0]);

fetch("https://example.com/profile/avatar", {
  method: "PUT",
  body: formData,
})
  .then((response) => response.json())
  .then((result) => {
    console.log("Success:", result);
  })
  .catch((error) => {
    console.error("Error:", error);
  });
```

## 自定义请求对象

通过`new Request`创建一个请求对象（和`fetch`接收的参数相同）

```js
const myHeaders = new Headers();

const myRequest = new Request("flowers.jpg", {
  method: "GET",
  headers: myHeaders,
  mode: "cors",
  cache: "default",
});

fetch(myRequest)
  .then((response) => response.blob())
  .then((myBlob) => {
    myImage.src = URL.createObjectURL(myBlob);
  });
```

或许在要复用某些代码的时候，这会很好用

# axios

> 环境: 浏览器，nodejs

```shell
npm install axios
```

CDN:

```js
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

## 请求

普通请求

```js
import axios from "axios";

// get请求（默认方式）
axios("/user/12345");

// post请求
axios({
  method: "post",
  url: "/user/12345",
  data: {
    firstName: "Fred",
    lastName: "Flintstone",
  },
});
```

请求 stream 类型数据

```js
// 在 node.js 用GET请求获取远程图片
axios({
  method: "get",
  url: "http://bit.ly/2mTM3nY",
  responseType: "stream",
}).then(function (response) {
  response.data.pipe(fs.createWriteStream("ada_lovelace.jpg"));
});
```

## 响应结构

```js
{
  // `data` 由服务器提供的响应
  data: {},

  // `status` 来自服务器响应的 HTTP 状态码
  status: 200,

  // `statusText` 来自服务器响应的 HTTP 状态信息
  statusText: 'OK',

  // `headers` 是服务器响应头
  // 所有的 header 名称都是小写，而且可以使用方括号语法访问
  // 例如: `response.headers['content-type']`
  headers: {},

  // `config` 是 `axios` 请求的配置信息
  config: {},

  // `request` 是生成此响应的请求
  // 在node.js中它是最后一个ClientRequest实例 (in redirects)，
  // 在浏览器中则是 XMLHttpRequest 实例
  request: {}
}
```

# got

> 环境: nodejs

```shell
npm install got
```

参考网站：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript">MDN</a>

<!-- <br/>
<br/>
<br/>
<div style="text-align: center"><span style="font-size: 80px">发送<span style="color: red;">HTTP</span>请求</span>
<br />
<span style="font-size: 50px">💻🔁🌏</span>
</div>
<br/>
<br/>
<br/>
<br/> -->
