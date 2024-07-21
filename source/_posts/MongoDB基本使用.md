---
title: MongoDB基本使用
toc: true
date: 2021-04-16 15:13:44
tags: 数据库
categories: 后端学习
thumbnail: https://z3.ax1x.com/2021/04/20/cHtWHs.png
cover: https://z3.ax1x.com/2021/04/20/cHtWHs.png
---

简单记录<span style="color: purple;">命令行</span>、<span style="color: green">Node.js</span>操作<span style="color: lightgreen">MongoDB</span>

使用版本 ：社区版`4.2.6`

<!-- more -->

MongoDB属于非关系型数据库，数据的存储类似于 *JSON对象*

```json
{
    name: 'lisi',
    age: 18,
    hobbies: ['吃饭', '睡觉']
}
```

# 安装

## MongoDB安装

👉 <a href="https://www.mongodb.com/try/download/community">MongoDB下载地址</a>

下载的是msi版本，点击安装，选择**custom**自定义安装

![自定义安装](https://z3.ax1x.com/2021/04/16/cfSZOe.png)

下一步、下一步，在 *Install MongoDB Compass* 处取消打钩，先不安装这个

![取消打钩](https://z3.ax1x.com/2021/04/16/cfSrlT.png)

MongoDB Compass 是 MongoDB 的一款图形化管理工具，除此外还有别的工具。这里不下载是因为下载很慢，可能会导致 MongoDB 安装不成功；可以自己在浏览器下载，这样快很多。

👉 <a href="https://www.mongodb.com/try/download/compass">MongoDB Compass下载地址</a>

## mongoose安装

<span style="color: green">Node.js</span>操作MongoDB，需要下载插件 *mongoose*

下载：**`npm install mongoose`**

# 启动

## 启动

**命令行启动**

首先进入MongoDB目录里的 *bin* 目录，执行命令，并且选择要执行的位置

```bash
mongod --dbpath 盘符:\目录\
```

**连接**

在 *bin* 目录下执行命令即可启动

```bash
mongo.exe
```

**启动、关闭服务**

```bash
net start MongoDB
```

```bash
net stop MongoDB
```

**移除服务**

在MongoDB目录下的 bin 里面

```bash
mongod.exe --remove
```

**任务管理启动**

对于 *Windows*，打开任务管理，点击 *服务*，*打开服务*，在服务窗口就可以找到MongoDB并且启动

## 添加环境变量

直接在系统变量的`path`中加上一条MongoDB目录的 bin 目录即可。

![环境变量](https://z3.ax1x.com/2021/04/16/chKMmd.png)

# 使用

分为命令行、可视化工具、Node.js

## 图形界面使用

![图形界面连接](https://z3.ax1x.com/2021/04/16/chndgg.png)

这串东西在命令行运行 `mongo` 的时候会出现。

## 命令行使用

在命令行直接使用就可连上，用户名和密码是可选的

```bash
mongo
```

如果想用用户名密码

```bash
mongodb://admin:123456@localhost/
```

如果还想指定数据库

```shell
mongodb://admin:123456@localhost/test
```

### 数据库

**使用**数据库，如果不存在就创建这个数据库 (刚创建的数据库是看不见的，要插入内容才能看见)

```shell
use 数据库名
```

查看当前数据库

```shell
db
```

查看所有数据库

```bash
show dbs
```

**删除**数据库，使用 `use` 选中要删除的数据库，然后执行

```shell
db.dropDatabase()
```

**导入数据库**

```shell
mongoimport -d 数据库名称 -c 集合名称 --file 要导入的数据文件
```

- 先要将其添加到环境变量

### 集合

**创建集合**

```shell
db.createCollection(name, options)
```

options选项，当使用时，以对象的形式使用

| 字段   | 类型   | 描述                                                         |
| ------ | ------ | ------------------------------------------------------------ |
| capped | 布尔值 | 当为 true 的时候，指创建固定大小的集合，必须指定 size 的值，超过时会覆盖最早的值 |
| sise   | 数值   | 指定集合的最大值                                             |
| max    | 数值   | 指定集合中文档的最大数量                                     |

**删除集合**

在要删除集合所处的数据库中

```shell
db.collection.drop()
```

### 文档 - 增删改查

#### 插入

```shell
db.集合名字.insert(document)
```

- 如果插入的数据主键已经存在，报错

```shell
db.集合名字.save(document)
```

- 如果主键存在，则更新数据；如果不存在则插入数据（逐渐不使用这个方法）

插入一条

```shell
db.document.insertOne(
	<document>,
	{
		writeConcern: <document>
	}
)
```

插入多条

```shell
db.document.insertMany(
	[ <document 1>, <document 2> ],
	{
		writeConcern: <document>,
		ordered: <boolean>
	}
)
```

以上用到的参数

| 参数         | 说明                                     |
| ------------ | ---------------------------------------- |
| document     | 要插入的文档                             |
| writeConcern | 写入策略，默认1，要求确认写操作，不要就0 |
| ordered      | 指定是否顺序写入，默认true               |

#### 删除

```shell
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

| 参数         | 说明                                                 |
| ------------ | ---------------------------------------------------- |
| query        | 可选，删除的条件                                     |
| justOne      | 如果为 1 或 true，只删除一条，如果不设置，删除匹配的 |
| writeConcern | 抛出异常的级别                                       |

#### 更新

```shell
db.collection.update(
	<query>,
	<update>,
	{
		upsert: <boolean>,
		multi:<boolean>,
		writeConcern: <document>
	}
)
```

| 参数         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| query        | 更新的查询条件                                               |
| update       | 要更新的对象，类似于sql中的 set xxx                          |
| upsert       | 可选，如果不存在update的记录，是否插入objNew，默认是false，不插入。 |
| multi        | 可选，只更新找到的第一条数据。默认 false                     |
| writeConcern | 可选，抛出异常的级别                                         |

#### 查询

```shell
db.collection.find(query, projection)
```

| 参数       | 说明                                               |
| ---------- | -------------------------------------------------- |
| query      | 查询条件                                           |
| projection | 指定要返回的文本的键，如果返回所有，不用写这个参数 |

```shell
db.col.find().pretty()
```

- 格式化输出的内容

👉 **<a href="https://www.runoob.com/mongodb/mongodb-operators.html" style="color: green;">去菜鸟教程看深入内容</a>**

## Node.js 操作

### 连接

```shell
mongoose
  .connect('mongodb://localhost/mongotest', { useNewUrlParser: true })
  .then(() => {
    console.log('连接成功')
  })
  .catch((err) => {
    console.log('连接失败', err)
  })
```

- 如果指定的数据库不存在，则会自动创建数据库（创建了不插入数据，在列表中是找不到的）
- `{ useNewUrlParser: true }`：新版的MongoDB会有这样的提示，不用特别去记，如果不加上系统会有提示的

### 增删改查

#### 创建

**创建集合**（数据表）：分为两步

1. 设置集合的规则
2. 创建集合

```js
// 设定集合规则
const courseSchema = new mongoose.Schema({
    name: String,
    author: String,
    isPublished: Boolean,
    tags: [String]
});
// 创建集合并应用规则
const Course = mongoose.model('Course', courseSchema);
```

- `model`：里面的集合名称首字母要大写，生成的名称会是小写，然后是复数形式，如上的*courses*，这个方法返回一个**构造函数**，用于操作数据

**创建文档**（每一条数据）：分为两步

1. 创建集合实例
2. 调用实例对象下的save方法将数据保存到数据库中

创建的时候会自动为每一条数据生成一个id，`_id`

```js
const course = new Course({
  name: 'node.js',
  author: '河马程序员',
  isPublised: true,
  price: '20'
})
course.save()
```

还有一种方法，使用 `create()` 方法，这不用调用 save()

- 也分为两种形式，回调形式和Promise形式

```js
Course.create(
  { name: 'JavaScript', author: '河马有用', isPublised: false, price: 44 },
  (err, result) => {
    console.log(err)
    console.log(result)
  }
)
```

Promise形式

```js
Course.create({
  name: 'Python额',
  author: '黑兔程序员',
  isPublised: true,
  price: 49,
  tag: ['程序员', 'python']
})
  .then((result) => console.log(result))
  .catch((err) => console.log(err))
```

#### 查询

基础查询

```js
Course.find().then((result) => console.log(result))
// 加入筛选条件，是一个对象;返回值是一个数组
Course.find({ author: '白马程序员' }).then((result) =>
  console.log('find返回数组', result)
)
// 只返回一条,默认返回第一条
Course.findOne().then((result) => console.log('findOne只返回一条', result))
```

范围查询

- $gt：大于， $lt：小于
- $in：包含

```js
Course.find({ price: { $gt: 20, $lt: 30 } }).then((result) =>
  console.log('范围查询', result)
)
```

```js
Course.find({ tag: { $in: ['js'] } }).then((result) =>
  console.log('包含', result)
)
```

字段查询

- 字段：选择要查询输出的字段，在find()后面链式调用select(),多个字段用空格隔开
- 不想查询谁，在前面加上 “-”，如：-id

```js
Course.find()
  .select('name author -id')
  .then((result) => console.log('字段查询', result))
```

排序

- 在find()后面链式调用sort()

```js
Course.find()
  .sort('price')
  .then((result) => console.log('排序', result))
```

跳过、限制

```js
Course.find()
  .skip(2)
  .limit(3)
  .then((result) => console.log(result))
```

#### 删除

删除一条

```js
Course.findOneAndDelete({ _id: '606c49904798cf25005e0cd6' }).then((result) =>
  // 返回的就是删除的那条，如果匹配了多条，会删除第一条
  console.log(result)
)
```

删除多条

- ！！如果传了空对象{}，会删除所有。返回值是一个对象，OK和n，ok为1代表成功，n代表删除的条目

```js
Course.deleteMany({}).then((result) => console.log(result))
```

#### 更新

更新一条

```js
Course.updateOne({ name: 'Python额' }, { name: 'Python学习' }).then((result) =>
  console.log(result)
)
```

更新多条

- 如果传递一个空对象，即是更新所有

```js
Course.updateOne({ name: 'Python额' }, { name: 'Python学习' }).then((result) =>
  console.log(result)
)
```

### mongoose验证

在创建验证规则的时候，可以设置当前字段的验证规则，验证失败就输入插入失败

- require: true 必传字段。如下，如果title没传就会报错
- require: [true, '请传入标题']，也可以是一个数组
- minlength类型对于字符串，min类型对于数值

这样

```js
const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true,
    minlength: 2,
    maxlength: 5,
    trim: true
  }
})
```

或者这样

```js
const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, '请传入文章标题'],
    minlength: [2, '标题最小不能小于2位'],
    maxlength: [5, '标题最小不能小于5位'],
    trim: true
  },
  age: {
    type: Number,
    min: 10,
    max: 60
  },
  publishDate: {
    type: Date,
    // default的可以不传
    default: Date.now
  },
  category: {
    type: String,
    // enum指定要传的值，不是指定的值不能传
    enum: ['html', 'css', 'js']
  },
  // 自定义验证
  author: {
    type: String,
    validate: {
      validator: (val) => {
        // 返回布尔值，val是要验证的值
        return val && val.length > 4
      },
      // 自定义消息
      message: '传入的值不符合验证规则'
    }
  }
})

