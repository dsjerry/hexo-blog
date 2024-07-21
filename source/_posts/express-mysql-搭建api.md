---
title: express基本使用
date: 2020-05-29 13:02:07
tags: [node]
categories: 部署运维
thumbnail:
cover:
toc: true
---

🌎🌎

<!-- more -->

# 安装

🌎 通过 npm 进行常规安装，仅下载 express 这个包，其他内容根据自己需求添加

🚀 通过生成器安装的是一个具有完整内容（js 文件、模板页面文件...）的脚手架，安装之后就可以直接运行

## 常规安装

```shell
npm init
```

输入命令后会有项目的基本信息要填，这些信息可以直接回车跳过，下面这个除外

```shell
entry point: (index.js)
```

- 这个是修改入口文件的，可以修改其默认值，修改入口文件为`app.js`。当然也可以以后手动修改

安装 Express 并将其保存到依赖列表中：

```shell
npm install express
```

### 运行

```js
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Hello World!");
});

app.listen(3000, () => {
  console.log("http://localhost:3000");
});
```

## express 应用程序生成器

通过 _express-generator_ 快速创建一个应用骨架

```shell
npx express-generator
```

```shell
npm install
```

- 或者是旧版本的 node，使用：

  ```shell
  npm install -g express-generator
  express
  ```

启动服务

```shell
npm start
```

## 基本信息

- bin：原始的入口文件目录，后面会改掉
- public：放一些静态资源的地方，图片、介绍、、css 之类的
- routes：路由文件位置
- views：静态页面位置
- app.js：在根目录下作为入口文件

### 改写入口文件

- 起初，入口文件是在 _bin_ 目录下的 _www.js_
- 将 _www.js_ 里面的 _http_、_server_ 模块转移到 _app.js_ ，_app.js_ 里面的暴露去掉，添加一个监听端口

# 使用 express 路由

作为中间件，接收和处理来自客户端的请求。路由都放在 _routes_ 目录下。

```javascript
var express = require("express");
var router = express.Router();
router.get("/", (req, res, next) => {
  // 返回内容给客户端
  res.send();
});
router.post("/login", (req, res, next) => {
  // post 请求的内容在 res.body 对象中
  var data = req.body;
  console.log(data);
});
```

## 安装使用 mysql 模块

```shell
npm install mysql --save
```

### 数据库配置

可以在根目录建一个文件夹，名字随便但最好是顾名思义。例如 _dbConfig_

创建文件 _config.js_ 用来保存连接数据库时所要用到的信息

```javascript
const DB = {
  host: "主机名",
  port: "端口",
  user: "用户名",
  password: "密码",
  database: "数据库",
};
module.exports = DB;
```

同目录下创建 _database.js_ 连接数据库

```javascript
const mysql = require("mysql");
const config = require("./config");

// 创建连接
const database = mysql.createConnection({
  host: config.host,
  port: config.port,
  user: config.user,
  password: config.password,
  database: config.database,
});
// 连接mysql
database.connect();
module.exports = database;
```

连接的同时可以做别的事情

```javascript
database.connect((err) => {
  if (err) console.log(err);
  else console.log("sucessful");
});
```

### 数据库使用

```javascript
database.query("select * from xxx", (err, rows, fields) => {
  if (err) throw err;
  console.log("查询结果：" + rows);
});
```

_err_ ：错误信息、_rows_ ：查询结果、_fields_ ：当前查询中涉及到的字段信息，一般很少用到

### router 搭配 mysql 处理客户端的请求

- 在 _routes_ 文件夹中的路由文件中，例如在 index.js 中

```javascript
var express = require("express");
var router = express.Router();
var db = require("../dbConfig/database");

// 此处使用 / 来表示当前路由文件，因为在 app.js 中还会配置
router.get("/", (req, res, next) => {
  var query = "select * from xxx";
  db.query(query, (err, rows, fileds) => {
    if (err) throw err;
    res.send(rows);
  });
});
```

- 在根目录的 _app.js_ 中

```javascript
// 引入路由文件
var indexRouter = require("./routes/index");
// 处理路由
app.use("/index", indexRouter);
```

这里配置处理路由之后，客户端要请求 index，直接请求 `/index` 就行。这样进行细分就更方便请求。。。

所以现在在 _index.js_ 的**基础**上可以加上其他的请求地址，例如实现插入

```javascript
router.post("/add", (req, res, next) => {
  var params = [req.body.id, req.body.name];
  var query = "insert into student(id,name) values (?,?)";
  db.query(query, params, (err, rows, fileds) => {
    if (err) console.log(err);
    return;
    res.send("成功！");
  });
});
```

## 安装使用 mongoose 模块

👉 <a href="https://smalljerry.cn/2021/04/16/MongoDB%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/">MongoDB 基本使用</a>

# express 托管静态文件

在入口文件中(如：app.js)添加配置

```js
app.use(express.static("public"));
```

- public 是当前静态文件所处的文件夹，文件夹名随便都行

然后就可以通过`http://localhost:3000/images/jerry.jpg`类似的形式访问静态文件了

同时，可以给访问的路径重命名

```js
app.use("/static", express.static("public"));
```

- static 为当前的别名

然后就可以通过`http://localhost:3000/static/images/jerry.jpg`类似的形式访问静态文件了

同时，如果当前 express 运行的目录和静态资源所处的目录不一样（负载均衡），就要使用`path`模块

```js
app.use("/static", express.static(path.join(__dirname, "public")));
```