const Post = mongoose.model('Post', postSchema)
Post.create({
  title: '开心',
  age: 21,
  category: 'html',
  author: 'asd'
})
  .then((result) => console.log(result))
  .catch((error) => console.log(error))
```

### 集合关联

```js
// 用户规则
const userSchema = new mongoose.Schema({
  name: {
    type: String
  }
})
// 书本规则
const bookSchema = new mongoose.Schema({
  title: {
    type: String
  },
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
})
// 用户集合
const User = mongoose.model('User', userSchema)
// 书本集合
const Book = mongoose.model('Book', bookSchema)

// 创建用户
User.create({ name: '铜锣湾扎fit人' }).then((result) => console.log(result))
// 创建书本
Book.create({
  title: '如何做洪兴大佬',
  author: '606daa780bd6c1573c703c48'
}).then((result) => console.log(result))

// 这条查出的是书本信息，但是作者是当前作者的id
// Book.find().then()
Book.find()
  .populate('author')
  .then((result) => console.log(result))
```

可能增补、修改......

<!-- <span style="color: green; font-size: 100px; text-shadow: 2px 2px 1px gold ">mongoDB</span> -->



<!-- <b style="color: gold; font-size: 30px">  📄&nbsp;基本使用</b>     <span style="color: orange; font-size: 25px">AND</span>     <span style="color: green; font-size: 45px;text-shadow: 2px 2px 1px lightgreen">Node.js</span> -->































